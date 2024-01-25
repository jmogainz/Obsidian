#### Motivation
Ever since the first generation of the Fitbit was released, I have been fascinated by fitness wearables and their future potential in improving not only the performance of athletes but also the quality of life for the general population. I find it so interesting that through the use of raw sensor data, algorithms can be developed to detect, classify, and predict various movement and health metrics. Some of the earliest examples of this were step counting, calorie burn estimation, distance tracking, and sleep monitoring. Most often, the accelerometer was used to do all of this calculation, and in the advanced models, GPS modules were also used. As of the most recent fitness wearables, features like heart rate monitoring and variability, elevation gain, VO2 Max Estimation, activity/sport detection, stress and recovery analysis,  ECG Monitoring, Advanced Sleep Analysis, Oxygen Saturation (SpO2) Measurement, Temperature. For me specifically, I have competed in 3 bodybuilding shows, and received benefits from fitness weables that I would never do without now that helped me stay on track with daily energy expenditure and muscle recovery. I have 3 years of experience working in ML/DL academic research and as fulltime employee for a government contractor. In light of this, I have always kind of known how these measurements were calculated, but I now get the oppurtunity to do an elementary version for myself. I have found a dataset that was built via a study (insert here description of UCI dataset Daily and Sports Activities). Due to hardware and software constraints, and the clear nature of this assignment, I believe I can demonstrate an understanding of the typical data science pipeline as well as displaying a decent analysis of trends and patterns with a downsized version of this dataset, so I will be using 10 of the 19 activities (chosen based on greater disparity than some of the other activities, again not much to with the core functionality of orange compared to complex deep learning frameworks). I want to see how well a strictly machine learning and simple NN approach can perform at predicting 1-10 possible activities given a single time instance of sensor data. I also will attempt to do feature extraction in order to create temporal samples.
#### Obstacles
My typical approach would be to use PyTorch deep learning framework in conjuction with Pandas to do the data preprocessing and model training. However, given the requirements layed out in this project, I may have to use pandas to do some up front processing to get the data in a place to feed it to orange. 

#### Dataset Info
'a17', 'a18', 'a01', 'a02', 'a12', 'a10', 'a05', 'a06', 'a03', 'a16'
Dataset Activities
- Rowing
- Jumping
- Sitting
- Standing
- Running
- Walking
- Going up stairs
- Going down stairs
- Lying down
- Indoor Cycling

Original file structure is as follows:

File structure: 
19 activities (a) (in the order given above) 
8 subjects (p) 
60 segments (s) 
5 units on torso (T), right arm (RA), left arm (LA), right leg (RL), left leg (LL) 
9 sensors on each unit (x,y,z accelerometers, x,y,z gyroscopes, x,y,z magnetometers) 

Folders a01, a02, ..., a19 contain data recorded from the 19 activities. For each activity, the subfolders p1, p2, ..., p8 contain data from each of the 8 subjects. In each subfolder, there are 60 text files s01, s02, ..., s60, one for each segment. In each text file, there are 5 units x 9 sensors = 45 columns and 5 sec x 25 Hz = 125 rows. Each column contains the 125 samples of data acquired from one of the sensors of one of the units over a period of 5 sec. Each row contains data acquired from all of the 45 sensor axes at a particular sampling instant separated by commas. 

Columns 1-45 correspond to: T_xacc, T_yacc, T_zacc, T_xgyro, ..., T_ymag, T_zmag, RA_xacc, RA_yacc, RA_zacc, RA_xgyro, ..., RA_ymag, RA_zmag, LA_xacc, LA_yacc, LA_zacc, LA_xgyro, ..., LA_ymag, LA_zmag, RL_xacc, RL_yacc, RL_zacc, RL_xgyro, ..., RL_ymag, RL_zmag, LL_xacc, LL_yacc, LL_zacc, LL_xgyro, ..., LL_ymag, LL_zmag. 

Therefore, columns 1-9 correspond to the sensors in unit 1 (T), columns 10-18 correspond to the sensors in unit 2 (RA), columns 19-27 correspond to the sensors in unit 3 (LA), columns 28-36 correspond to the sensors in unit 4 (RL), columns 37-45 correspond to the sensors in unit 5 (LL).

Given this information...I will plan to simplify this structure into something orange can take in much easier. 

In good faith of comparing my models to smart wearables through time, I will only be using features from the sensor data for the left arm to resemble my motivation as closely as possible. I will begin with just accelerometer data as that was what the first generation of the fitbit has in 2009, I will move to accelerometer and gyro next as the Pebble Time in 2015 and the early android wear devices had them. FInallly I will test all three sensors as the the most modern smart watches contain ecc, gyro, and magnetometers, the Apple Watch for example. 

Doing temporal approach and non temporal approach.

Non-Temporal approach:
- Python
	- Combine all raw sensor data for 5 activities and all subjects into a dataset with each sample labeled by activity
