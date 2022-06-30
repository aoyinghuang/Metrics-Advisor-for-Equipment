# Preprocessing your data

Multivariate industrial time-series data can be challenging to deal with. This article will show you how to preprocess your data to Metrics Advisor for equipment.

## 1. Data format requirements 

Data format should be store all the sensor data in one file/table.

- One table/ file per asset.
- All sensors from that asset are represented in that one file/table.

## 2. Data schema requirements 

Metrics Advisor for equipment uses your data in the following steps:

- to train a model 
- to evaluate a model
- to conduct a near real time inference

Metrics Advisor for equipment accepts tables with only three columns:

- The first column headers of the file are a timestamp that indicates the date and time. 
- The second column headers contain the sensor name. 
- The third column headers show the data value from the sensor.

| Timestamp     | Sensor  name | Sensor  value |
| ------------- | ------------ | ------------- |
| 1/1/2020 0:00 | Sensor 1     | 2.0456        |
| 1/1/2020 0:00 | Sensor  2    | 6.4948        |
| 1/1/2020 0:05 | Sensor  1    | 4.2938        |
| 1/1/2020 0:05 | Sensor  2    | 4.3894        |
| 1/1/2020 0:10 | Sensor  1    | 5.4098        |

**_NOTE:_** Timestamp format : yyyy-MM-ddTHH:mm.

The easiest way to prevent problems with column headers is to take the following precautions:
o	Make sure your column headers don't include any invalid characters, such as spaces.
o	Valid characters are: 200 characters.
o	Valid characters are: 0-9, a-z, A-Z, and # $ . \ - (hyphen) _ (underscore)
o	Make sure that the timestamp column is the one furthest to the left in your CSV file.
o	Make sure that you don't have any duplicated column headers.

## 3. Data quality

- As the model learns normal patterns from historical data, the training data should represent the **overall normal** state of the system. It's hard for the model to learn these types of patterns if the training data is full of anomalies. An empirical threshold of abnormal rate is **1%** and below for good accuracy.

- In general, the **missing value ratio of training data should be under 20%**. Too much missing data may end up with automatically filled values (usually linear values or constant values) being learned as normal patterns. That may result in real (not missing) data points being detected as anomalies.

  

### Data quantity

- **Minimum number of data points to train a model:** The empirical rule is that you need to provide **15,000 or more data points (timestamps) per variable** to train the model for good accuracy. In general, the more the training data, better the accuracy. However, in cases when you're not able to accrue that much data, we still encourage you to experiment with less data and see if the compromised accuracy is still acceptable.
- **Minimum number of data points to batch inference: **At most 20000, at least 12 sliding window length.
- **Minimum number of data points to near real time inference:** At most 2880, at least 1 sliding window length. Detecting timestamps: From 1 to 10.

### Timestamp round-up

In a group of variables (time series), each variable may be collected from an independent source. The timestamps of different variables may be inconsistent with each other and with the known frequencies. Here's a simple example.

*Variable-1*

| timestamp | value |
| :-------- | :---- |
| 12:00:00  | 1.0   |
| 12:01:00  | 1.5   |
| 12:02:00  | 0.9   |
| 12:03:00  | 2.2   |
| 12:04:00  | 1.3   |

*Variable-2*

| timestamp | value |
| :-------- | :---- |
| 12:01:00  | 2.2   |
| 12:05:00  | 2.6   |
| 12:07:00  | 1.4   |
| 12:08:00  | 1.7   |
| 12:09:00  | 2.0   |

We have two variables collected from two sensors which send one data point every 1 min. However, the sensors aren't sending data points at a strict frequency, but sometimes earlier and sometimes later.In the above example, timestamps of variable 1 and variable 2 must be properly 'rounded' to their frequency before alignment.

Let's see what happens if they're not pre-processed. The merged table will be

| timestamp | Variable-1 | Variable-2 |
| :-------- | :--------- | :--------- |
| 12:00:00  | 1.0        | `nan`      |
| 12:01:00  | 1.5        | 2.2        |
| 12:02:00  | 0.9        | `nan`      |
| 12:03:00  | 2.2        | `nan`      |
| 12:04:00  | 1.3        | `nan`      |
| 12:05:00  | 0.9        | 2.6        |
| 12:06:00  | `nan`      | `nan`      |
| 12:07:00  | `nan`      | 1.4        |
| 12:08:00  | `nan`      | 1.7        |
| 12:09:00  | `nan`      | 2.0        |

`nan` indicates missing values. Obviously, the merged table isn't what you might have expected. 

Therefore, the timestamps of variable 1 and variable 2 should be pre-processed (rounded to the nearest 1 min timestamps) and the new time series are

*Variable-1*

| timestamp | value |
| :-------- | :---- |
| 12:00:00  | 1.0   |
| 12:01:00  | 1.5   |
| 12:02:00  | 0.9   |
| 12:03:00  | 2.2   |
| 12:04:00  | 1.3   |

*Variable-2*

| timestamp | value |
| :-------- | :---- |
| 12:00:00  | 2.2   |
| 12:01:00  | 2.6   |
| 12:02:00  | 1.4   |
| 12:03:00  | 1.7   |
| 12:04:00  | 2.0   |

Now the merged table is more reasonable.

| timestamp | Variable-1 | Variable-2 |
| :-------- | :--------- | :--------- |
| 12:00:00  | 1.0        | 2.2        |
| 12:00:30  | 1.5        | 2.6        |
| 12:01:00  | 0.9        | 1.4        |
| 12:01:30  | 2.2        | 1.7        |
| 12:02:00  | 1.3        | 2.0        |

Values of different variables at close timestamps are well aligned, and the model can now extract correlation information.




