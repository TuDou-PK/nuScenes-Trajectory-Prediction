# nuScenes-Trajectory-Prediction
[Deep Learning](https://sites.google.com/diag.uniroma1.it/fabriziosilvestri/home/teaching/deep-learning) project for Trajectory Prediction using nuScenes dataset. DIAG, Sapienza University in Roma

![image](https://user-images.githubusercontent.com/24941293/189555310-370e716e-14df-4660-b204-bd3fb54dd4eb.png)

Co-author:
- [PK](https://github.com/TuDou-PK)
- [SCC](https://github.com/skant626)
- DJN

# How to Run

## Preparing Dataset

![image](https://user-images.githubusercontent.com/24941293/189555084-af18bdb3-cbd1-406c-ae95-97f6e443775b.png)
The dataset folder link is [https://drive.google.com/drive/folders/118Z18sWEg4CqHAhFcYmDqXvDj2qk4YtI?usp=sharing](https://drive.google.com/drive/folders/118Z18sWEg4CqHAhFcYmDqXvDj2qk4YtI?usp=sharing) , check "Add shortcut to Drive" and then you can import it.

## Runing Code
The main file is "DL_Project.ipynb", download and run it directly on your notebook or colab. Other files are supplementary to the main file.

# Backgroud Introduction

Trajectory prediction is the problem of predicting the short-term (1-3 seconds) and long-term (3-5 seconds) spatial coordinates of various road-agents such as cars, buses, pedestrians, rickshaws, and animals, etc. These road-agents have different dynamic behaviors that may correspond to aggressive or conservative driving styles.

For nuScenes dataset, that requesting 6 second predictions at 2 Hz, that means we have to predict 12 points as predicted trajectory.

# Data processing
nuScenes dataset is huge , we only use part of it, only using instance position x, y, velocity v, acceleration a, and head rating r.

![Show what data is used](https://user-images.githubusercontent.com/24941293/189556493-830a6aff-8566-457c-b53d-38e6a3a7dc02.png)
![Simplified figure for showing what data is used](https://user-images.githubusercontent.com/24941293/189556504-ce264f1c-e068-4149-942b-0ab88742d279.png)

The rough dataset structure be like: [frame_id, instance_token, x, y, z, v, a, r].

In this step, the order of the dataset is in chronological order of the scene, but we only need continuous chronological data for each instance as the continuous trajectory of the instance.

![Example data in chronological order of the scene](https://user-images.githubusercontent.com/24941293/189556612-42ccf7ec-ea70-4d68-949b-b515ffa7b9e0.png)


So the next step is getting unique instance ids and using those ids to reorganize the order of trajectory points of each instance.

The final dataset is arranged in the chronological order of the instances, that is, in the order of the trajectories of the instances.

![Example data in continuous chronological data for each instance](https://user-images.githubusercontent.com/24941293/189556626-a44736d7-9c5d-437c-adec-6360ca7629e3.png)

# Model
We have tried many different architecture of models include baselines and our modified architecture , then compared their performance respectively. And our final modified model has the best performance.

- First we tested a model that used basic LSTM model, which use incremental prediction, in the iteration, we predict the next point (x, y) of the trajectory in turn, and then use (x, y) as the training trajectory point to add to the sequence.
- As we know, this basic method have a big flaw when we have predict a long frames squence, because it's have a accumulated error(This is called naive forecasting). So we want to get the whole predicted position sequence at once as output throughout Linear layer.

## 1. LSTM + LinearX, LinearY
- [Code Link](https://colab.research.google.com/drive/1GFNBxcYfbNqtltdHGVLmRfCC-Q5a5ivK?usp=sharing)
- This is a plain basline which has accumlated error with iteration prediction. The following models are all modified architectures except this.



![image](https://user-images.githubusercontent.com/24941293/189557086-16d00fe5-35d2-460b-814e-3edf8f0be8fd.png)

How it predicts.

![image](https://user-images.githubusercontent.com/24941293/189557510-435cc05f-4193-4fa5-84bf-e4ae914ef7b3.png)


## 2. LSTM + Linear + LinearX, LinearY
- Add one more Linearr Layer to change the output to whole prediction.
![image](https://user-images.githubusercontent.com/24941293/189557107-f1de1bde-3805-43b9-98a9-c3c0e1ceaf2a.png)

After changing the predict way.

![image](https://user-images.githubusercontent.com/24941293/189557553-37f61a9f-d0c5-4e1b-b0fe-4ef87cd788dc.png)


## 3. LSTM + LinearXY
- Remove the last two Linear output layers to increase the effect of LSTM on loss.
![image](https://user-images.githubusercontent.com/24941293/189557143-b78e7966-43e3-4ef4-8885-bf1a89793a30.png)


## 4. Conv1D + Linear + LinearX, LinearY
- [Code Link](https://colab.research.google.com/drive/1WEYCfWhV2OyPEEdP4YKwZU3AhyvExuEL#scrollTo=7IuPfoPGZfVD)
- Add a CNN layer instead of LSTM to extract the feature of sequence. 

![image](https://user-images.githubusercontent.com/24941293/189557177-bec6a5b4-fff9-4867-bf5a-55638bf4de1f.png)


## 5. Conv1D + LinearXY
- Same as 5.3, increase the effect of CNN on loss.

![image](https://user-images.githubusercontent.com/24941293/189557195-66febb7e-7800-4077-ab89-34f0a92d2df6.png)

## 6. Conv1D + LSTM + Linear + LinearX, LinearY
- [Code Link](https://colab.research.google.com/drive/1C6lxeM4XG24USte5rYujX41FEUvchGuM#scrollTo=7IuPfoPGZfVD)
- Keep the LSTM to extract the time sequence information.
![image](https://user-images.githubusercontent.com/24941293/189557219-2a8ec1b3-0be4-41d1-9b32-8b790528710a.png)


## 7. Our final model: Conv1D + LSTM + LinearXY
- Same as 3, 5, and then get our final model.
- The architecture of model as follow.
![image](https://user-images.githubusercontent.com/24941293/189557265-50f71c9e-6e4e-4649-9c29-e87d49aa50c4.png)


# Training

- In order to crop the data into the same shape and expand the training set data，we divide the single trajectory into several samples using sliding window.
- We chose the five features contain target x, y, speed, acceleration and heading rate as input feed to the model, and get the predicted (x, y) as output to propagate loss.

![image](https://user-images.githubusercontent.com/24941293/189557634-3e0d8cd1-1606-46f2-9159-1f40781009e2.png)


# Testing

- avg_ade: 1.152837239360219
- avg_fde: 2.1436020780866762
- avg_missRate: 0.7627215551743853

## Trajectory Visulization

![image](https://user-images.githubusercontent.com/24941293/189557316-23e80206-6f6e-4609-a63a-218fec2cb9ad.png)

## Model Comparison

In the final section, we compared the metrics from 7 different models.

![image](https://user-images.githubusercontent.com/24941293/189557349-7e87d766-2aa1-41d9-a214-e1f05890dbea.png)


|Model|LSTM+LinearX,LinearY|LSTM+Linear+LinearX,LinearY|LSTM+LinearXY|Conv1d+Linear+LinearX,LinearY|Conc1d+LinearXY|Conv1d+LSTM+Linear+LinearX,LinearY|Conv1d+LSTM+LinearXY|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|Average ADE|4.03|1.35|1.35|1.32|1.32|1.16|**1.15**|
|Average FDE|7.34|2.59|2.59|2.42|2.74|2.11|**2.14**|
|Miss Rate|66.61%|74.96%|75.12%|74.10%|74.39%|76.19%|**76.27%**|

