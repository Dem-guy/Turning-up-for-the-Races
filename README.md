# Turning up for the Races
A final data analysis project for ETH course Methods II. This analysis uses logistic regression to estaimate the realtionship between how close an election is and voter turnout.

## Motivation
Understanding voting behavior is a cornerstone of political science. Debates on when and why people decide to vote have generated an enormous amount of academic attention. Amongst the many possible influences contributing to voter turnout, the closeness of an election is a frequently analyzed aspect (Geys 2006: 647). Despite this, there is disagreement within the literature whether closeness truly does impact voter turnout. Using a relatively new dataset, I aim to answer whether citizens are more likely to vote in closer elections, and in doing so contribute to the ongoing debate on the predictive quality of closeness on voter turnout. Therefore, I attempt to test for the following hypothesis.

H1: The closer an election is, the higher voter turnout will be.

## Data
### Global Dataset on Turnout
**Source**
Martínez i Coma, F., & Leiva Van De Maele, D. (2023). Global Dataset on Turnout (GD-Turnout). Retrieved from: https://doi.org/10.7910/DVN/NYXHU9

**Description:**  
The Global Dataset on Turnout (GDT) dataset contains panel data on all national presidential and parliamentary elections between 1945-2020. The GDT itself does not contain an indicator for election closeness. Instead, the independent variable is determined by taking the margin of victory, which is calculated by subtracting the share of votes the runner-up received from that of the victor. A smaller gap indicates a closer election. This means that this measurement of closeness is determined ex-post, i.e. after the election is already concluded. While this approach is common (Geys 2006: 647), it is not without issue. Ex-post closeness does not indicate the perceived closeness of an election which, which is the theorized explanation for higher turnout. Not only that, but as the actual results are a function of voter turnout, results derived using an ex-post measurement run the risk of being inherently biased. Despite these shortcomings, research has shown that ex-post measurements provide similar results on turnout as ex-ante measurements (Fauvelle-Aymar and Francois 2006). 

Turnout is calculated by the percentage of votes cast in relation to the total number of registered voters. The utlized control variables include compulsory elections, concurrency of elections, and election type  are all measured binarily. 

**Preprocessing**
- Dropped observations where political candidate/party which came in second are recorded as having achieved more votes then first place    candidate/party
- Created the continous variable  `closness_per`, which indicated the difference in received votes of all registerd voters of a country    between the first and secnd placed candidate/party
  - 0 if only non‐repressive responses (accommodation or “ignore”) occurred.
- Removed unused columns. Maintained the following columns for anaylsis: 
  - country
  -  year
  -  turnoutreg (registered turnout percentage)
  -  closeness (closness of an election as differnce in vote count of two largest candidate/party) (unused in follwing alayisis)
  -  closeness_per (closness of an election as % differnce in vote share of two largest candidate/party)
  -  weekday (The name of the day of the week in which the elections are held. From Monday to Sunday)
  -  streltype (String variable of election type, including presidential (P) and parliamentary (L))
  -  compulsory (Whether there was compulsory voting or not)
  -  populationNohl (This variable includes the total population)
  -  concurrent (Whether the election was concurrent with another election)
- Manually renaming various country to match county names with other utlized datasets (see below)
  
### Gini Coefficient Data
**Source**
Hasell, J., Arriagada, P., Ortiz-Ospina, E., & Roser, M. (2023). Economic Inequality. OurWorldInData.org. Retrieved from: https://ourworldindata.org/economic-inequality

**Description:**  
Economic inequality in a specific country-year. It is scaled 0-1, with higher values indicating greater inequality.

**Preprocessing**
- Filtered data to years 1945-2020
  -  country
  -  year
  -  turnoutreg (registered turnout percentage)
  -  closeness (closness of an election as differnce in vote count of two largest candidate/party) (unused in follwing alayisis)
  -  closeness_per (closness of an election as % differnce in vote share of two largest candidate/party)
  -  weekday (The name of the day of the week in which the elections are held. From Monday to Sunday)
  -  streltype (String variable of election type, including presidential (P) and parliamentary (L))
  -  compulsory (Whether there was compulsory voting or not)
  -  populationNohl (This variable includes the total population)
  -  concurrent (Whether the election was concurrent with another election)
- Manually renaming various country to match county names with other utlized datasets

### Varieties of Democracy
**Source**
Coppedge, M., Gerring, J., Knutsen, C. H., Lindberg, S. I., Teorell, J., Altman, D., Bernhard, M., Cornell, A., Fish, M. S., Gastaldi, L., Gjerløw, H., Glynn, A., Good God, A., Grahn, S., Hicken, A., Kinzelbach, K., Krusell, J., Marquardt, K. L., McMann, K., … Ziblatt, D. (2023). V-Dem [Country-Year/Country-Date] Dataset v13. Varieties of Democracy (V-Dem) Project. Retrieved from:  https://doi.org/10.23696/vdemds23

**Description**
Country‐level data from 1789 to present. Contains hundreds of indicators on governance, democracy, civil liberties, media freedom, corruption, and more.

**Preprocessing**
- Removed unused columns. Maintained the following columns for anaylsis: 
  -  country
  -  year
  -  v2x_polyarchy (Electoral democracy index)
- Turned v2x_polyarchy interval scale inot a binary indicator named `democracy`
- Removed missing values 
- Manually renaming various country to match county names with other utlized datasets

### Final Merged Datasets
**Data without Gini value**
- **Observations:** 2643 
- **Countries:** 156 
- **Features:** 12

**Data with Gini value**
- **Observations:** 964
- **Countries:** 116
- **Features:** 13

## Methods
To answer H1, I utilize ordinarily least squares (OLS) methodology, meaning that I assume that the relationship between closeness and turnover is linear.  Results are be considered statistically significant at the 5% level. 

Model 1 is a simple linear regression model, thus regressing turnout only on closeness. It shall be applied to all observations in the GDT. 
𝑌 = 𝛼 + 𝛽𝑋  + 𝜖

Where:
•	Y is turnout, 
•	X is closeness,
•	𝛼 is the intercept,
•	𝛽 is the coefficient for closeness,
•	𝜖 is the error term.

Model 2 is calculated the same way but is only applied to democracies. Subsequent models are also only applied to democracies. 

Model 3 is a two-way fixed effects model, controlling for both country and year fixed effects. Subsequent models also control for fixed effects.

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

### 3. Render the R Markdown (excludes data cleaning procedure in 
In R
```r
rmarkdown::render("State_repression_prediction.Rmd")
```

## License: All rights reserved





