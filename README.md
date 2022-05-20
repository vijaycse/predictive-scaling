# Predictive-Scaling

### Objective

We are paying lot of money just for Google cloud platform (cloud infrastructure to run our business) in our product space.  
We roughly run about ~500 instances aka machines (central and east) during normal traffic. 

Needless to say, the number of instances are super high during events such as Black friday or Cyber monday etc. 
We scale up and down few times a month, sometimes weekly based on our perf tested scaling guidelines or for event hrs.


#### Input Dataset Analysis:

Order trend/Transaction per second for most US based retailers are very much predictable (except during event hrs and few days in Nov and December).  
In other words, impacted by predicatable seasonality and data is relatievely less stationary . The following dataset of 2 yrs worth from one of the retailers data 
found in kaggle shows

  -  Input dataset [Dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-purchase-history-from-electronics-store)

![Typical Order Trend](https://github.com/vijaycse/predictive-scaling/blob/main/Order_Trend.png)



### Design Idea:

The idea is to look at the last 2 yrs data and run Machine Learning TimeSeries linear model to closely predict and auto scale hrly or every day 
based on predictions (numbers can be overridden for planned events) so that we can control the infra cost significantly. 


#### Design Overview:
  
![Design Overview](https://github.com/vijaycse/predictive-scaling/blob/main/Design_overview_3.png)



#### Database:

The idea is to store the predicted order per hr in a simple DB like postgres and write a micro batch (scheduled every hr) to scan the 
table to the read the predicted order the for the next hr ahead of time.


```
create table oph_forecast
(
   day_hr_ts timestamp,
   order_per_hr double precision, 
   manual_override boolean ,   // for manual override
   override_order_per_hr double   // manual input to override
);

```


#### Machine Learning:

 The ARIMA family of time series forecasting models will likely to predict the order per hour or Transaction per second closely as input dataset
 reveals that order trend and transaction trend is very much predicatable (with weekly seasonality).
 

#### Cloud Auto Scaler APIs

 Google Cloud  or Amazon web services has a provision to scale up or down the machines (EC2 instances) using APIs. The micro batch that reads the 
 predicted trend will scale up and down the machines hourly.
 
   - Considerations:
        - When the scale up and scale down numbers from the previous number of machines vary by more than 20% , the micro batch should scale up or down iteratively there by less impact to the guests.
        - there should be a provision to override the value for any planned events that way the predictive scaler will not scale up or down based on predictions.



### Why not Auto-scaling?

We scale ahead of time before the events or spike seasons (holiday seasons Nov-Dec) generally. Auto scaling is reactive in nature
and often times, its based on CPU and memory usage. 
Predictive scaling works based on TPS/order trend.Predictive scaling and auto scaling can co-exist.

### How predictive autoscaling works

Predictive autoscaler forecasts your scaling metric based on the metric's historical trends. Forecasts are recomputed every few minutes, which lets the autoscaler rapidly adapt its forecast to very recent changes in load. Predictive autoscaler needs at least 3 days of history from which to determine a representative service usage pattern before it can provide predictions. Compute Engine uses up to 3 weeks of your MIG's load history to feed the machine learning model.

Predictive autoscaler calculates the number of VMs needed to achieve your utilization target based on numerous factors, including the following:

- The predicted future value of the scaling metric
- The current value of the scaling metric
- Confidence in past trends, including past variability of the scaling metric
- The configured application initialization period, also referred to as the cool down period

Based on such factors, the predictive autoscaler scales out your group ahead of anticipated demand.

<img width="764" alt="Screen Shot 2022-05-20 at 4 43 07 PM" src="https://user-images.githubusercontent.com/4589748/169616420-69b29071-1094-472e-9fc3-84cf50770780.png">




#### Contribution:
 Currently only the author(Vijaycse) is planning to contribute to this project. It is possible to open it for others to contribute in the future.
