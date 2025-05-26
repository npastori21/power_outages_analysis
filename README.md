# Introduction
In this analysis, I evaluate a collection of power outages from 2000-2016 across the U.S. to determine what factors cause severe outages. While there are many questions to ask such as "Do socioeconomic factors affect the severity of outages?" or "Does the power consumption in the state affect outage severity?", I decided to answer the question "Is the severity of power outages affected by location?". Below are the relevant columns I chose for my analysis.

| Variable Name | Description |
| :---          | :---------- |
| `OUTAGE.DURATION` | Duration of outage in hours |
| `DEMAND.LOSS.MW` | The total or peak demand lost during an outage |
| `CUSTOMERS.AFFECTED` | Number of customers affected by the power outage event |
| `ANOMALY.LEVEL` | This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season |
| `TOTAL.SALES` | Total electricity consumption in the U.S. state (megawatt-hour) |
| `POSTAL.CODE` | Postal Code of the U.S. states |
| `CLIMATE.REGION` | U.S. Climate regions as specified by National Centers for Environmental Information |
| `CAUSE.CATEGORY` | Categories of all the events causing the major power outages |
| `MONTH` | Indicates the month when the outage event occurred |

# Data Cleaning and Exploratory Analysis

## Imputation of Values
I was primarily focused on measuring the severity of outages by outage duration. Of the 1534 original datapoints, 244 rows of the dataframe either had `OUTAGE.DURATION` values that were missing or were less than 5 minutes. I chose to drop the rows that fit this criteria since outages under 5 minutes are trivial and I did not want to use imputed values for my regression model later in the analysis. This leaves us with 1281 datapoints to use for the initial analysis.

## Univariate Analysis
This bar chart shows the number of outages by climate region. The Northeast, and South have the most outages which could be due to more extreme weather conditions that knock out grids or cause outages due to increased demand for climate-controlling appliances.
 <iframe
 src="plots/region_outages.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

## Bivariate Analysis
Below is a heatmap showing the average outage duration by states with at least one outage. The map provides more insight into the relationship of states with more extreme weather and the severity of their outages. For example, Wisconsin, Michigan, and New York have average outage durations of over 85 hours, which could be attributed to their harsh winter weather and storms. Additionally, states like Florida and Louisiana's outages average over 65 hours possibly due to their hot summers and exposure to harsh tropical storms.

 <iframe
 src="plots/states.html"
 width="1000"
 height="600"
 frameborder="0"
 ></iframe>

## Interesting Aggregate
This pivot table shows the relationship between average outage duration by season and climate region. I originally hypothesised that colder months in colder climates and hotter months in hot climates would have longer outage durations. As you can see, northern regions like the Northeast and Northwest have higher outage durations during the colder seasons of fall and winter. What's surpising is the fact that the South and Southeast regions have high average outage durations during the fall as well.

| ('OUTAGE.DURATION', 'Central') | ('OUTAGE.DURATION', 'East North Central') | ('OUTAGE.DURATION', 'Northeast') | ('OUTAGE.DURATION', 'Northwest') | ('OUTAGE.DURATION', 'South') | ('OUTAGE.DURATION', 'Southeast') | ('OUTAGE.DURATION', 'Southwest') | ('OUTAGE.DURATION', 'West') | ('OUTAGE.DURATION', 'West North Central') |
|------------:|------------:|------------:|------------:|------------:|------------:|------------:|------------:|------------:|
|                         67.5606 |                                     65.772 |                          93.1744 |                          40.4548 |                      106.331 |                          62.1304 |                          11.6857 |                      27.0896 |                                  1.39167 |
|                         44.4734 |                                    147.403 |                          39.8406 |                          13.7667 |                       27.458 |                          33.9304 |                          12.9403 |                      30.3234 |                                  0.933333 |
|                         34.1657 |                                     59.9082 |                          56.8113 |                          17.3439 |                       36.2646 |                          36.6167 |                          83.6962 |                      12.0482 |                                  1.42143 |
|                         73.2139 |                                    115.79  |                          74.2989 |                          43.8083 |                       68.1231 |                          24.9246 |                           9.48406 |                      46.0239 |                                 86       |


# Framing a Prediction Problem
To further investigate the connection of outage statistics and outage duration, I developed a regression problem between the columns of the outage dataset and outage duration. 

## Data Overview
Below is a preview of the columns I used to train my regression models. Due to there being over 500 missing values in the `DEMAND.LOSS.MW` column and over 300 missing values in the `CUSTOMERS.AFFECTED` column, imputation would not be feasible since it would bias the predictions of my models. Instead, I chose to investigate the relationship between outage duration and the features below on a subset of 490 rows that had no missing values for any column. This guarantees that any connections discovered are found using true values and therefore have more meaning.

| DEMAND.LOSS.MW | CUSTOMERS.AFFECTED | ANOMALY.LEVEL | TOTAL.SALES | CLIMATE.REGION     | MONTH | CAUSE.CATEGORY     | SEASONS |
|----------------|--------------------|----------------|-------------|--------------------|--------|---------------------|----------|
| 250            | 250000             | 1.2            | 5970339     | East North Central | 7      | severe weather      | Summer   |
| 75             | 300000             | 0.2            | 5607498     | East North Central | 6      | severe weather      | Summer   |
| 20             | 5941               | 0.6            | 5599486     | East North Central | 3      | intentional attack  | Spring   |
| 100            | 64000              | -0.5           | 7278927     | Central            | 4      | severe weather      | Spring   |
| 300            | 63000              | -0.5           | 7278927     | Central            | 4      | severe weather      | Spring   |

## Baseline Model
For my baseline model, I trained a Multiple Linear Regression model on the columns above. I used a one hot encoder to transform the `CLIMATE.REGION`, `MONTH`, `CAUSE.CATEGORY`, and `SEASONS` categorical columns. I did not modify any of the quantitative columns.