# Predictive-Scaling

### Objective

We are paying lot of money just for Google cloud platform (cloud infrastructure to run our business) in our product space.  
We roughly run about ~500 instances (central and east) during normal traffic. 

Needless to say, the number of instances are super high during events such as Black friday or Cyber monday etc. 
We scale up and down few times a month, sometimes weekly based on our perf tested scaling guidelines or for event hrs.


#### Input Dataset Analysis(Exploratory Data Analysis):

Order trend/Transaction per second for most US based retailers are very much predictable (except during event hrs and few days in Nov and December).  
In other words, less impacted by unpredictable seasonality . The following dataset of 1 yr worth from one of the retailers data found in kaggle shows

  -  Input dataset [Dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-purchase-history-from-electronics-store)

![Typical Order Trend](https://github.com/vijaycse/predictive-scaling/blob/Dev/Order_Trend.png)


### Design Idea:

The idea is to look at the last 2 yrs data and run Machine Learning TimeSeries linear model to closely predict and auto scale hrly or every day 
based on predictions (numbers can be overridden for planned events) so that we can control the infra cost significantly. 


#### Design Overview:
  
![Design Overview](https://github.com/vijaycse/predictive-scaling/blob/main/Design_overview_3.png)



#### Database:

The idea is to store the predicted order per hr in a simple DB like postgres and write a micro batch (scheduled every hr) to scan the 
table to the read the predicted order the for the next hr ahead of time.

##### Entity Relationship Diagram:
 
![Screen Shot 2022-05-23 at 8 51 22 PM](https://user-images.githubusercontent.com/4589748/169932548-42a037bc-80d4-48b6-b207-bc2bb46acc1a.png)

We have two tables one to store `order_per_hr(oph)` and another one `order_per_hr_forecast(oph_forecast)` to store the predicated order_per_hr where `day_hr_ts` is a primary key to join them

```
create table oph
(
   day_hr_ts timestamp,
   order_per_hr integer
);

```

```
create table oph_forecast
(
   day_hr_ts timestamp,
   order_per_hr integer, 
   manual_override boolean ,   // for manual override
   override_order_per_hr integer   // manual input to override
);

```


#### Machine Learning:

 The ARIMA family of time series forecasting models will likely to predict the order per hour or Transaction per second closely as input dataset
 reveals that order trend and transaction trend is very much predicatable (with weekly seasonality).
 
 ##### SARIMA
   Seasonal Autoregressive Integrated Moving Average models are the extension of the ARIMA model 
   that supports uni-variate time series data involving backshifts of the seasonal period

     - The data is not stationary so used shifting and logging technique to smooth the seasonality
     - Tested with Adfuller test to find out the p-value
     - Trained the model using best sarimax and found out (p,d,q)1,1,1 and 0,1,1,24 are the best possible combinations
  
    
   
   Results:
     
   SARIMAX-FORECAST:
    
   ![SARIMAX-FORECAST](https://github.com/vijaycse/predictive-scaling/blob/main/images/SARIMAX_FORECAST_RESULTS.png)
   
   
   SARIMAX-FORECAST-RESULTS:
    
   ![SARIMAX-FORECAST-RESULTS](https://github.com/vijaycse/predictive-scaling/blob/main/images/SARIMAX_RESULTS.png)
   
    
 ##### Conclusion:
 
    SARIMA's forecasting was not good enough for us to consider.


##### Prophet
  Prophet is a procedure for forecasting time series data based on an additive/multiplicative model where non-linear trends are fit with yearly, weekly, and daily seasonality, plus holiday effects.It works best with time series that have strong seasonal effects and several seasons of historical data.Prophet is robust to missing data and shifts in the trend, and typically handles outliers well.

  Trend parameters:
    growth: 'linear' 
  
  Seasonality and Holiday Parameters:

    yearly_seasonality: Auto seasonality
    weekly_seasonality: Auto seasonality
    daily_seasonality: Auto seasonality
    holidays: Standard US holidays

  Prophet requires the variable names in the time series to be:
  
    y – Target
    ds – Datetime
    

  #### Results:

   Prophet-Forecasting:

   ![Prophet-Forecasting](https://github.com/vijaycse/predictive-scaling/blob/main/images/Prophet_results.png)
   
   
   Prophet-Measures:
   
   ![Prophet-Measures](https://github.com/vijaycse/predictive-scaling/blob/main/images/Prophet_measures.png)
   
   
   Prophet-Variation:
   
   ![Prophet-Variation](https://github.com/vijaycse/predictive-scaling/blob/main/images/Prophet_variation_results.png)
   
   
  
  ###### Conclusion:
  
   Prophet results are much better than SARIMAX and hence choosing prophet prediction with buffer to make a suggestion for predictive scaling



#### Cloud Auto Scaler APIs

 Google Cloud  or Amazon web services has a provision to scale up or down the machines (EC2 instances) using APIs. The micro batch that reads the 
 predicted trend will scale up and down the machines hourly.
 
   - Considerations:
        - When the scale up and scale down numbers from the previous number of machines vary by more than 20% , the micro batch should scale up or down iteratively there by less impact to the guests.
        - there should be a provision to override the value for any planned events that way the predictive scaler will not scale up or down based on predictions.

##### Companion Repo:

   [Predictive-Scaler][https://github.com/vijaycse/predictive-autoscaler]


#### Contribution:
 Currently only the author(Vijaycse) is planning to contribute to this project. It is possible to open it for others to contribute in the future.
 
#### Presentation:
 [Presentation](https://docs.google.com/presentation/d/1NQZ56vdSED6xQISN1o39OoY3Ud5eo83k3olHIZJETts/edit#slide=id.p1)

#### Visualization:

Actual VS Prediction:

![Actual VS Prediction](https://github.com/vijaycse/predictive-scaling/blob/main/images/ActualVsPrediction.png)

Prediction Insights:

![Prediction Insights](https://github.com/vijaycse/predictive-scaling/blob/main/images/prediction_insights.png)

Cost Savings:

![Cost Savings](https://github.com/vijaycse/predictive-scaling/blob/main/images/Cost_Saving_Monthly.png)
