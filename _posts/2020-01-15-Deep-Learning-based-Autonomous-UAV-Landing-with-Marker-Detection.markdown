---
layout: post
title: Deep Learning-based Autonomous UAV Landing with Marker Detection
date: 2020-01-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
image: img_drone.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Deep learning, Computer vision]
---

As a Lead Scientist I work for our Innovation Lab. One of the projects I have been working on is the development of an AI based solution to allow the autonomous landing of a drone using a monocular camera as the main sensor . I successfully completed this 5 months project and tested the results on a drone simulator built on Unreal Engine (UE). We are currently trying to implement the algorithm I developped in a small drone equipped with a Rasberry pi 3. There are other computational and hardware challenges that I will explain along the way. 

# Objectif of the Project
Autonomous landing of an Unmanned Aerial Vehicle (UAV) is a difficult problem. Although the GPS signal is essential to assist the drone in landing, its error in meters remains significant. In addition, the received signal is often very weak when the drone is in environments with poor GPS connection (urban area, mountains, etc.). We want to remedy this problem by using the standard means available, ie, a monocular vision camera, coupled with the latest advances in Deep Learning for object detection and localization. The object in question in our case is a ground marker. An algorithm based on convolutional neural networks will be proposed in order to help the drone fly autonomously towards the target and land there safely with better precision.


# Context
The autonommous precision landing is part of the AI ​​autopilot project which aims to equip drones with the latest advances in artificial intelligence in order to make the drone autonomous and more stable in the face of environmental variations. The project consists of three main axes: 
* Design of a robust autopilot using reinforcement learning approaches. 
* Develop an AI algorithm to improve the performance of the estimator 
* Develop an algorithm allowing the drone to perform an autonomous landing with the help of a ground marker. 

# Challenges
For the practical implementation development of the precision landing part, we note the following scientific and technical obstacles:
* The reflection of the sun or the shadow on the tag can make detection difficult
* Difficulty detecting the tag if the image from the camera is distorted.
* The lower the resolution of the camera, the more difficult it is to recognize the tag, especially at high altitude
* The techniques found in the literature are generally based on the detection of the geometric characteristics of the tag, and this is where the detection problems that have just been mentioned mainly come from. The proposed solution that relies on the object detection algorithm, named YOLO can overcome most these challenges 

# Introduction
My main interest was in the development of the last, but not least point. In order to carry out the necessary experiments, it was important to choose a drone simulator where the dynamics of the UAV (quadcopter in my case) is well defined. I went through a literature survey to identify numerous autonomous vehicle simulator with the idea to find the one that is most suitable for our type of application. In the survey, the simulator [AirSim][AirSim] (Aerial Informatics and robotics Simulation) was selected. 
>[AirSim][AirSim] is a simulator for drones, cars and more, built on Unreal Engine (we now also have an experimental Unity release). It is open-source, cross platform, and supports hardware-in-loop with popular flight controllers such as PX4 for physically and visually realistic simulations. It is developed as an Unreal plugin that can simply be dropped into any Unreal environment. Similarly, we have an experimental release for a Unity plugin. 

The drone will thus have the ability to interact with a virtual environment that is closest to reality. This will allow a smoother transition when the time comes to try the algorithms developed on a real drone.  


## Main tasks
An algorithm which allows the tracking of a landmark in real time, as well as the calculation of its coordinates using only a single monocular vision camera is proposed. Tiny Yolo has been chosen for the object detection part for its small architecture, which allows it to run faster. Learning the network requires time and significant computing power. It is performed on an Ubuntu 16.04 machine equipped with a GTX 1080 graphics card. The Cuda and cudNN libraries are also installed to allow parallel calculation and thus increase performance and have faster training. ComputerVision mode makes it easier to control the drone, a small Python script is used to acquire the set of images necessary for training the network. To assess the performance of the proposed approach, a quadcopter drone equipped with a camera is simulated with [AirSim][AirSim] on the "Landscape Mountain" environment. The results obtained in simulation make it possible to confirm that the drone can detect the marker in real time and, based on the algorithm that will be described later, to calculate the coordinates of the landmark, and therefore guide the drone up to a distance of less than a meter in height and with an error of 10 cm from the center of the tag (landmark). The drone, when reaching this position, triggers a secure automatic landing using a feature that is available in the flight controller (autopilot).   


