# Regional Power Outage Classification 

By: Nikitha Kerudi 


## Introduction

    

This project investigates a dataset of major power outages in the continental United States collected between January 2000 and July 2016. Each row represents a single major power outage event effecting exactly one U.S. state at the time of occurance. The original dataset from [https://engineering.purdue.edu/LASCI/research-data/outages](https://engineering.purdue.edu/LASCI/research-data/outages) contianed 55 columns and 1534 rows (excluding the empty "columns" and "rows" in excel before importing it as a pd.DataFrame object in VSCode) and provides information about when outages occured, how severe it was, and characteristics of the affected region such as climate, population density, and electricity consumption. The full dataset's columns list can be found at this link : [https://www.sciencedirect.com/science/article/pii/S2352340918307182](https://www.sciencedirect.com/science/article/pii/S2352340918307182). After exploring this data thoroughly, some of the main ideas that posed meaninful questions and analyses that could be of potential benefit to society when dealing with major outage events surrounded the cause of the outages. I was intrigues as to what exactly causes outages, identifying how and in what frequency each major power outage cause occurs, and thus how these outages can be prevented. In a day and age where we as a society are crucially dependent on electricity, though not everyone has fair and equal access to it, these major power outages can be a huge disturbance for everyday individuals and a huge waste of capital and labor for businesses, making this a worthwhile phenomena to investigate and get a step closer to preventing. My overarching research question for this project is as stated below:

### **Question**

### **Which characteristics best predict the cause of a major power outage?**

----------

## **2. Relevant Columns to Power Outage Cause**

Bellow are the initial columns I investigated that were potentially relevant to predicting (or more specifically, classifying) the cause of major power outages. 

**General Information (Where)**

- YEAR : Indicates the year when the outage event occurred

- MONTH : Indicates the month when the outage event occurred

- U.S._STATE : Represents all the states in the continental U.S.

- POSTAL.CODE : Represents the postal code of the U.S. states

- NERC.REGION : The North American Electric Reliability Corporation (NERC) regions involved in the outage event


**Outage Events Information (When)**
- OUTAGE.START.DATE : 	This variable indicates the day of the year when the outage event started (as reported by the corresponding Utility in the region)

- OUTAGE.START.TIME : This variable indicates the time of the day when the outage event started (as reported by the corresponding Utility in the region)

- OUTAGE.RESTORATION.DATE : This variable indicates the day of the year when power was restored to all the customers (as reported by the corresponding Utility in the region)

- OUTAGE.RESTORATION.TIME : This variable indicates the time of the day when power was restored to all the customers (as reported by the corresponding Utility in the region)


**Cause of the Event**
- CAUSE.CATEGORY : 	Categories of all the events causing the major power outages

- CAUSE.CATEGORY.DETAIL : 	Detailed description of the event categories causing the major power outages

- HURRICANE.NAMES : If the outage is due to a hurricane, then the hurricane name is given by this variable


**Extent of Outages**
- OUTAGE.DURATION : Duration of outage events (in minutes)

- DEMAND.LOSS.MW : Amount of peak demand lost during an outage event (in Megawatt) [but in many cases, total demand is reported]

- CUSTOMERS.AFFECTED : Number of customers affected by the power outage event


**Regional Climate Information**
- CLIMATE.REGION : 	U.S. Climate regions as specified by National Centers for Environmental Information (nine climatically consistent regions in continental U.S.A.)

- ANOMALY.LEVEL : This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season. It is estimated as a 3-month running mean of ERSST.v4 SST anomalies in the Niño 3.4 region (5°N to 5°S, 120–170°W)

- CLIMATE.CATEGORY : This represents the climate episodes corresponding to the years. The categories—“Warm”, “Cold” or “Normal” episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)


**Population/Regional/Other Possibly Relevant Columns** 
- TOTAL.SALES : Total electricity consumption in the U.S. state (megawatt-hour) 

- TOTAL.CUSTOMERS : Annual number of total customers served in the U.S. state

- TOTAL.PRICE : Average monthly electricity price in the U.S. state (cents/kilowatt-hour)

- PCT_LAND	: Percentage of land area in the U.S. state as compared to the overall land area in the continental U.S. (in %)

