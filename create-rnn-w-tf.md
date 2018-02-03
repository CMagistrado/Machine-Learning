# Create RNN w/ TF

Imports

```py
import numpy as np
import tensorflow as tf
import matplotlib.pyplot as plt
%matplotlib inline
```

Create a class that allows us to initialize the data and send batches back.

```py
class TimeSeriesData():
    
    def __init__(self, num_points, xmin, xmax):
        self.xmin = xmin
        self.xmax = xmax
        self.num_points = num_points
        self.resolution = (xmax - xmin) / num_points # how fine of a resolution are we creating
        self.x_data = np.linspace(xmin, xmax, num_points)
        self.y_true = np.sin(self.x_data)
    
    def ret_true(self, x_series):
        return np.sing(x_series)
    
    def next_batch(self, batch_size, steps, return_batch_ts=False):
        # Grab a random starting point for each batch of data
        rand_start = np.random.rand(batch_size, 1)

        # Convert to be on the same time series
        ts_start = rand_start * (self.xmax - self.xmin - (steps * self.resolution))

        # Create batch series on the x-axis
        batch_ts = ts_start + np.arange(0.0, steps+1) * self.resolution

        # Create the y data for the time series x-axis from previous step
        y_batch = np.sin(batch_ts)

        # FORMATTING for RNN.
        if (return_batch_ts):
            return y_batch[:,:-1].reshape(-1, steps, 1), y_batch[:,1:].reshape(-1, steps, 1) , batch_ts
        return y_batch[:,:-1].reshape(-1, steps, 1), y_batch[:,1:].reshape(-1, steps, 1)
```

So far, we have a timeseries class that takes in the number of points wanted, and the xmin and xmax. It then creates a bunch of attributes to store information. It creates the resolution , x_data, and y_true. y\_true takes in x\_data through the numpy sin function.

We will also create a convience method called **return true.** it takes in any series of x values and will return np.This will makes things easier.

Create a function to generate batches of this data

Let's go over the **next\_batch\(\).**

1. We created a random starting point. However we don't know if it's on the time-series data or not because the time series data was first defined when we initialzed this, with a xmin, xmax, and a number of points. 
2. We then convert this random start to BE on the tiem series. We do that by multiply the random start with the \(xmax - xmin - \(steps \* resolution\).
3. We need to create the X\_data for our time series batch. It's our starting point that we decided on + \(0, steps + 1\) and multiply it by the resolution. It's that resoluition times the arrangement, then t-start plus the following points.
4. We take the np.sin of our batch, to determine our true value.

Let's create some data so we can visualize what's happening.

```py
# TimeSeriesData(num_points, xmin, xmax)
ts_data = TimeSeriesData(250,0,10)
```

```
250 points between points 0 and 10. Now let's plot it.
```

```py
plt.plot(ts_data.x_data, ts_data.y_true)
```

![](/assets/Screen Shot 2018-02-02 at 10.16.10 PM.png)

Of **ts\_data**, we have our **x\_data** and our **y\_data**

#### Create Random 30-step batches

We can use this for predictions steps in the future.

```py
num_time_steps = 30
```

#### Create 1 batch of 30 steps

```py
y1,y2,ts = ts_data.next_batch(1,num_time_steps, True)
```

True says we want to True to say we want that time series data so we can plot it.  
But before we can do that, we have to **flatten** it, meaning we need to put it in a 1-D array.

```py
ts
```

```py
array([[ 5.25343895,  5.29343895,  5.33343895,  5.37343895,  5.41343895,
         5.45343895,  5.49343895,  5.53343895,  5.57343895,  5.61343895,
         5.65343895,  5.69343895,  5.73343895,  5.77343895,  5.81343895,
         5.85343895,  5.89343895,  5.93343895,  5.97343895,  6.01343895,
         6.05343895,  6.09343895,  6.13343895,  6.17343895,  6.21343895,
         6.25343895,  6.29343895,  6.33343895,  6.37343895,  6.41343895,
         6.45343895]])
```

Let's check this matrix

```py
ts.shape
```

```py
(1, 31)
```

We need to make it into a vector

```py
ts.flatten().shape # this is what matplotlib needs
```

```py
(31,)
```

Perfect. Now we must set the dimensions so they match up. As far as the training batches, 1 is shifted over. so we must start our training set data +1.

```py
y1,y2,ts = ts_data.next_batch(1,num_time_steps, True)
plt.plot(ts.flatten()[1:], y2.flatten(), '*')
```

![](/assets/Screen Shot 2018-02-02 at 10.27.06 PM.png)

This is our batch of data in our dataset. Let's see it relative to our entire data.

```py
plt.plot(ts_data.x_data, ts_data.y_true, label='sin(t)')
plt.plot(ts.flatten()[1:], y2.flatten(), '+')
plt.legend()
plt.tight_layout()
```

![](/assets/Screen Shot 2018-02-02 at 10.28.36 PM.png)

So we are trying to predict a time-series shifted by 1 step.  


### Create Training Data

```py
train_inst = np.linspace(5, 5 + ts_data.resolution * (num_time_steps + 1), num_time_steps + 1)
```

This training set is a bunch of values

```py
train_inst
```

```py
array([ 5.        ,  5.04133333,  5.08266667,  5.124     ,  5.16533333,
        5.20666667,  5.248     ,  5.28933333,  5.33066667,  5.372     ,
        5.41333333,  5.45466667,  5.496     ,  5.53733333,  5.57866667,
        5.62      ,  5.66133333,  5.70266667,  5.744     ,  5.78533333,
        5.82666667,  5.868     ,  5.90933333,  5.95066667,  5.992     ,
        6.03333333,  6.07466667,  6.116     ,  6.15733333,  6.19866667,
        6.24      ])
```

Let's plot where our training data SHOULD predict to.

```py
plt.title('A Training Instance')
plt.plot(train_inst[:-1], ts_data.ret_true(train_inst[:-1]), 'bo', markersize=15, alpha=0.5, label='Instance')

plt.plot(train_inst[1:], ts_data.ret_true(train_inst[1:]), 'ko', markersize=7, label='Target')
```

![](/assets/Screen Shot 2018-02-02 at 10.44.05 PM.png)

Given the blue points, can you predict the black points?
