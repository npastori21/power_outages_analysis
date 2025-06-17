# Introduction
In this analysis, I evaluated a collection of power outages from 2000-2016 across the U.S. to determine what factors cause severe outages. While there are many questions to ask such as "Do socioeconomic factors affect the severity of outages?" or "Does the power consumption in a state affect outage severity?", I decided to answer the question "Is the severity of power outages affected by location?". Below are the relevant columns I chose for my analysis.

| Variable Name | Description |
| :---          | :---------- |
| `OUTAGE.DURATION` | Duration of outage in hours |
| `DEMAND.LOSS.MW` | The total or peak demand lost during an outage |
| `CUSTOMERS.AFFECTED` | Number of customers affected by the power outage event |
| `ANOMALY.LEVEL` | This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season |
| `TOTAL.SALES` | Total electricity consumption in the U.S. state (megawatt-hour) |
| `POPPCT_URBAN` | Percentage of the total population of the U.S. state represented by the urban population (in %)|
| `POPDEN_URBAN` | Population density of the urban areas (persons per square mile) |
| `POSTAL.CODE` | Postal Code of the U.S. states |
| `CLIMATE.REGION` | U.S. Climate regions as specified by National Centers for Environmental Information |
| `CAUSE.CATEGORY` | Categories of all the events causing the major power outages |
| `MONTH` | Indicates the month when the outage event occurred |

# Data Cleaning and Exploratory Analysis

## Imputation of Values
I primarily focused on measuring the severity of outages by outage duration. Of the 1,534 original datapoints, 244 rows of the dataframe either had `OUTAGE.DURATION` values that were missing or were less than 5 minutes. I chose to exclude the rows that fit this criteria since outages under 5 minutes are trivial and I did not want to use imputed values for my regression model later in the analysis. This leaves us with 1,281 datapoints to use for the initial analysis.

## Univariate Analysis
The bar chart shows the number of outages by climate region. The Northeast, and South have the most outages which could be due to more extreme weather conditions that knock out grids or cause outages due to increased demand for climate-controlling appliances.
 <iframe
 src="plots/region_outages.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>


## Bivariate Analysis
Below is a heatmap showing the average outage duration for states with at least one outage. The map provides more insight into the relationship of states with more extreme weather and the severity of their outages. For example, Wisconsin, Michigan, and New York have average outage durations of over 85 hours, which could be attributed to their harsh winter weather and storms. Additionally, states like Florida and Louisiana's outages average over 65 hours possibly due to their hot summers and exposure to harsh tropical storms and hurricanes. While West Virginia seems to have the most sever outages, further analysis of its outages shows a small sample size (3) with only one major outage of 288 hours.

 <iframe
 src="plots/states.html"
 width="1000"
 height="700"
 frameborder="0"
 ></iframe>
 
## Interesting Aggregate
The table below shows the relationship between average outage duration by season and climate region. I originally hypothesized that colder months in colder climates and hotter months in hot climates would have longer outage durations. As you can see, northern regions like the Northeast and Northwest have higher outage durations during the colder seasons of fall and winter. What's interesting is the fact that the South and Southeast regions have high average outage durations during the fall as well. This correlates closely with hurricane season in the southern regions and could be a possible factor of these outages.

<iframe src="plots/seasons_pivot.html" width="800" height="225"></iframe>


# Framing a Prediction Problem
To further investigate the connection of outage statistics and outage duration, I developed a regression problem between the columns of the outage dataset and outage duration. 

## Evaluation Metrics

| Metric | Definition |
| :-------| :----------------|
| **Mean Absolute Error(MAE)** | The average absolute difference between true outage duration and predicted outage duration. Ensures over and underestimation of outage duration is treated equally when evaluating. The units are hours, making it an easy metric to evaluate. |
| **R-Squared Score** | The proportion of variance in outage duration that the regression model explains. Essentially this tells us how well our model will fit to data with natural variance. The best score is 1.0, but it can be negative if the model is worse. |

## Original Attempt
Below is a preview of the data I used to train my regression models. There were over 500 missing values in the `DEMAND.LOSS.MW` column and over 300 missing values in the `CUSTOMERS.AFFECTED` column. As a result, imputation would not be feasible since it would bias the predictions of my models. Instead, I chose to investigate the relationship between outage duration and the features below on a subset of 490 rows that had no missing values for any column. This guaranteed that any connections discovered were found using true values and therefore have more meaning.

