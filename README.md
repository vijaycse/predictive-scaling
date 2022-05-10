# predictive-scaling


We are burning lot of cash just for GCP in our product space.  
We roughly run about ~500 instances (central and east) during normal traffic. 

Needless to say, the number of instances are super high during events such as (xbox launch go crazy with all BOTs traffic), 
gaming consoles etc. We scale up and down few times a month, sometimes weekly based on our perf tested scaling guidelines or for event hrs.

Unlike store orders, digital order trend is very much predictable (except during event hrs and few days in Nov and December).  
In other words, less impacted by seasonality and the data is stationary almost.

The idea is to look at the last 2 yrs data and run Machine Learning TimeSeries linear regression model 
to closely predict and auto scale hrly or every day based on predictions (numbers can be overridden for planned events) 
so that we can control the infra cost significantly. 

We scale ahead of time before the events or spike seasons (holiday seasons Nov-Dec) generally. Auto scaling is reactive in nature
and often times, its based on CPU and memory usage. 
Predictive scaling works based on TPS/order trend.Predictive scaling and auto scaling can co-exist.
