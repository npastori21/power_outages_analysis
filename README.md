## Introduction
In this analysis, I evaluate a collection of power outages from 2000-2016 across the U.S. to determine what factors cause severe outages. While there are many questions to ask such as "Do socioeconomic factors affect the severity of outages?" or "Does the power consumption in the state affect outage severity?", I decided to answer the question "Is the severity of power outages affected by location?". Below are the relevant columns I chose for my analysis.

| Variable Name | Description |
| :---          | :---------- |
| `OUTAGE.DURATION` | Duration of outage in hours |
| `DEMAND.LOSS.MW` | The total or peak demand lost during an outage |
| `CUSTOMERS.AFFECTED` |  |
| `ANOMALY.LEVEL` | This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season |
| `TOTAL.SALES` | Total electricity consumption in the U.S. state (megawatt-hour) |
| `POSTAL.CODE` | Postal Code of the U.S. states |
| `CLIMATE.REGION` | U.S. Climate regions as specified by National Centers for Environmental Information |
| `CAUSE.CATEGORY` | Categories of all the events causing the major power outages |
| `MONTH` | Indicates the month when the outage event occurred |