- POPPCT_URBAN	: Percentage of the total population of the U.S. state represented by the urban population (in %)

- POPDEN_URBAN	: Population density of the urban areas (persons per square mile)

    

----------

# Data Cleaning and EDA

### **Data Cleaning**

The raw outage dataset required several preprocessing steps before meaningful analysis was possible. First, I removed formatting artifacts from the Excel file and kept only the columns relevant to predicting outage causes. The original dataset separates outage start and restoration times into date and time components, so I combined these pairs into unified OUTAGE.START and OUTAGE.RESTORATION Timestamp columns. This transformation matches the data generating process: utilities record start and end times separately, but analysis requires a single timestamp variable.

Next, I examined several outcome variables — OUTAGE.DURATION, CUSTOMERS.AFFECTED, and DEMAND.LOSS.MW — and found many entries recorded as 0. Since a major outage cannot reasonably have zero duration, zero customers affected, or zero megawatts of lost demand, these zeros likely reflect unreported or missing values. I therefore replaced 0 with NaN in those columns to more accurately distinguish true values from missing data.

To capture regional population structure, I normalized and combined three related variables (POPPCT_URBAN, POPDEN_URBAN, and AREAPCT_URBAN) into a single URBAN index. This index reflects how urbanized each state is relative to others in the dataset. This engineered feature simplifies analysis by reducing three correlated variables into one interpretable metric that facilitates both visualization and later modeling.

I also extracted temporal information from timestamps by creating START_YEAR and START_MONTH, dropped unnecessary identifiers (POSTAL.CODE, PCT_LAND), and standardized formatting for categorical columns by stripping whitespace and lowercasing text. Finally, I created a binary column, IS_WEATHER, indicating whether the outage was caused by severe weather-related conditions. This variable is helpful in understanding broad trends in outage causes.

The head of the final cleaned DataFrame is shown below, split up to give a visual depiction of all the columns:

|   YEAR |   MONTH | U.S._STATE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   |
|-------:|--------:|:-------------|:--------------|:-------------------|----------------:|:-------------------|
|   2011 |       7 | Minnesota    | MRO           | East North Central |            -0.3 | normal             |
|   2014 |       5 | Minnesota    | MRO           | East North Central |            -0.1 | normal             |
|   2010 |      10 | Minnesota    | MRO           | East North Central |            -1.5 | cold               |
|   2012 |       6 | Minnesota    | MRO           | East North Central |            -0.1 | normal             |
|   2015 |       7 | Minnesota    | MRO           | East North Central |             1.2 | warm               |
|   2010 |      11 | Minnesota    | MRO           | East North Central |            -1.4 | cold               |
|   2010 |       7 | Minnesota    | MRO           | East North Central |            -0.9 | cold               |
|   2005 |       6 | Minnesota    | MRO           | East North Central |             0.2 | normal             |
|   2015 |       3 | Minnesota    | MRO           | East North Central |             0.6 | warm               |
|   2013 |       6 | Minnesota    | MRO           | East North Central |            -0.2 | normal             |

| CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   HURRICANE.NAMES |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |
|:-------------------|:------------------------|------------------:|------------------:|-----------------:|---------------------:|
| severe weather     | nan                     |               nan |              3060 |              nan |                70000 |
| intentional attack | vandalism               |               nan |                 1 |              nan |                  nan |
| severe weather     | heavy wind              |               nan |              3000 |              nan |                70000 |
| severe weather     | thunderstorm            |               nan |              2550 |              nan |                68200 |
| severe weather     | nan                     |               nan |              1740 |              250 |               250000 |
| severe weather     | winter storm            |               nan |              1860 |              nan |                60000 |
| severe weather     | tornadoes               |               nan |              2970 |              nan |                63000 |
| severe weather     | thunderstorm            |               nan |              3960 |               75 |               300000 |
| intentional attack | sabotage                |               nan |               155 |               20 |                 5941 |
| severe weather     | hailstorm               |               nan |              3621 |              nan |               400000 |