| DEMAND.LOSS.MW | CUSTOMERS.AFFECTED | ANOMALY.LEVEL | TOTAL.SALES | CLIMATE.REGION     | MONTH | CAUSE.CATEGORY     | SEASONS |
|----------------|--------------------|---------------|-------------|---------------------|--------|---------------------|---------|
| 250            | 250,000             | 1.2           | 5,970,339     | East North Central  | 7      | severe weather      | Summer  |
| 75             | 300,000             | 0.2           | 5,607,498     | East North Central  | 6      | severe weather      | Summer  |
| 20             | 5,941               | 0.6           | 5,599,486     | East North Central  | 3      | intentional attack  | Spring  |
| 100            | 64,000              | -0.5          | 7,278,927     | Central             | 4      | severe weather      | Spring  |
| 300            | 63,000              | -0.5          | 7,278,927     | Central             | 4      | severe weather      | Spring  |

I trained Linear Regression, Feed-Forward Neural Network, and Random Forest Regressor models on this data and used 5-fold cross validation to select the best one. The best model was the Random Forest Regressor with a test **MAE** of 45.96 hours and an **R-Squared Score** of -4.46. This indicated that my model was approximately 46 hours off for outage duration and did not generalize to unseen data well at all. Given that most outages were less than 100 hours, my model is not fit to predict outage duration at all.

## Final Attempt
I tried something different with this attempt. I replaced the `DEMAND.LOSS.MW` and `CUSTOMERS.AFFECTED` column data for the `POPPCT_URBAN` and `POPDEN_URBAN` column data. Not only were there more rows of data to work with leading to better training, but also introduced a different angle into the possible influences of outage duration. I predicted that states with both a high urban population density and high urban population would have longer outage duration times since the grids have to support a high concentration of people in smaller spaces. Additionally, I transformed my prediction column(`OUTAGE.DURATION`) by taking the log base 2. This transformation helps normalize the right-skewed outage duration data. After dropping rows that had missing values, there were 1,265 samples to run regression on. A preview of the input parameter data is below. 


| ANOMALY.LEVEL | TOTAL.SALES | CLIMATE.REGION     | CAUSE.CATEGORY | POPDEN_URBAN | POPPCT_URBAN | SEASONS |
|---------------|--------------|---------------------|----------------|---------------|----------------|---------|
| -0.3          | 6,562,520    | East North Central  | severe weather | 2,279          | 73.27          | Summer  |
| -1.5          | 5,222,116    | East North Central  | severe weather | 2,279          | 73.27          | Fall    |
| -0.1          | 5,787,064    | East North Central  | severe weather | 2,279          | 73.27          | Summer  |
| 1.2           | 5,970,339    | East North Central  | severe weather | 2,279          | 73.27          | Summer  |
| -1.4          | 5,374,150    | East North Central  | severe weather | 2,279          | 73.27          | Fall    |


### Baseline Model
I initially trained a linear regression model using all the columns of the dataset. I transformed the categorical variables using a `OneHotEncoder`. The baseline model showed significant performance increases with the new parameter columns and transformation of the prediction data. The test MAE of the model was 1.66 log hours and the **R-Squared Score** was 0.32. To get the predicted outage duration in hours, we would raise 2 to our log outage duration. For instance, the test **MAE** in hours was approximately 3.16 hours, significantly better than before.

### Feature Generation
To make the final model more robust, I generated the following features.

| Feature Name | Description |
| :-------------------------- | :------------------------------|
| **Sales Urban** | `(TOTAL.SALES * POPPCT_URBAN)/100`. Weight the consumption of the state by its urban population percentage. States with high total consumption and urban population percentage should expect longer outages. Used a degree 3 `PolynomialFeatures` transformer to extrapolate different values and a `QuantileTransformer` to normalize the features to help my final model's performance.|
| **Scaled Anomaly Level** | Used a `StandardScaler` to scale the `ANOMALY.LEVEL` column to have a mean of 0 and a standard deviation of 1 to help the final model's performance. |
| **Polynomial Urban Population Density** | Used a degree 2 `PolynomialFeatures` transformer and `QuantileTransformer` on the `POPDEN_URBAN` column. |

### Final Model
The final model I chose was a Random Forest Regressor whose parameters were tuned using 5-fold cross validation by a `GridSearchCV` instance. The best parameters for the search and their definitions are below.

| Parameter | Definition |
|:--------- | :--------- |
|`max_depth`: 10 | The maximum depth of each tree. |
|`min_samples_split`: 8 | The minimum number of samples required to split an internal node. |
|`n_estimators`: 70 | The number of trees in the forest. |

With a test **MAE** of 1.57 log hours and a **R-Squared Score** of 0.39, the final model predicts log outage duration better and generalizes to unseen data better than the baseline model. Converting the **MAE** to hours, my model is typically only off by 2.97 hours when predicting outage duration, making it a decent candidate for regression imputation on similar data with missing outage duration values. While the test **R-Squared Score** is still less than 0.5, it is interesting to note that the training **R-Squared Score** was 0.69, meaning the final model generalized quite well to the variance of the training data. Given that there were only 948 samples out of the total 1265 available for training(around 75% of the total data), I believe a larger training dataset would improve the model's validation **R-Squared Score** significantly.