# Turning up for the Races
A final data-analysis project for the ETH Methods II course. This analysis uses (fixed-effects) regression to estimate the relationship between electoral closeness and voter turnout.

## Motivation
Understanding voter turnout is a cornerstone of political science. Scholars have long debated why people decide to vote, and whether institutional or contextual factors—like compulsory voting, election timing, or media access—affect participation.

A frequently studied influence is **electoral closeness**: do citizens turn out in larger numbers when an election is expected to be competitive? While many case studies examine this question, empirical findings remain mixed. Using the new Global Dataset on Turnout, this project asks whether ex-post measures of electoral closeness (victory‐margin) predict higher turnout, thereby contributing to the ongoing debate on whether “closeness” genuinely motivates voters.

**Hypothesis (H1):** The closer an election is, the higher voter turnout will be.

## Data
### Global Dataset on Turnout (GDT)

**Source:**  
[Martínez i Coma, F., & Leiva Van De Maele, D. (2023). Global Dataset on Turnout (GD-Turnout). Harvard Dataverse.](https://doi.org/10.7910/DVN/NYXHU9)

**Description:**  
The GDT contains panel data on all national presidential and parliamentary elections worldwide between 1945–2020. Key points:
- **Turnout**: percentage of votes cast out of total registered voters (`turnoutreg`).  
- **Closeness**: not provided directly. I compute two versions:
  1. `closeness` = absolute vote‐count margin (first minus second).  
  2. `closeness_per` = percentage‐point margin (vote share of winner minus runner‐up).
  - A smaller value → a closer election (ex-post measure).  
  - Note: ex-post closeness can be biased (perception vs. actual), but prior work shows ex-post and ex-ante measures yield similar turnout effects (Fauvelle-Aymar & Francois 2006).

- **Control variables** (all binary unless noted):  
  - `compulsory` (1 if voting is legally compulsory, 0 otherwise)  
  - `concurrent` (1 if held alongside another national election, 0 otherwise)  
  - `streltype` (election type: “P” = presidential, “L” = legislative)  

- **Additional covariates**:  
  - `weekday` (day of week: “Monday”–“Sunday”)  
  - `populationNohl` (total population, continuous)

**Preprocessing:**  
- Dropped observations where the runner-up’s vote count exceeds the winner’s (data error).  
- Created `closeness_per` = (winner_vote_share – runner_up_vote_share).
  - A smaller value → a closer election (ex-post measure).  
  - Note: ex-post closeness can be biased (perception vs. actual), but prior work shows ex-post and ex-ante measures yield similar
    turnout effects (Fauvelle-Aymar & Francois 2006).
- Kept only these columns for analysis:  
  - `country`  
  - `year`  
  - `turnoutreg`  
  - `closeness`  
  - `closeness_per`  
  - `weekday`  
  - `streltype`  
  - `compulsory`  
  - `populationNohl`  
  - `concurrent`  
- Renamed country codes/strings to match the other datasets (Gini, V-Dem) (e.g., “Russian Federation” → “Russia,” etc.).

### Gini Coefficient Data

**Source:**  
[Hasell, J., Arriagada, P., Ortiz-Ospina, E., & Roser, M. (2023). Economic Inequality. OurWorldInData.org.](https://ourworldindata.org/economic-inequality)

**Description:**  
Annual Gini coefficient (scaled 0–1) for each country. A higher Gini value indicates greater income inequality.

**Preprocessing:**  
- Filtered years to 1945–2020 (to match GDT span).  
- Kept only:  
  - `country`  
  - `year`  
  - `gini`  
- Renamed countries for consistency with GDT and V-Dem.

### Varieties of Democracy (V-Dem)
**Source:**  
[Coppedge, M., Gerring, J., Knutsen, C. H., Lindberg, S. I., Teorell, J., Altman, D., … Ziblatt, D. (2023). V-Dem [Country-Year/Country-Date] Dataset v13. Varieties of Democracy Project.](https://doi.org/10.23696/vdemds23)

**Description:**  
Annual country-level democracy indicators dating from 1789. Contains hundreds of variables coded by experts, including electoral and liberal democracy indices.

**Preprocessing:**  
- Kept only:  
  - `country`  
  - `year`  
  - `v2x_polyarchy` (Electoral Democracy Index, continuous 0–1)  
- Created binary indicator `democracy`:  
  - `1` if `v2x_polyarchy ≥ 0.50` (electoral democracy),  
  - `0` otherwise.  
- Dropped missing values in `v2x_polyarchy`.  
- Renamed country labels for consistency with GDT and Gini.

### Final Merged Datasets

After merging GDT + V-Dem (+ Gini), we end up with two versions:

1. **Without Gini:**  
   - **Observations:** 2,643 election instances (1945–2020)  
   - **Countries:** 156  

2. **With Gini:**  
   - **Observations:** 964 (subset where Gini is available)  
   - **Countries:** 116  



## Methods
I estimate the linear effect of electoral closeness on turnout. Each model’s results are considered significant at α = 0.05.

#### Model 1: Pooled OLS (All Countries)

𝑌 = 𝛼 + 𝛽𝑋  + 𝜖

Where:
•	Y is turnout, 
•	X is closeness,
•	𝛼 is the intercept,
•	𝛽 is the coefficient for closeness,
•	𝜖 is the error term.

#### Model 2: Pooled OLS (Democracies Only)
Same as Model 1, but restricted to `democracy` = 1.


#### Model 3: Two-Way Fixed Effects (Democracies)
\[
  \text{turnoutreg}_{i,t} = \alpha_{i} + \delta_{t} + \beta_{1} \times \text{closeness\_per}_{i,t} + \epsilon_{i,t}
\]
- \(\alpha_{i}\): country FE  
- \(\delta_{t}\): year FE  

𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
•	𝑌𝑖,𝑡 is turnout in country i in the year t,
•	𝑋1𝑖,𝑡 is the closeness of the election in country i in the year t, 
•	𝛼𝑖 is the intercept for country i,
•	𝛿𝑡 is the intercept for time t,
•	𝛽1 is the coefficient for closeness,
•	𝜖𝑖,𝑡 is the error term or residual for country i in year t.

Model 4 adds control variables from the GDT dataset. These include population size, compulsory voting, election concurrency, the election type, and the weekday.

𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 + 𝛽2𝑋2𝑖,𝑡 + 𝛽3𝑋3𝑖,𝑡 + 𝛽4𝑋4𝑖,𝑡 + 𝛽5𝑋5𝑖,𝑡 + 𝛽6𝑋6𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
•	𝑋2𝑖,𝑡 is the population of a country i in year t,
•	𝑋3𝑖,𝑡 is whether voting was compulsory in country i in year t,
•	𝑋4𝑖,𝑡 is whether there where concurrent elections in country i in year t,
•	𝑋5𝑖,𝑡 is the type of election in country i in year t,
•	𝑋6𝑖,𝑡 is the weekday the vote was held on in country i in year t,
•	𝛽2 is the coefficient for population size,
•	𝛽3 is the coefficient for compulsory voting,
•	𝛽4 is the coefficient for concurrent elections,
•	𝛽5 is the coefficient for the election type,
•	𝛽6 is the coefficient for the weekday.

Model 5 adds the variable indicating economic inequality.

𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 +… + 𝛽7𝑋7𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
•	𝛽7𝑋7𝑖,𝑡 is the economic inequality of a country i in year t,
•	𝛽7 is the coefficient for the economic inequality.

## Results
For the interpretation of the coefficients, it is important to keep in mind that lower values of the independent variable indicate a closer election, as lower values indicate a smaller margin of victory. Consequently, as the margin increases in size, turnout is expected to decrease.

Model 1 estimates the coefficient for closeness at 0.040 with a standard error of 0.013, indicating that on average, when an election decreases in its closeness by 1 percentage point, it will increase voter turnout by 0.04 percentage points. The coefficient is statistically significant at the 1% level.
The results of Model 1 imply three things: First, the effect of closeness on turnout appears to be very small. Second, there is sufficient evidence to reject the null hypothesis. Third, the results do not support H1, which predicted that turnout would decrease as the vote gap increases.
Regarding this final point, Graph 1 can partially explain this result. Election closeness affects turnout differently in democracies and non-democracies. As an election should be competitive for closeness to motivate citizens participation, including non-democracies into the analysis may be masking the true effect of the independent variable. 

The R2 of Model 1 is 0.004, a small number by any reasonable standard. Closeness alone does not sufficiently explain the variance of turnout. Consequently, the explanatory power and predictive ability of this model are limited.

Model 2 only looks at democracies, with results visualized in Graph 2. The coefficient of -0.148 (standard error of 0.027) falls in line with H1 expectations and is larger in absolute terms than that of Model 1, as turnout is predicted to decrease by -0.148 percentage points per unitary increase in the gap of the vote share. The results remain significant at the 1% level.
Despite this, with a R2 of 0.018, Model 2 can still barely explain the variance within the data. 

Graph 2 also indicates what could already be observed in Graph 1, which is that most elections appear to be close affairs irrespective of their turnout levels. That said, most of the datapoints appear to cluster around small vote gap and high turnout.

Model 3 is a major improvement in terms of its ability to explain the variation of turnout. With an adjusted R2 of 0.673, the explanatory power and predictive ability of Model 3 are relatively high.
The model estimated the coefficient for closeness at -0.0574 (standard error of 0.020), which remains significant at the 1% level, which again soundly rejects the null hypothesis, and the relationship appears in line with H1.

With the inclusion of additional variables into Model 4, the coefficient for closeness “increases” to -0.093. The estimation is still statically significant at the 1% level. These results would indicate that an election with a vote share difference of 10 percentage points between the leading candidates should see an approximate 1 percentage point decrease in turnout compared to an election that is neck on neck.

Model 5 follows the same trend, with the coefficient of closeness being -0.096. This model features the highest R2 and adjusted R2 but is also based on the lowest number of observations. Economic inequality (gini) appears to have an extremely large effect on turnout, but as the variable is measured 0-1, it represents the difference in turnout between a country with complete economic equality to a country with none. 

While the first two models are too weak in their explanatory power to draw meaningful conclusions, models 3-5 all indicate that closeness has a statistically significant, non-zero effect on turnout. Additionally, in all models except Model 1, the relationship between closeness and turnout is predicted to be negative, as H1 expected. Overall, the results provide sufficient evidence to reject the null hypothesis and appear to support H1.
That said, the actual effect size of closeness on turnout is small across all models, indicating that closer elections are not to be associated with large increases in turnout. This falls in line with what previous authors have previously found (see Blais 2006). Based on the two models with the highest R2, when the margin of victory increases by one percentage point, one can expect an average decrease in voter turnout between -0.093 and -0.096 percentage points.

There are limitations to this analysis. First, measuring closeness ex-post is not unproblematic. When possible, it would be preferable that future research truly measure perceived closeness. If causal claims are to be made about the relationship between closeness and turnout, this point holds especially true, as the margin of victory cannot truly be causal for something that proceeds it. 
Second, in such a cross national observational study, it is impossible to account for all factors. While many relevant variables were controlled for, some notable exclusions include population concentration, population ethnicity and campaign expenditures. Potentially most relevant of all, the analysis did not control for weather an election took place in a majoritarian or consensual democracy, and there are reasons to believe that measuring closeness in the latter would need to be conducted differently (see Blais 2006: 120). Despite this, considering that the results remain robust and statistically significant over three models with a high R2, there are grounds to remain confident in these findings.

Shortcomings aside, this analysis has provided evidence that citizens are more likely to vote in close elections, and therefore finds itself in line with the previous findings works such as that of Blais and Dobrzynska (1998) and Cancela and Geys (2016), even if the projected influence of closeness on turnout appears to be small. While more research is needed to explain under which condition closeness may play a greater or lesser role, determining the validity of the association is a necessary and relevant first step.

## Usage

Data is already preprocessed in the Turning-up-for-the-Races.Rmd file. Preprocessing can be run sepeartly if desired with the Preprocessing.Rmd (Note: V-Dem-CY-Core-v13.csv.zip will need ot be unziped forst)


### 1. Clone the Repository

```bash
git clone https://github.com/Dem-guy/Turning-up-for-the-Races.git
cd Turning-up-for-the-Races
```

### 2. Install R Packages
In R
```r
install.packages(c(
  "tidyverse",
  "haven",
  "stargazer"
))

```

### 3. Render the R Markdown 
In R
```r
rmarkdown::render("Turning-up-for-the-Races.Rmd")
```

Or, if preprocessing step should be rerun, run following first
```r
rmarkdown::render("Preprocessing.Rmd")
```

## References
Fauvelle-Aymar, C., & François, A. (2006). The impact of closeness on turnout: An empirical relation based on a study of a two-round ballot. Public Choice, 127, 461–483.

## License: All rights reserved