|   TOTAL.PRICE |   TOTAL.SALES |   TOTAL.CUSTOMERS |   POPPCT_URBAN |   POPDEN_URBAN |   AREAPCT_URBAN | OUTAGE.START        |
|--------------:|--------------:|------------------:|---------------:|---------------:|----------------:|:--------------------|
|          9.28 |       6562520 |           2595696 |          73.27 |           2279 |            2.14 | 2011-07-01 17:00:00 |
|          9.28 |       5284231 |           2640737 |          73.27 |           2279 |            2.14 | 2014-05-11 18:38:00 |
|          8.15 |       5222116 |           2586905 |          73.27 |           2279 |            2.14 | 2010-10-26 20:00:00 |
|          9.19 |       5787064 |           2606813 |          73.27 |           2279 |            2.14 | 2012-06-19 04:30:00 |
|         10.43 |       5970339 |           2673531 |          73.27 |           2279 |            2.14 | 2015-07-18 02:00:00 |
|          8.28 |       5374150 |           2586905 |          73.27 |           2279 |            2.14 | 2010-11-13 15:00:00 |
|          9.12 |       6374935 |           2586905 |          73.27 |           2279 |            2.14 | 2010-07-17 20:30:00 |
|          7.36 |       5607498 |           2474912 |          73.27 |           2279 |            2.14 | 2005-06-08 04:00:00 |
|          9.03 |       5599486 |           2673531 |          73.27 |           2279 |            2.14 | 2015-03-16 07:31:00 |
|         10    |       5490631 |           2622305 |          73.27 |           2279 |            2.14 | 2013-06-21 17:39:00 |


| OUTAGE.RESTORATION   |   POPPCT_URBAN_norm |   POPDEN_URBAN_norm |   AREAPCT_URBAN_norm |    URBAN |   START_YEAR |
|:---------------------|--------------------:|--------------------:|---------------------:|---------:|-------------:|
| 2011-07-03 20:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2011 |
| 2014-05-11 18:39:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2014 |
| 2010-10-28 22:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2010 |
| 2012-06-20 23:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2012 |
| 2015-07-19 07:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2015 |
| 2010-11-14 22:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2010 |
| 2010-07-19 22:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2010 |
| 2005-06-10 22:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2005 |
| 2015-03-16 10:06:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2015 |
| 2013-06-24 06:00:00  |            0.564232 |            0.121337 |            0.0209105 | 0.235493 |         2013 |

|   START_MONTH |   IS_WEATHER |
|--------------:|-------------:|
|             7 |            1 |
|             5 |            0 |
|            10 |            1 |
|             6 |            1 |
|             7 |            1 |
|            11 |            1 |
|             7 |            1 |
|             6 |            1 |
|             3 |            0 |
|             6 |            1 |


----------

### **Univariate Analysis**

#### **Distribution of Outage Causes**

A bar chart of CAUSE.CATEGORY shows that severe weather overwhelmingly dominates all other outage causes. This reinforces the importance of including climate and seasonal features in later modeling.

<iframe
  src="plots/univan-dist-cause.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>


#### **Distribution of Urbanization Index**

A histogram of the URBAN score reveals that many outages occur in less urbanized (more rural) regions. This suggests that rural infrastructure may be more vulnerable, particularly to weather-related outages.

<iframe
  src="plots/univan-dist-urban.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>


----------

### **Bivariate Analysis**

#### **Outage Causes by Month**

A heat-map of outage count by month and cause category shows clear seasonality: severe weather events peak during summer months, while intentional attacks and equipment failures do not show strong monthly patterns. This indicates that temporal features may help distinguish between weather-related and non-weather-related outages.

<iframe
  src="plots/bivan-outage-by-mo.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>

#### **Outage Causes by Climate Region**

Another heat-map comparing climate regions to outage causes shows that severe weather events are concentrated in particular regions, especially the Southeast and Northeast. This pattern supports the hypothesis that geography strongly influences outage cause.

<iframe
  src="plots/bivan-outage-by-climate.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>



----------

### **Interesting Aggregates**

Finally, I constructed a pivot table aggregating outage counts by climate region and cause category. This table highlights which regions experience which types of outages most frequently. For example, severe weather dominates in the Southeast, while equipment failures are more common in the Northeast. These aggregates help contextualize the patterns seen in the heatmaps.

