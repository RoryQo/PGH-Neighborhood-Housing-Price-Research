# Pittsburgh Housing Prices Modeling Project

## Overview

This project explores housing prices in Pittsburgh, with a focus on identifying factors that influence property values, such as property characteristics, neighborhood ratings, and sale year. We use a range of statistical techniques, including linear regression, weighted least squares (WLS), ANOVA, and ANCOVA, to understand trends and relationships in housing prices across different neighborhoods.


## Data

The data used in this project is sourced from **Allegheny County**, and we also use additional data from the **Western Pennsylvania Regional Data Center (WPRDC)** for real estate sales and property assessments.

### Transaction Data Set (2013 to Present)  

[![View Original Research Paper](https://img.shields.io/badge/Real%20Estate%20Sales%20Data-0056A0?style=flat&logo=external-link&logoColor=white&color=0056A0)](https://www.aeaweb.org/articles?id=10.1257/app.3.3.1)  

This dataset provides **transactional data** from 2013 to the present. It includes sales prices, sale dates, and property identification numbers (PINs), but **does not contain property characteristics** (such as house size, condition, etc.).  
**Note**: The primary key is **PARID**, which is equivalent to **PIN** (Property Identification Number).

### Assessor Records (Property Assessments)  



[![View Original Research Paper](https://img.shields.io/badge/Assessor%20Records%20Data-0056A0?style=flat&logo=external-link&logoColor=white&color=0056A0)](https://data.wprdc.org/dataset/property-assessments)

This dataset includes **property characteristics** such as square footage, number of rooms, property condition, and assessment values. It provides detailed information about the physical attributes of properties, which is essential for modeling house prices.  
**Note**: The primary key is **PARID**, which corresponds to the **PIN** (Property Identification Number), ensuring that it can be matched with transactional data.

### Pittsburgh Police Arrest Data

[![View Original Research Paper](https://img.shields.io/badge/Pittsburgh%20Arrest%20Data-0056A0?style=flat&logo=external-link&logoColor=white&color=0056A0)](https://catalog.data.gov/dataset/pittsburgh-police-arrest-data)

Arrest data contains information on people taken into custody by City of Pittsburgh police officers. More serious crimes such as felony offenses are more likely to result in an arrest. However, arrests can occur as a result of other offenses, such as parole violations or a failure to appear for trial. All data is reported at the block/intersection level, with the exception of sex crimes, which are reported at the police zone level.

This dataset only contains information reported by City of Pittsburgh Police. It does not contain information about incidents that solely involve other police departments operating within the city (for example, campus police or Port Authority police).

### Allgeheny County ZipCode Boundries

[![View Original Research Paper](https://img.shields.io/badge/Zipcode%20Boundries%20Data-0056A0?style=flat&logo=external-link&logoColor=white&color=0056A0)](https://catalog.data.gov/dataset/allegheny-county-zip-code-boundaries)


This dataset demarcates the zip code boundaries that lie within Allegheny County.If viewing this description on the Western Pennsylvania Regional Data Center’s open data portal, this dataset is harvested on a weekly basis from Allegheny County’s GIS data portal





### Data Cleaning & Preprocessing

- **Merge**: Merge both data sets together horizontally on the key of zipcodes, then remove identical columns from the merge

- **Pittsburgh Specific**: Since this is county wide data we filtered it to include only data that is within the Pittsburgh city limits (ie. City== Pittsburgh)

- **Missing Values**: Approximately 20% of the 90,000 observations had missing data. After inspection, these missing values were determined to be random, so they were dropped. We retained enough remaining observations for meaningful analysis.

- **Sale Descriptions**: The sale descriptions labeled as "love and affection" were removed because they were associated with very low sale prices that did not reflect accurate market values. Upon inspection, the rate of these sales was stable across ZIP codes, so they were deemed to distort the overall analysis. 

- **Other Sale Descriptions**: Other sale descriptions, such as bank repossessions, could lead to lower sale prices. However, these are not equally distributed across ZIP codes and show an underlying pattern in the data. Since they provide valuable insights into the market, these observations were retained in the dataset.

- **Zipcodes**: Zipcodes needed to have enough observations to get a valid comparison and average price, 4 zipcodes were dropped because they had many less observations than the others (< 220)
  
- **Interaction Grid**: Before building the max model and adjusting means we must check for interaction terms, a grid of scatter plots showing all possible interactions was created. **No interaction terms were significant**, and therefore not included in maximum model.

### Data Transformation

- **Extracting Sale Year**: The sale year was extracted from the `SALEDATE.x` column, which contains the full date. This year is then converted into a factor for use in analysis.

- **Encoding Ratings**: The `GRADEDESC` column, which represents neighborhood ratings, was transformed into an ordered factor. This transformation ensures that the neighborhood ratings are treated as ordinal data in the statistical models. The ratings are ordered from "POOR" to "Highest Cost."

### Cleaned Data Availability

The cleaned data is available for download in this repository. You can find the file under the "data" folder or in the relevant section of this project.

## Statistical Models and Analysis

### 1. Exploratory Data Analysis (EDA)
The initial steps involve exploring the data with summary statistics and visualizations to identify patterns and understand the structure of the data:
- **Summary Statistics**: Basic descriptive statistics for the dataset are calculated.
- **Price by ZIP Code**: The average housing prices per ZIP code are computed and visualized using a bar chart with error bars indicating the standard error of the mean.


<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Unadj_Price_zip.jpg?raw=true" width=600px/>
</p>


| Statistic            | N       | Mean        | St. Dev.     | Min       | Max         |
|----------------------|---------|-------------|--------------|-----------|-------------|
| LOTAREA              | 146,470 | 8,215.283   | 23,225.260   | 0         | 2,337,386   |
| STORIES              | 146,470 | 1.742       | 0.501        | 1.000     | 4.000       |
| YEARBLT              | 146,470 | 1,937.094   | 27.891       | 1,780     | 2,024       |
| BASEMENT             | 146,470 | 4.753       | 0.900        | 1         | 5           |
| GRADEDESC            | 146,457 | 7.834       | 1.682        | 1         | 19          |
| CONDITION            | 146,470 | 3.143       | 0.796        | 1         | 8           |
| TOTALROOMS           | 146,470 | 6.457       | 1.830        | 0         | 87          |
| BEDROOMS             | 146,470 | 3.011       | 0.968        | 0         | 18          |
| FULLBATHS            | 146,470 | 1.451       | 0.678        | 0         | 9           |
| HALFBATHS            | 146,470 | 0.416       | 0.554        | 0         | 8           |
| FIREPLACES           | 146,470 | 0.342       | 0.550        |           | 9           |
| BSMTGARAGE           | 146,470 | 0.550       | 0.737        | 0         | 6           |
| FINISHEDLIVINGAREA   | 146,470 | 1,645.914   | 733.217      | 0         | 15,657      |
| PRICE                | 146,470 | 184,973.500 | 270,583.300  | 0         | 10,662,000  |
| SALEYEAR             | 146,470 | 2,018.022   | 3.658        | 2,012     | 2,025       |
| FAIRMARKETTOTAL      | 146,470 | 139,303.800 | 0            | 2,793,400 |             |


### 2. Linear Regression Models
A series of linear regression models are constructed to predict housing prices based on several independent variables.

#### a. Full Model (Max Model)
The **max model** includes all potential predictors of house price, such as `LOTAREA`, `STYLE`, `YEARBLT`, `GRADEDESC`, `CONDITION`, and more.

```math
\text{PRICE} = \beta_0 + \beta_1 \cdot \text{LOTAREA} + \beta_2 \cdot \text{STYLE} + \beta_3 \cdot \text{STORIES} + \beta_4 \cdot \text{YEARBLT} + \beta_5 \cdot \text{EXTERIORFINISH} + \beta_6 \cdot \text{BASEMENT} + \beta_7 \cdot \text{GRADEDESC} + \beta_8 \cdot \text{CONDITION} + \beta_9 \cdot \text{TOTALROOMS} + \beta_{10} \cdot \text{BEDROOMS} + \beta_{11} \cdot \text{FULLBATHS} + \beta_{12} \cdot \text{HALFBATHS} + \beta_{13} \cdot \text{HEATINGCOOLING} + \beta_{14} \cdot \text{FIREPLACES} + \beta_{15} \cdot \text{BSMTGARAGE} + \beta_{16} \cdot \text{FINISHEDLIVINGAREA} + \beta_{17} \cdot \text{SALEYEAR} + \beta_{18} \cdot \text{SALEDESC.x} + \beta_{19} \cdot \text{CONDITION} + \epsilon
```

#### b. Null Model (Min Model)
The **null model** includes only an intercept, essentially predicting the mean price without any independent variables.

```math
\text{PRICE} = \beta_0 + \epsilon
```

#### c. Stepwise Model Selection
The **best model** is selected using a stepwise selection process. This function selects the best predictors based on AIC (Akaike Information Criterion), which balances model fit and complexity. This model will give us an intial jumping off point for future transformations and analysis.


```math
\text{PRICE} = \beta_0 + \beta_1 \cdot \text{GRADEDESC} + \beta_2 \cdot \text{FINISHEDLIVINGAREA} + \beta_3 \cdot \text{SALEDESC.x} + \beta_4 \cdot \text{SALEYEAR} + \beta_5 \cdot \text{HEATINGCOOLING} + \beta_6 \cdot \text{STYLE} + \beta_7 \cdot \text{FULLBATHS} + \beta_8 \cdot \text{LOTAREA} + \beta_9 \cdot \text{HALFBATHS} + \beta_{10} \cdot \text{CONDITION} + \beta_{11} \cdot \text{FIREPLACES} + \beta_{12} \cdot \text{TOTALROOMS} + \beta_{13} \cdot \text{EXTERIORFINISH} + \beta_{14} \cdot \text{YEARBLT} + \beta_{15} \cdot \text{BEDROOMS} + \beta_{16} \cdot \text{STORIES} + \epsilon
```

```
best_model = step(min_model, direction = "both", scope = max_model)
```

### 3. Model Diagnostics

<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Best_modelDiag.jpg?raw=true" width="500"/>
</p>

- **Multicollinearity**: The variance inflation factor (VIF) is calculated to check for multicollinearity among the predictors. A VIF value greater than 10 indicates high collinearity.
- **Influential Points**: Cook's Distance is used to identify and remove highly influential points that might distort the model’s estimates.
- **Heteroskedasticity**: The Breusch-Pagan test detected heteroskedasticity (non-constant variance of residuals). Robust standard errors are applied to correct for this issue.

### 4 Adressing Diagnostics

```
# 3. Check for multicollinearity using VIF
vif_values <- vif(model)
print(vif_values)
```

#### GVIF and GVIF^(1/(2*Df)) for Variables

| Variable                            | GVIF       | Df  | GVIF^(1/(2*Df)) |
|-------------------------------------|------------|-----|-----------------|
| GRADEDESC                           | 1.839543   | 1   | 1.356298        |
| log(FINISHEDLIVINGAREA + 0.001)     | 3.509536   | 1   | 1.873376        |
| SALEDESC.x                          | 1.274342   | 25  | 1.004860        |
| HEATINGCOOLING                      | 1.777424   | 15  | 1.019357        |
| STYLE                                | 2813.159214| 28  | 1.152372        |
| FULLBATHS                           | 3.333085   | 9   | 1.069171        |
| log(LOTAREA + 0.001)                | 39.207922  | 1   | 6.261623        |
| HALFBATHS                           | 1.549149   | 7   | 1.031759        |
| CONDITION                           | 1.118446   | 1   | 1.057566        |
| FIREPLACES                          | 1.486511   | 8   | 1.025087        |
| log(TOTALROOMS + 0.001)             | 4.141634   | 1   | 2.035100        |
| EXTERIORFINISH                      | 1.110825   | 1   | 1.053957        |
| BEDROOMS                            | 6.372283   | 13  | 1.073827        |
| STORIES                              | 15.339543  | 10  | 1.146280        |
| AGE                                  | 3.318723   | 1   | 1.821736        |



#### Identify and remove influential points based on Cook’s Distance

```
influence <- influence.measures(model)
high_influence <- which(influence$infmat[, "cook.d"] > 4/(nrow(df)-length(model$coefficients)-2))
df <- df[-high_influence, ]
```

#### Weighted Least Squares Approach

By applying weights based on the fitted values from an initial model, WLS minimizes the influence of high-variance observations, providing more accurate estimates of the regression coefficients. In **OLS regression**, the ordinary least squares method minimizes the sum of squared residuals under the assumption that the variance of residuals is constant across all levels of the independent variables (homoscedasticity). However, when there is **heteroscedasticity** (non-constant variance of residuals), OLS estimates become inefficient, and the standard errors of the coefficients may be biased. 

To address this, **Weighted Least Squares (WLS)** regression applies a **weight** to each observation, which accounts for the varying variability in the data. The weights are typically the **inverse of the variance** of each observation, and they down-weight observations that have larger residuals (i.e., higher variance) and up-weight those with smaller residuals (i.e., lower variance).

The weights are computed as the inverse of the fitted values from an initial **OLS regression** model:

```math
w_i = \frac{1}{\sqrt{\hat{y}_i}}
```

```math
\text{LOG\_PRICE} = \beta_0 + \beta_1 \times \text{GRADEDESC} + \beta_2 \times \log(\text{FINISHEDLIVINGAREA} + 0.01) + \beta_3 \times \text{SALEDESC.x} + 
\beta_4 \times \text{HEATINGCOOLING} + \beta_5 \times \text{STYLE} + \beta_6 \times \text{FULLBATHS} + \beta_7 \times \log(\text{LOTAREA} + 0.01) + 
\beta_8 \times \text{HALFBATHS} + \beta_9 \times \text{CONDITION} + \beta_{10} \times \text{FIREPLACES} + \beta_{11} \times \log(\text{TOTALROOMS} + 0.01) + 
\beta_{12} \times \text{EXTERIORFINISH} + \beta_{13} \times (\text{SALEYEAR} - \text{YEARBLT}) + \beta_{14} \times \text{BEDROOMS} + \beta_{15} \times \text{STORIES} + \epsilon
```

#### Model Diagnostics

<p align="center">
<img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/1stdiag.jpg?raw=true" width=500px />
</p>

### 5. Box-Cox Transformation
The **Box-Cox transformation** is applied to the dependent variable (`PRICE`) to identify an optimal transformation (lambda) that improves the model’s normality. The test revealed that the optimal value of \(\lambda\) was **2**. This suggests that applying a square transformation to the data (i.e., `PRICE^2`) would be the most effective. However, to handle the skewness effectively, we applied the **square root transformation** instead, which is a commonly used method for right-skewed data such as housing prices. we compared the square root version to the squared version, and the square root version diagnostic was significantly better than the squared. 

The general formula for the Box-Cox transformation is as follows:

```math
\text{PRICE\_transformed} = \frac{\text{PRICE}^\lambda - 1}{\lambda} \quad \text{if} \quad \lambda \neq 0
```

<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Box.jpg?raw=true" width="500"/>
</p>


```math
\text{PRICE\_transformed} = \sqrt{\text{PRICE}}
```

#### Full Transformed Model
```math
\text{PRICE\_transformed} = \beta_0 + \beta_1 \times \text{GRADEDESC} + \beta_2 \times \log(\text{FINISHEDLIVINGAREA} + 0.01) + 
\beta_3 \times \text{SALEDESC.x} + \beta_4 \times \text{HEATINGCOOLING} + \beta_5 \times \text{STYLE} + 
\beta_6 \times \text{FULLBATHS} + \beta_7 \times \log(\text{LOTAREA} + 0.01) + \beta_8 \times \text{HALFBATHS} + 
\beta_9 \times \text{CONDITION} + \beta_{10} \times \text{FIREPLACES} + \beta_{11} \times \log(\text{TOTALROOMS} + 0.01) + 
\beta_{12} \times \text{EXTERIORFINISH} + \beta_{13} \times (\text{SALEYEAR} - \text{YEARBLT}) + 
\beta_{14} \times \text{BEDROOMS} + \beta_{15} \times \text{STORIES} + \beta_{16} \times \text{as.factor(PROPERTYZIP.x)} + 
\beta_{17} \times \text{as.factor(SALEYEAR)} + \epsilon
```
#### Transformed Model Diagnostics

<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/3rddiag.jpg?raw=true" width=500px/>
</p>


### 6. Analysis of Covariance (ANCOVA)
ANCOVA is performed to compare the mean prices of homes across different ZIP codes, adjusting for other variables in the model. The significance of sale year and zipcode tells us that there is significant variation in housing prices from these variables, controlling for all other property and land characteristics from our previous models. Moving forward, we will calculate adjusted means for both the transformed model and the wls model, since they are both incorrect, but are incorrect in opposite ways.  Taking them together can give us a more complete picture.

#### ANOVA Table

| Variable                           | Df   | Sum Sq | Mean Sq | F value   | Pr(>F)    |
|------------------------------------|------|--------|---------|-----------|-----------|
| as.factor(PROPERTYZIP.x)           | 37   | 12011  | 325     | 246.582   | < 2e-16   |
| as.factor(SALEYEAR)                | 13   | 7691   | 592     | 449.356   | < 2e-16   |
| ...                                | ..   | ....   | ...     | .......   | .......   |
| Residuals                          | 140620 | 185128 | 1       |           |           |




### 7. Adjusted Means
Finally, **adjusted means** for the housing prices across ZIP codes are calculated, accounting for nuisance variables like `GRADEDESC`, `SALEDESC.x`, and `HEATINGCOOLING`. These adjusted means are visualized to identify which ZIP codes are the most and least expensive.

#### Transformed Power Model
<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Adj_Price_zip_transformed.jpg?raw=true" width=600px/>
</p>

#### WLS Model
<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Adju_Price_zip_log.jpg?raw=true" width=600px/>
</p>

While the adjusted average housing prices across various Pittsburgh ZIP codes show clear trends, it is important to interpret the data with caution:

**Overlapping Confidence Intervals:**


The overlapping error bars in the visualizations for the top and bottom ZIP codes indicate that, while the prices for these areas differ, the variation within these neighborhoods makes it difficult to confidently state that one ZIP code is definitively the most expensive or on the other hand the cheapest.

**Significant Price Differences Between Highest and Lowest ZIP Codes:**

The red line in the visualization represents the upper limit of the confidence interval for the lowest-priced ZIP code. This means that, while there is overlap in some of the adjusted mean prices, those ZIP codes whose lower confidence interval does not cross the red line can be considered significantly different in terms of pricing from the lowest-priced neighborhood.

Despite the overlapping error within the groups of ZIP codes (highest price and lowest priced), we can confidently say that there is a significant difference in price when comparing the highest and lowest priced neighborhoods, holding all house and land features constant. The clear price gap between these groups indicates that the factors driving high prices in areas like 15232 are substantially different from those in areas like 15235, even after accounting for property features such as size, age, and condition.

## Viewing Adjusted Mean Trends Through Time

####  WLS Model
<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/OGtime.jpg?raw=true" width=600px/>
</p>

The adjusted average housing prices across various Pittsburgh ZIP codes reveal important insights into the housing market dynamics:

**Parallel Trends Across ZIP Codes:**

Both the top three highest and lowest ZIP codes in terms of housing prices, including 15232, 15217, 15222 (high-priced areas), and 15204, 15210, 15235 (lower-priced areas), show fairly parallel trends over time. Despite differences in their price levels, all these areas have experienced similar effects from broader market forces such as inflation and fluctuations in the housing market.
Influence of Timing and Inflation:

This suggests that, although the price points differ across neighborhoods, the overall trend of increasing housing prices (with occasional fluctuations) has been a common experience for both high-end and more affordable areas. This trend reflects a generalized market shift rather than isolated changes in individual neighborhoods.

## Geo-Spatial Analysis

<p align="center">
    <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/HousepriceG.png" alt="Housing Price Analysis" width=500px />
</p>


## Analyzing Factors Influencing Housing Prices in Pittsburgh ZIP Codes

The aim of this analysis is to understand why certain ZIP codes in Pittsburgh have higher housing prices than others by analyzing factors such as **crime rates**, **school district quality**, and other socioeconomic factors. By combining spatial, demographic, and geographic data, this project provides a comprehensive analysis of the key drivers behind property value disparities across Pittsburgh.

## Data Sources

### 1. **Housing Price Data** (`ADJPrice.csv`)
   - **Source**: Real estate listings and property data for Pittsburgh.
   - **Details**: This dataset contains adjusted housing prices for properties in Pittsburgh, including ZIP codes and property features. This dataset helps map housing price trends geographically.

### 2. **Crime Data** (`e03a89dd-134a-4ee8-a2bd-62c40aeebc6f.csv`)
   - **Source**: Pittsburgh’s public crime reporting system.
   - **Details**: Contains data on criminal incidents (arrests) in Pittsburgh, including coordinates of incidents. This data is used to calculate **arrest density**, which helps us understand the relationship between crime rates and housing prices.

### 3. **Geospatial Data**:
   - **ZIP Code Boundaries** (`zips.geojson`): Geographic boundaries of Pittsburgh ZIP codes, which are essential for visualizing housing prices geographically and merging them with crime and school district data.
   - **School District Boundaries** (`alcogisallegheny-county-school-district-boundaries.geojson`): Boundaries of school districts in Pittsburgh. School quality plays a significant role in determining the desirability of an area, and this dataset is used to examine how school district rankings impact housing prices.

### 4. **Population Data** (Manually Compiled)
   - A dataset containing population figures for each ZIP code, used to understand how population density influences housing prices in Pittsburgh.

## Methodology

### Data Merging and Preparation

The analysis requires integrating data from various sources with different structures. The key merging steps are:

1. **Merge Housing Prices with ZIP Codes**: Merge the `zipcode_avg_price` dataset (which contains average prices by ZIP code) with the `zips` GeoDataFrame to analyze how prices vary geographically.
2. **Merge Crime Data**: Perform a spatial join between the **crime data** (arrest incidents) and the **ZIP code boundaries** to calculate **arrest density** in each ZIP code.
3. **School District Data Integration**: Using fuzzy matching, align the **school district rankings** to the corresponding **ZIP codes**.
4. **Population Data Merge**: Merge population data with arrest counts to normalize arrest density by population, allowing for a more accurate comparison of crime rates across neighborhoods.

### Crime Data Analysis

#### Crime Data and Its Importance

Crime is often a significant factor affecting property values. Areas with higher crime rates typically see lower property values due to concerns over safety. **Crime data** in this project comes from Pittsburgh’s public crime reporting system, which provides details on incidents, including arrests, and the locations (latitude and longitude) of these events.

#### Steps for Crime Data Integration:

1. **Geo-spatial Join**: Crime data, which contains coordinates for each incident, is joined with the **ZIP code boundaries** using **GeoPandas**. This spatial join links each crime incident to a specific ZIP code.
   
2. **Arrest Density Calculation**: The number of arrests in each ZIP code is divided by the area of the ZIP code to calculate **arrest density**—the number of arrests per square meter. High arrest density in a ZIP code typically correlates with lower property values, which is expected based on general urban real estate trends.

3. **Crime Rate and Housing Prices**: Once the arrest density is calculated, it is merged with the **housing price data**. This allows us to explore how areas with higher crime rates (higher arrest density) correlate with lower property values.


<p align="center">
    <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Arrest_density.png" alt="Housing Price Analysis" width=500px />
</p>

#### Insights from Crime Data:
- **Higher crime rates** are generally associated with **lower property prices**.
- **Crime hotspots** in Pittsburgh, where arrest density is high, show a marked decrease in average housing prices compared to safer neighborhoods.
- **Neighborhoods with low crime rates** tend to have higher property values, highlighting the importance of safety as a key factor in homebuying decisions.

### School District Data Analysis

#### School Districts and Their Influence on Housing Prices

The quality of local schools is often a significant factor in determining real estate prices, as many homebuyers are willing to pay a premium for access to high-performing school districts. For this project, school district boundaries were mapped using the **GeoJSON file for school districts** in Allegheny County, Pittsburgh.

#### Steps for School District Data Integration:

1. **Fuzzy Matching**: Because the names of school districts in the external dataset may not exactly match those in the ZIP code data, fuzzy matching techniques were used to align school district names to the correct ZIP codes.
   
2. **Merging School District Boundaries**: After the fuzzy matching, the **school district boundaries** were merged with the **ZIP codes** to associate each ZIP code with its corresponding school district.

<p align="center">
    <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/distandzip.png" alt="Housing Price Analysis" width=400px />
</p>

3. **School District Rankings**: School district rankings were then linked to each ZIP code. Since the zipcodes and school district lines do not perfectly match up we average the rank of each school within the zipcode. A higher ranking (indicating a better-performing school district) is associated with a higher price for homes in that ZIP code, as families often prioritize access to quality education.

4. **Rank Inversion**: To ensure better clarity in interpretation, the school district ranking values were inverted, so that a higher value represented a better-ranked school district.


<p align="center">
    <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/schoolrank.png" alt="Housing Price Analysis" width=500px />
</p>


#### Insights from School District Data:
- **Better-performing school districts** correlate with **higher housing prices**. Homebuyers are often willing to pay more for properties located in areas with highly-ranked schools.
- **Areas with high-ranked schools** (such as Mt. Lebanon, Fox Chapel, and Pine-Richland) exhibit higher property values, making school district quality a significant factor in the housing price equation.
- **Neighborhoods with poor-performing schools** tend to have more affordable housing, which might be due to lower demand for homes in those areas.

### Geographic Trends:
- Certain **neighborhoods** in Pittsburgh, even without the best-ranked schools or lowest crime rates, maintain higher property values due to geographic features such as proximity to the city center or desirable scenic views.

## Appendix: Simple Alternative Analysis Approach

In this alternative analysis, we aim to isolate the **underlying value of the location** by subtracting the **fair market value** of the house's physical structure (which includes the appraised value of the building and the lot) from the actual house price. This analysis is based on the assumption that the price of a house can be decomposed into two main components:

- **Fair Market Value (FMV)**: The FMV is assumed to reflect the appraised value of both the land and the building itself. For this analysis, we subtract this value from the house price to estimate the **pure location value**.
- **Location Value**: This value represents the premium placed on a property's location — factors such as proximity to schools, amenities, transportation, and other location-based advantages or disadvantages are not reflected by the FMV.
### Process Overview:

1. **Subtract Fair Market Value**: We subtract the **fair market value (FMV)** of the house (which is the combined appraised value of the building and the land) from the **price of the house**. This leaves us with the **location value**, which captures how much the area or neighborhood contributes to the house price, independent of its physical structure.

```math
    \text{Location\_Value} = \text{Price} - \text{Fair Market Value}
```

2. **Group Differences by ZIP Code**: After calculating the location value for each house, we group the **location value differences** by **ZIP code** to identify whether different areas command different underlying values that are not captured by the physical structure of the house itself.

3. **Analysis Objective**: By comparing the location value across different ZIP codes, we can explore whether certain regions have significantly higher or lower underlying values, independent of the size or condition of the houses themselves. This allows us to assess how much value the **neighborhood or location** adds to the overall house price.


### Interpretation of Results:



<p align="center">
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Diff%20Zip.jpg?raw=true" width="500"/>
  <img src="https://github.com/RoryQo/PGH-Neighborhood-Housing-Price-Analysis/blob/main/Figures/Diff_Time.jpg?raw=true" width="500"/>
</p>


Without controlling for the types of houses and other factors, we can see that over time the difference fluctuates much more than in our main analysis. This suggests that, when ignoring the influence of other variables such as house size, condition, and location, the differences in house prices across time appear more volatile. However, without accounting for these other factors, the **confidence intervals of differences between ZIP codes** become much smaller. This indicates that while the raw differences may appear larger, this indicates that the types of houses are similar within zip codes, but can vary widely between zip codes. This is why other confounding factors are important. When controlling for these factors, the differences between ZIP codes become less pronounced but more reliable.


## Libraries Used

This project leverages the following R libraries:

- **tidyverse**: For data manipulation and visualization.
- **car**: For regression diagnostics and VIF (Variance Inflation Factor).
- **ggplot2**: For creating visualizations.
- **MASS**: For statistical modeling functions.
- **readxl**: For reading Excel files (if needed).
- **softImpute**: For matrix factorization techniques (if any missing values).
- **stargazer**: For presenting summary statistics and regression results.
- **gridExtra**: For arranging multiple plots in a grid.

