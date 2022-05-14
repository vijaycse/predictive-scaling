# Predictive-Scaling

### What

We are paying lot of money just for Google cloud platform (cloud infrastructure to run our business) in our product space.  
We roughly run about ~500 instances (central and east) during normal traffic. 

Needless to say, the number of instances are super high during events such as (xbox launch go crazy with all BOTs traffic), 
gaming consoles etc. We scale up and down few times a month, sometimes weekly based on our perf tested scaling guidelines or for event hrs.

### Possiblity

Unlike store orders, digital order trend is very much predictable (except during event hrs and few days in Nov and December).  
In other words, less impacted by seasonality and the data is stationary almost.

![Typical Order Trend](https://github.com/vijaycse/predictive-scaling)




### How

The idea is to look at the last 2 yrs data and run Machine Learning TimeSeries linear regression model 
to closely predict and auto scale hrly or every day based on predictions (numbers can be overridden for planned events) 
so that we can control the infra cost significantly. 


#### Design Overview:
  
![Design Overview](https://github.com/vijaycse/predictive-scaling)

#### Input Dataset Analysis:



#### Database:



#### Machine Learning:



#### Google Cloud Auto Scaler APIs



### Why not Auto-scaling?

We scale ahead of time before the events or spike seasons (holiday seasons Nov-Dec) generally. Auto scaling is reactive in nature
and often times, its based on CPU and memory usage. 
Predictive scaling works based on TPS/order trend.Predictive scaling and auto scaling can co-exist.