| CLIMATE.REGION     |   equipment |        fuel |   intentional |   islanding |   public |    severe |        system |
|                    |     failure |      supply |        attack |             |   appeal |   weather |   operability |
|                    |             |   emergency |               |             |          |           |    disruption |
|:-------------------|------------:|------------:|--------------:|------------:|---------:|----------:|--------------:|
| Central            |           7 |           4 |            38 |           3 |        2 |       135 |            11 |
| East North Central |           3 |           5 |            20 |           1 |        2 |       104 |             3 |
| Northeast          |           5 |          14 |           135 |           1 |        4 |       176 |            15 |
| Northwest          |           2 |           1 |            89 |           5 |        2 |        29 |             4 |
| South              |          10 |           7 |            28 |           2 |       42 |       113 |            27 |
| Southeast          |           5 |           0 |             9 |           0 |        5 |       118 |            16 |
| Southwest          |           5 |           2 |            64 |           1 |        1 |        10 |             9 |
| West               |          21 |          17 |            31 |          28 |        9 |        70 |            41 |
| West North Central |           1 |           1 |             4 |           5 |        2 |         4 |             0 |



----------

# **Assessment of Missingness**

### **NMAR Analysis**

One column in the dataset that is very likely to be NMAR (Not Missing At Random) is CUSTOMERS.AFFECTED. The number of customers impacted by a major outage should always be known by a reporting utility, but nearly 43% of entries are missing. This pattern suggests that missing values occur when the utility chose not to report the number, perhaps because the outage affected very few customers or occurred in a location where customer counts are difficult to obtain. In other words, the reason the value is missing is directly tied to the unobserved true value itself, which is the definition of NMAR. To determine whether this could instead be MAR, we would need additional information such as the identity of the reporting utility, their reporting requirements, or metadata describing why some outages were documented incompletely. Without this supplementary information, the missingness mechanism remains NMAR.

----------

### **Missingness Dependency Analysis**

For this section, I focused on the missingness of OUTAGE.DURATION, a column that had a large amount of missing data found during the initial EDA portion. I conducted permutation tests to determine whether duration missingness depends on other variables.

----------

### **Test 1: Dependency on CAUSE.CATEGORY**

**Null Hypothesis (Ho):** The distribution of CAUSE.CATEGORY is the same when OUTAGE.DURATION is missing vs not missing.  
**Alternative Hypothesis (Ha):** The distribution differs.

**Observed TVD:** ~ 0.46  
**p_value:** ~ 0.0

This extremely small p_value indicates that the observed TVD is far outside the permutation distribution. I reject the null hypothesis and conclude that missing outage durations depend strongly on the cause of outage.

<iframe
  src="plots/missingness-perm.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>

----------

### **Test 2: Dependency on MONTH**

**Null Hypothesis (Ho):** The distribution of MONTH is the same for missing and non-missing duration values.  
**Alternative Hypothesis (Ha):** The distributions differ.

**Observed TVD:** ~ 0.1434  
**p_value:** ~ 0.1815

Because the p-value is not significant, I fail to reject the null hypothesis. This indicates that missing duration values do not depend on the month in which the outage occurred.

<iframe
  src="plots/missingness-perm-2.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>

----------

# **Hypothesis Testing**

### **Does climate region distribution differ between severe weather outages and equipment failure outages?**

To further develop my understanding of which features are associate with major power outage causes, I conducted a hypothesis test examining whether severe weather outages occur in different climate regions than equipment failure outages. Climate region is an important contextual factor that may influence outage risk, thus determining whether the distribution of outages differs by cause can help validate this idea.

**Null Hypothesis (Ho):** The distribution of climate regions is the same for severe weather outages and equipment failure outages.  
**Alternative Hypothesis (Ha):** The distribution of climate regions is different.

This is a two sided test.

Because climate region is a categorical variable, I used Total Variation Distance (TVD) as my test statistic with a significance level of a = 0.05. To carry out the permutation test, I restricted the dataset to outages by either severe weather or equipment failures, then calculated the TVD between the climate region distributions. I randomly permuted the "CAUSE.CATEGORY" labels 3,000 times and recomputed TVD.

**P_value:** ~ 0.0  
**Observed TVD:** ~ 0.367

This suggests that climate region is meaningfully related to the cause of major power outages.

<iframe
  src="plots/hyp-perm-test.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>

----------

# **Framing a Prediction Problem**

