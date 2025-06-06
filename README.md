# Turning up for the Races
A final data-analysis project for the ETH Methods II course. This analysis uses (fixed-effects) regression to estimate the relationship between electoral closeness and voter turnout.

## Motivation
Understanding voter turnout is a cornerstone of political science. Scholars have long debated why people decide to vote, and whether institutional or contextual factors, like compulsory voting, election timing, or media access, affect participation.

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
-	Y is turnout, 
-	X is closeness,
-	𝛼 is the intercept,
-	𝛽 is the coefficient for closeness,
-	𝜖 is the error term.

#### Model 2: Pooled OLS (Democracies Only)
Same as Model 1, but restricted to `democracy` = 1.


#### Model 3: Two-Way Fixed Effects (Democracies)

𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
-	𝑌𝑖,𝑡 is turnout in country i in the year t,
-	𝑋1𝑖,𝑡 is the closeness of the election in country i in the year t, 
-	𝛼𝑖 is the intercept for country i,
-	𝛿𝑡 is the intercept for time t,
-	𝛽1 is the coefficient for closeness,
-	𝜖𝑖,𝑡 is the error term or residual for country i in year t.

#### Model 4: Two-Way FE + Controls (Democracies)
𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 + 𝛽2𝑋2𝑖,𝑡 + 𝛽3𝑋3𝑖,𝑡 + 𝛽4𝑋4𝑖,𝑡 + 𝛽5𝑋5𝑖,𝑡 + 𝛽6𝑋6𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
-	𝑋2𝑖,𝑡 is the population of a country i in year t,
-	𝑋3𝑖,𝑡 is whether voting was compulsory in country i in year t,
-	𝑋4𝑖,𝑡 is whether there where concurrent elections in country i in year t,
-	𝑋5𝑖,𝑡 is the type of election in country i in year t,
-	𝑋6𝑖,𝑡 is the weekday the vote was held on in country i in year t,
-	𝛽2 is the coefficient for population size,
-	𝛽3 is the coefficient for compulsory voting,
-	𝛽4 is the coefficient for concurrent elections,
-	𝛽5 is the coefficient for the election type,
-	𝛽6 is the coefficient for the weekday.

#### Model 5: Two-Way FE + Controls + Gini (Democracies)

𝑌𝑖,𝑡 = 𝛼𝑖 + 𝛿𝑡 + 𝛽1𝑋1𝑖,𝑡 +… + 𝛽7𝑋7𝑖,𝑡 + 𝜖𝑖,𝑡

Where:
-	𝛽7𝑋7𝑖,𝑡 is the economic inequality of a country i in year t,
-	𝛽7 is the coefficient for the economic inequality.
-	Runs on subset with Gini values.

## Results
A larger value of `closeness_per` means a less competitive election. A negative β implies that as margins widen (less competitive), turnout declines.

### Model 1 (Pooled OLS, All Countries)
- **β₁ (closeness_per):** 0.040 (SE = 0.013) *** (p < 0.01)  
- **R²:** 0.004  
- **Interpretation:**  
  - A 1 pp wider margin → +0.04 pp turnout.  
  - Statistically significant but trivially small.  
  - R² shows almost no explanatory power.  

### Model 2 (Pooled OLS, Democracies)
- **β₁ (closeness_per):** –0.148 (SE = 0.027) *** (p < 0.01)  
- **R²:** 0.018  
- **Interpretation:**  
  - In democracies, a 1 pp wider margin → –0.148 pp turnout (significant but still small).  
  - Low R² again.  

### Model 3 (Two-Way FE, Democracies)
- **β₁ (closeness_per):** –0.0574 (SE = 0.020) *** (p < 0.01)  
- **Adj R²:** 0.673  
- **Interpretation:**  
  - Controlling for country & year FE, a 1 pp wider margin → –0.0574 pp turnout.  
  - Fixed effects capture most variation; closeness remains negative.
 
### Model 4 (Two-Way FE + Controls, Democracies)
- **β₁ (closeness_per):** –0.093 (SE = 0.022) *** (p < 0.01)  
- **Adj R²:** 0.687  
- **Key Controls:**  
  - `populationNohl` (positive coefficient)  
  - `compulsory` (positive)  
  - `concurrent` (positive)   
- **Interpretation:**  
  - A 10 pp wider margin → –0.93 pp turnout, holding other factors constant.


### Model 5 (Two-Way FE + Controls + Gini, Democracies)
- **β₁ (closeness_per):** –0.096 (SE = 0.025) *** (p < 0.01)  
- **β₇ (gini):** (c. 0.40, SE = 0.08) *** (p < 0.01)  
- **Adj R²:** 0.702  
- **Notes:**  
  - Only 964 observations (many country-years lack Gini).  
  - Inequality has a large positive association—moving from min to max Gini yields a big turnout change (caution: not realistic to go 0 → 1).  
- **Interpretation:**  
  - Closeness effect unchanged (–0.096).  
  - Income inequality appears important but must be interpreted over observed Gini range (≈0.25–0.60).


## Overall Takeaways

1. **Models 1 & 2 (Pooled OLS)**: Closeness is significant but explains almost no variance (R² = 0.004–0.018).  
2. **Models 3–5 (Two-Way FE)**: Closeness remains negative (–0.057 to –0.096) and significant (p < 0.01). Fixed effects capture most of the variation (Adj R² ≈ 0.67–0.70).  
3. **Effect sizes**: A 1-pp wider margin reduces turnout by 0.04–0.10 pp—small but consistent with prior literature (Blais 2006).  
4. **Controls & Gini**:  
   - Compulsory voting, population size, and election concurrency all behave as expected.  
   - Income inequality (Gini) has a large positive coefficient, but we only observe Gini ∈ [0.25, 0.60], so a more modest effect is realistic.  

**Conclusion:** We reject the null that “closeness has zero effect.” Closer elections do see marginally higher turnout, but the practical impact is modest.  


## Usage

The analysis is contained in `Turning-up-for-the-Races.Rmd`. If you want to rerun preprocessing from raw files, see `Preprocessing.Rmd`.

### 1. Clone the Repository

```bash
git clone https://github.com/Dem-guy/Turning-up-for-the-Races.git
cd Turning-up-for-the-Races
```

### 2. Unzip V-Dem Data
```bash
unzip data/V-Dem-CY-Core-v13.csv.zip -d data/
```

### 3. Install R Packages
In R
```r
install.packages(c(
  "tidyverse",
  "haven",
  "stargazer"
))
```

### 4. Render the R Markdown 
In R
```r
rmarkdown::render("Turning-up-for-the-Races.Rmd")
```

To only rerun data preprocessing (e.g., if you changed merges), run:
```r
rmarkdown::render("Preprocessing.Rmd")
```
This generates the cleaned merged datasets which Turning-up-for-the-Races.Rmd works with.

## References
Fauvelle-Aymar, C., & François, A. (2006). The impact of closeness on turnout: An empirical relation based on a study of a two-round ballot. Public Choice, 127, 461–483.

## License: All rights reserved





