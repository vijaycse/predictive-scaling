# Predictive-Scaling

### Objective

We are paying lot of money just for Google cloud platform (cloud infrastructure to run our business) in our product space.  
We roughly run about ~500 instances (central and east) during normal traffic. 

Needless to say, the number of instances are super high during events such as (xbox launch go crazy with all BOTs traffic), 
gaming consoles etc. We scale up and down few times a month, sometimes weekly based on our perf tested scaling guidelines or for event hrs.



#### Input Dataset Analysis:

Order trend/Transaction per second for most US based retailers are very much predictable (except during event hrs and few days in Nov and December).  
In other words, less impacted by seasonality and the data is stationary almost. The following dataset of 2 yrs worth from one of the retailers data 
found in kaggle shows

  -  Input dataset [Dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-purchase-history-from-electronics-store)

![Typical Order Trend](https://github.com/vijaycse/predictive-scaling/blob/main/Order_Trend.png)



### Design Idea:

The idea is to look at the last 2 yrs data and run Machine Learning TimeSeries linear regression model 
to closely predict and auto scale hrly or every day based on predictions (numbers can be overridden for planned events) 
so that we can control the infra cost significantly. 


#### Design Overview:
  
![Design Overview](https://github.com/vijaycse/predictive-scaling/blob/main/Design_Overview_2.png)



#### Database:

The idea is to store the predicted order per hr in a simple DB like postgres and write a micro batch (scheduled every hr) to scan the 
the table to the read the predicted order the for the next hr ahead of time.


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