The prediction problem I aim to solve is:

### **"Given the information available at the start of a major power outage, can we predict the cause of the power outage?"**

### **Response Variable: `CAUSE.CATEGORY`**

1.  **Type:** This would be a classification model, used to predict the categorical "cause" of a given major power outage
    
2.  **Class Structure:** Multi-Class Classification Model - Since there are a variety of main causes for major power outages in our data, this would be a multi-class classification model with the options of ['severe weather', 'intentional attack', 'system operability disruption', 'equipment failure','public appeal', 'fuel supply emergency', 'islanding']. 
    
3.  **Why this variable?** I chose this response variable because it directly connects to the overarching theme of this analysis which is seeing WHY major outages occur. 
    
4.  **Metric:** The metric I'll be using to quantify the outcome of this model is F1-Score (weighted). I decided upon F1-score as opposed to accuracy since given out initial EDA steps, we found that the "sever weather" class dominates the rest, thus we may get mislead by the accuracy score as a metric.
    

### **Features available at the time of prediction:**

-   `"U.S._STATE"`, `"CLIMATE.REGION"`, `"NERC.REGION"`
    
-   `"YEAR"`, `"MONTH"`, `"OUTAGE.START"`, `"START.MONTH"`, `"START.YEAR"`, `"START.SEASON"`
    
-   `"URBAN"`, `"TOTAL.SALES"`, `"TOTAL.CUSTOMERS"`, `"TOTAL.PRICE"`
    

### **Features NOT available at prediction time:**

-   `"OUTAGE.DURATION"`
    
-   `"OUTAGE.END"`
    

----------

# **Baseline Model**

### **Baseline Model: RandomForestClassifier**


To establish a reference point for predicting the major power outage cause, I created a Baseline Model that uses only a subset of features which are simple, interpretable, and available at the "time of prediction" as described when framing this prediction problem. The features included in this baseline model are as follows :

  
  

Features: 

**NERC.REGION (nominal -> one hot encoding)**

NERC.REGION : for this nominal feature, I encoded it using the OneHotEncoder from sklearn.preprocessing and ColumnTransformer from sklearn.compose within my pipeline

**CLIMATE.REGION (nominal -> one hot encoding)**

CLIMATE.REGION : for this nominal feature, I encoded it using the OneHotEncoder from sklearn.preprocessig and ColumnTransformer from sklearn.compose within my pipeline

**YEAR (quantitative)**

YEAR : quantitative column already transformed into dt.year during preprocessing (outage start year)

**MONTH (ordinal -> already ordinally encoded!)**

MONTH : ordinal column already ordinaly encoded during preprocessing using dt.month (outage start month)



  
For the model, I used a RandomForestClassifier with class_weight="balanced" to handle the strong class imbalance observed in Step 2 and default hyperparameters otherwise, since this is only a baseline. A random forest is a reasonable choice here because it handles nonlinear interactions automatically, works well with mixed categorical/quantitative features, and it establishes a strong benchmark that future engineered features must outperform.

  

## Model Performance: 
Using a 80/20 train test spit, this model had a **Weighted F1 score of 0.592**. Though this score is quite low, it is indicitive of other features that may be worthy to note. Using seasonal and regional features suggests that those are of some significance when classifying the cause of major power outages, but that there are OTHER features and variables that contribute to the cause that we can now re-explore moving forward to the final model.

  Weighted F1-score : 0.592

----------

# **Final Model**


Final Model Features To Add:

  

**1. START.SEASON** - nominal:  As "severe weather" is the dominant class we've seen in our training data, we know that changes in weather often occur seasonally/quarterly, making this column potentially indicative of the cause of the major power outage, this feature will be engineering using **MONTH**. (If we know that the outage occurred in a season where sever weather occurs more in particular reasons, that could drive our prediction!)

Transformation: This is a nominal feature (since seasons don't really have a order as we aren't looking at this chronologically) so we'll be one-hot encoding this using OneHotEncoder and QuantileTransformer from sklearn.preprocessing.

  

MONTH → START.SEASON (0 = Winter, 1 = Spring, 2 = Summer, 3 = Fall) via a FunctionTransformer

  
  

