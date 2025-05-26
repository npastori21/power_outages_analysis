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
I was primarily focused on measuring the severity of outages by outage duration. Of the 1534 original datapoints, 244 rows of the dataframe either had `OUTAGE.DURATION` values that were or were less than 5 minutes. I chose to drop the rows that fit this criteria since outages under 5 minutes are trivial and I did not want to use imputed values for my regression model later in the analysis. This leaves us with 1281 datapoints to use for the initial analysis.

## Univariate Analysis
This bar chart shows the number of outages by climate region. The Northeast, and South have the most outages which could be due to more extreme weather conditions that knock out grids or cause outages due to increased demand for climate-controlling appliances.

## Bivariate Analysis
Below is a heatmap showing the average outage duration by states with at least one outage. The map provides more insight into the relationship of states with more extreme weather and the severity of their outages. For example, Wisconsin, Michigan, and New York have average outage durations of over 85 hours, which could be attributed to their harsh winter weather and storms. Additionally, states like Florida and Louisiana's outages average over 65 hours possibly due to their hot summers and exposure to harsh tropical storms.

## Interesting Aggregate
This pivot table shows the relationship between average outage duration by season and climate region. I originally hypothesised that colder months in colder climates and hotter months in hot climates would have longer outage durations. As you can see, norhtern regions like the Northeast and Northwest have higher outage durations during colder seasons like fall and winter while southern climate regions like the South, Southeast, and Southwest have higher outage durations during spring and summer.

<iframe src="plots/seasons_pivot.html" width="800" height="300"></iframe>