- Orange
	- Bring in single csv input file
	- Edit labels to map the a# to human readable activity for easier analysis
	- Shrink so that I can perform different predicts fast (in a python world I would just parallel the studies instead of having to wait to make a change and keep iterated synchronously)
	- Select columns to use in prediction
	- Remove Outliers
	- Normalize data
	- Randomize
	- Visualize
	- Analyze data as above similar
	- Perform standard 70-30 train-test split as to ensure no overfitting is occurring
	- Onehot encode the label column before going into learner (overriding learner's default preprocessor)
	- Feed into traditional mlp, gb, or kn

Temporal approach:
- Python
	- Approach is done as a work around to not having access to powerful recurrent neural networks through orange such as basic LSTM
	- Combine data for select activities and all patients (preserving original 5 minute order for each patient) ordered by activity and then patient within each activity
	- Reduce dimensionality
	- Calculate statistical features per 25 timestep (1 second) window
	- Label the resultant feature vector as a single activity sample
	- Will reduce amount of samples per activity but hopefully will give us some temporal insight
- Orange process is same as above

#### Approach and Considerations
Preprocessor was used to normalize data before visualization and analysis
So that the normalization was not done twice, the one-hot encoding of the labels was done as an attachment to the learners (overriding the learners default preprocessors)...I am aware that the documentation recommends you to do all preprocessing as an attachment to the learner but as long as you override the learners default preprocessor it will prevent any sort double preprocessing from being implemented. 

I chose to remove outliers due to the fact that although this is sensor data from various activities and unique movements in activities may be important to understanding the activity, from the activities that I have picked out, they are primarily constant repetitive movement activities, so any outliers are probably more noise than anything and thus should be disregarded as a valid data point. With that being said I still kept the outlier detection contamination percentage very low...down at 4%. Which basically says that it is expected that 4% of the data is noise. 

#### Results
The images here will be used in report document

**Peak into the Head of the Data Table**
![[Pasted image 20240125012529.png]]

Data at 600k samples, too large to perform iterative analysis on, so I downsized 25% to 150k, removed 4% outliers, taking it 145k, and then did a 70-30 train test split, could do less down to like 50-50 with the large amount of data since their is no need for that much data to train and want to make sure overfitting is not occurring
Non-temporal static raw sensor data samples

**Gyro xyz, mag xyz, acc xyz**
t-SNE Plot
![[Pasted image 20240125005821.png]]

PCA Scatter Plot
![[Pasted image 20240125005935.png]]

Classification Prediction Metrics
![[Pasted image 20240125010045.png]]

Classification Prediction Confusion Matrix GB
![[Pasted image 20240125010409.png]]


**acc xyz, gyro xyz**

t-SNE Plot
![[Pasted image 20240125012601.png]]

PCA Scatter Plot
![[Pasted image 20240125012650.png]]

Classification Prediction Metrics
![[Pasted image 20240125012816.png]]

Classification Prediction Confusion Matrix
![[Screenshot from 2024-01-25 01-28-56.png]]

**acc xyz**

t-SNE Plot
![[Pasted image 20240125011518.png]]

PCA Scatter Plot
![[Pasted image 20240125011605.png]]

Classification Prediction Metrics
![[Pasted image 20240125011722.png]]

Classification Prediction Confusion Matrix GB
![[Pasted image 20240125011823.png]]

Full Data Amount, 600k, Static samples, 50-50 train test, now that there is plenty of data, can decrease train set and increase test to ensure we are not getting any overfitting
Want to see how good we can possibly train a model with this full data amount. 

t-SNE Plot
![[Screenshot from 2024-01-25 02-05-51.png]]

PCA Scatter Plot
![[Screenshot from 2024-01-25 02-07-07.png]]

Classification Prediction Metrics
![[Screenshot from 2024-01-25 02-07-55.png]]

Classification Prediction Confusion Matrix GB
![[Screenshot from 2024-01-25 02-08-41.png]]


As an extra effort I really wanted to see if using the non-temporal models provided by orange, I was capable of doing enough feature engineering on only pure acc data available in the lower-end/older fitness tracking watches that the model could actually learn from less overall samples and achieve a higher performance than the latest watches that have all three sensors throwing hardware at the problem. So single samples that represent the statistical properties and window based properties of 1 second worth of data which equates to 25 samples recorded at 25 HZ. 

This dataset ends up being 27000 samples, derived from the full 600k, and is 50-50 split as well

Data Peak
![[Screenshot from 2024-01-25 03-09-09.png]]

**Acc xyz**

t-SNE Plot
![[Pasted image 20240125043617.png]]

PCA Scatter Plot
![[Pasted image 20240125043641.png]]

Classification Pred Metrics
![[Pasted image 20240125043715.png]]

Classification Pred Confusion Matrix
![[Pasted image 20240125043741.png]]

So in conclusion, time series modeling is the go-to for sensor data if you can get enough data. It goes without saying the absolute highest achieving model would be one with more sensors and a time-component baked in or recurrent nn. 

t-SNE
![[Screenshot from 2024-01-25 04-11-04.png]]

PCA Scatter PLot
![[Screenshot from 2024-01-25 04-11-36.png]]

Preds
![[Screenshot from 2024-01-25 04-12-11.png]]

Conf Matrix
![[Screenshot from 2024-01-25 04-13-34.png]]