**2. URBAN_CAT ** - nominal :  This feature is already engineered from the original "POPPCT_URBAN", "POPDEN_URBAN", and "AREAPCT_URBAN" columns, which we normalized and combined into one normalized "URBAN" score telling us how rural or urban a particular region is. This is being added as a feature because it could be potentially indicative of weather related outages, occuring in more rural regions, and attack/overload related outages may occur in more densly populated urban cities.

  

Transformation: This is a numerical feature, since we've already engineered this feature into a normalized score, however analytically thinking about this feature, it would be of more use to our model as a categorical feature (categorized as rural, mixed (in between rural and urban), and urban) since an abstract continuous index doesn't tell us as much GEOGRAPHICALLY and may cause our model to overfit due to the precision of the feature. To transform this continous quantitative variable into more of a numerical categorical feature, we'll be one-hot encoding it using KBinsDiscretizer from sklearn.preprocessing.

  

Note: You may be wondering, why not just make this a binary categorical feature, using Binarizer and setting a urban/rural "threshold"? The answer to this is quite simple in that from exploring the data, we know that many regions that are being used to train our data AND many regions GENERALLY speaking are not just either rural or urban, but rather something in between, thus having another category to account for that will make this model more generalizable to unseen data.

  

**3. ANOMOLY.LEVEL** - quantitative : This is a quantitative variable that helps capture climate disruptions (from El Nino / La Nina cycles) from the Oceanic Nino Index. I included this feature the most dominant class, severe weather, is affected by the increased winter storms, thunderstorms, droughts, hurricanes, etc. caused regionally by these climate disruptions which could be indicative of the true cause. Because “severe weather” is the dominant outage cause in the dataset, incorporating a predictor directly linked to yearly and seasonal climate anomalies helps the model detect when a weather-related cause is more likely.

  

**4. CUSTOMERS.AFFECTED **- quantitative : Although we cannot use outage duration or end-of-event information (those occur after the outage starts), **CUSTOMERS.AFFECTED** is often known early and utilities generally know how many customers go offline as soon as the outage begins. This feature was included because the scale of the power outage could also be indicative of the cause. For example severe weather outages would typically have a larger value for the customers affected then perhaps an equipment failure outage that which may only affect a few buildings.

  

**5. NERC.REGION** - nominal : Nerc regions represent the different infrastures affected by major power outages which is indicative of factors like the age and quality of equipment, typical weather patterns, regulatory practices, etc. This feature encodes infrastucture and operational context helping the model identify causes more precisely.

  

**6. CLIMATE.REGION** - nominal : Unlike Nerc regions which distinguish more electrical grid jurisdiction, climate regions are better for capturing the geographical and meteorological aspects of certain regions which is crucial for predicting whether or not a certain power outage is more likely to have been caused by severe weather for example we know the South is more hurricane prone while the Northeast is more prone to heaver winter storms.

  

### Modeling Algorithm:

  

I chose to use a RandomForestClassifier for my final model (as used in my baseline model as well) for multi-class classification of the predicted cause of major power outages. The reason I used RandomForestClassifier as opposed to a DecisionTreeClassifier (also a valid classification model) is because RandomForestClassifiers tend to just be better at predicting, there is more room for complex decision boundaries for power outages with multiple causes, they are more robust to imbalanced classes (which we know we have with severe weather being the dominant class), and they handle non linear interactions between features naturally. One singular tree won't be able to capture all aspects of interaction like **CLIMATE.REGION x URBAN_CAT**. Using a DecisionTree would drastically increase the potential for overfitting which we want to avoid for generalizability, thus handling a noisier/messy environment (like this one) better for more stable and accurate predictions.

  

### Hyperparameters:

  

Using a RandomForrestClassifier, the hyperparameters being fine tuned are max_depth (the maximum depth of any decision tree generated in the forest) to identify the optimal level of complexity needed to minimize train and mostly testing error, min_samples_split (minimum samples needed in a node for it to be split) to reduce overly specific fits, and n_estimators (number of trees in the forest, often larger values are better) to ensure stability and accuracy. Below is each hyperparameter being tested and the values passed into GridSearchCV to identify the most optimal combination of hyperparameters:

  

**max_depth : [10, 20, 30, None]**

**min_samples_split : [2, 5, 10, 20]**

