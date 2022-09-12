# nuScenes-Trajectory-Prediction
[Deep Learning](https://sites.google.com/diag.uniroma1.it/fabriziosilvestri/home/teaching/deep-learning) project for Trajectory Prediction using nuScenes dataset. DIAG, Sapienza University in Roma

![image](https://user-images.githubusercontent.com/24941293/189555310-370e716e-14df-4660-b204-bd3fb54dd4eb.png)



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