![airsim]({{site.baseurl}}/images/airsim_01.jpg){:height="50%" width="50%"}

*[AirSim][AirSim] is used for image acquisition. The Figure shows 4 images taken from a drone's down facing camera at different altitudes)*
# Training of Tiny Yolo

Yolo (You Only Look Once) is a fast object detection algorithm developed by Redmon et al. Tiny Yolo is just a compact version of it. Object detection is a task in computer vision which consists of identifying the presence, location and type of one or more objects in a given image. In our case, we are talking about a single object and a single class which represents our ground marker. YOLO makes it possible to predict the coordinates of the detected objects as well as the probability of their associated classes. On a computer equipped with an NVIDIA GTX1080 GPU, it is possible for a YOLOv3, to run at more than 30 fps. YOLO processes the detection of the type of object, and its location inside the image simultaneously. Thanks to its good generalizable representations, it is possible to do transfer learning, that is to say, use weights that are pre-trained and build a new network by changing only the last connected layer.

![training]({{site.baseurl}}/images/chart1.png){:height="50%" width="50%"}

*Evolution of the mean average precision (`mAP`) together with the average loss during the training, for 2000 iteration*


{% highlight markdown %}
Running the full YOLOv3 model on a Rasberry Pi is a challenging task, provided tha a substantial processing power is needed. Tiny YOLO makes it possible to embark YOLOv3, with a small decrease in precision, however,  with a considerable gain in computing time. The structure of Tiny Yolo network in version 3 is presented in the Table below
{% endhighlight %}

<!-- The above chart displays the evolution of the mean average precision (`mAP`) together with the average loss during the training.  -->

![Yolo Architecture]({{site.baseurl}}/images/yolo_architecture.png){:height="50%" width="50%"}

*You Only Look Once Tiny (YOLO tiny) Architecture*
# Landing System 
The diagram below illustrates the operating procedure for the autonomous landing system. The Tiny YOLO localisation algorithm processes the video stream from the on-board camera image by image, so for each image processed, the network generates a set of coordinates composed of the detected tag center (x, y), with its dimension given in width and height (w, h), as well as the probability "s" of belonging to a class. the full vector is given as follows: [x, y, w, h, s]. These coordinates are then converted to offset angles with respect to the center of the ground marker. These angles are sent to the autopilot so that the drone moves in a way  the marker is always in the center of the image. It is assumed that the angles of the MAV (yaw, pitch, roll) can be neglected when the offset angles are calculated. 

![diagram]({{site.baseurl}}/images/diagram.png){:height="30%" width="30%"}
![ubuntu simulated drone]({{site.baseurl}}/images/sim_drone.jpg){:height="50%" width="50%"}  
*Autonomous landing procedure &nbsp;&nbsp;&nbsp;   &nbsp;   &nbsp;   &nbsp;  &nbsp;  &nbsp;  &nbsp;  &nbsp;   Autonomous landing simulation with Unreal Engine Editor*

On the screenshot above, one can observe the Unreal Editor (UE) on the left-hand side running the simulation inside the Landscape Mountain environnement, and showing the drone flying over the marker. On the right-hand side, an extra window is generated to clearly observe the bounding blue box around the detected marker. The Linux terminal is used to launch the UE and of course to run the python code for the autonomous landing procedure.

[AirSim]: https://github.com/microsoft/AirSim



# Technologies and Frameworks 
A good part of the project involved training Tiny YOLO with respect to the ground marker. For this reason a set of 400 images including the ground marker were acquired from the simulator. I did not explain how I did the annotation of the The images are used to train the deep neural network Tiny Yolo. The main tools and framework used during the development are as follows 

* `Python` as the main programming langage
* `Keras` with `Tensorflow` as a backend
* `OpenCV`
* `AirSim API's`

<!-- ![training]({{site.baseurl}}/images/chart1.png){:height="50%" width="50%"} -->