**n_estimators : [150, 250, 350, 450]**

  

These were derived from trial and error alongside an analysis of the amount fo data we had to determine what was feasible and made sense for classification.

  

### Best Hyperparameters:

 {
 'model__max_depth': 20, 
 'model__min_samples_split': 5, 
 'model__n_estimators': 450
 }

  

The Final Model improved performance by over 42% absolute F1. This improvement is substantial and reflects the added value of engineered features, discretization of the **URBAN** index, and hyperparameter tuning. The Final Model's score is 0.844 compared to the Baseline Model's 0.592 which is a significant improvement!

  

Specifically:

  

**START.SEASON** captured stronger seasonal patterns in severe weather outages

  

**URBAN_CAT** allowed the model to better distinguish rural vs. urban outage characteristics

  

**ANOMALY.LEVEL** and **CUSTOMERS.AFFECTED** added climate and event-scale context

Below is a confusion matrix outlining the accuracy of our Final Model's predictions for each category : 

<iframe
  src="plots/category-cm.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>

Below is a confusion matrix outlining the accuracy of our Final Model's predictions overall : 

<iframe
  src="plots/overall-cm.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>


----------

# **Fairness Analysis**

### **Groups Compared**

-   **Urban** (URBAN_BIN = 0)
    
-   **Rural** (URBAN_BIN = 1)

To analyze the fairness of the RandomForrestClassifier above, I've created two groups, one for "urban" areas and one for "rural" areas as the most meaninful comparison for major power outages is in regards to geographical exposure to environmental risks. These groups will be generated from the engineered column "URBAN" containing a normalized score for the area each power outage occurred, creating a categorical column using the median urban score as the threshold, quantifying whether an area is more rural or densely populated and urban. We did use a similarly engineered column in our model as a feature, the URBAN_CAT column, categorizing regions as urban, mixed, or rural, however for the pruposes of fairness, using a binary separation may be more helpful. Since urban and rural areas tend to differ in infrastructure,
weather exposure, and reporting, there may be fairness disparities here worth investigating.
    

### **Metric:** Weighted Precision

The fairness metric being used here is precision (weighted) as this model is a multi-class classification model with imbalances classes. Weighted precision would be the most useful metric for evaluating the fairness of this model because it reflects how often the prediction are correct, taking into account the major class imbalance.

### **Hypotheses**

-   **Ho:** The model is fair and its weighted precision for rural and urban regions is the same, any observed difference is due to random chance.
    
-   **Ha:** The model is unfair meaning its weighted precision for rural regions is lower then its weighted precision for urban regeions.
    

### **Test Statistic**

  

The test statistic we'll be using here, since we are trying to explore if rural regions have a lower weighted precision than urban, is the difference in urban and rural weighted precisions. A positive test statistic here would be indicitive of our alternative hypothesis above which we'll be testing using the significance level a = 0.05. To test for fairness, I performed a permutation test. I kept the model's predictions fixed and repeatedly permuted (shuffled) the URBAN_BIN group labels. For each permutation, I recomputed the difference in weighted precision between the two groups in order to identify if our observed value was statistically significant or not.

`T = precision_urban - precision_rural`

`α = 0.05` 


### **Results**

-   **Observed T:** ~ -0.0416
    
-   **p_value:** ~ 0.9996

<iframe
  src="plots/fairness-perm.html"
  width="850"
  height="650"
  frameborder="0"
></iframe>
    

### **Conclusion**

After running this test, our observed statistic for T = precision_urban - precision_rural was actually a negative value of ~ -0.0416, and our final p_value after running a permutation test shuffling URBAN_BIN labels was actually ~ 0.9996 meaning we fail to reject the null hypothesis. Because the p_value is extremely large compared to our significance level, there is no statistical evidence that the model is less precise for rural regions than for urban regions. We can actually go as far to suspect (non conclude, suspect) that the opposite may be happening and that the model may tend to favor rural regions. Overall, based on this analysis, I do not find the evidence that the classifier exhibits a fairness disparity between urban and rural outages. The model's performance appears statistically indistinguishable across these two groups.

There is **no statistical evidence** that the model performs worse for rural regions. 
 
We fail to reject the null hypothesis and conclude the model exhibits **no fairness disparity** across these groups.
