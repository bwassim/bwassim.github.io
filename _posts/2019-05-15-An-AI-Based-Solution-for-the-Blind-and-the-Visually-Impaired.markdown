---
layout: post
title: An AI-Based Solution For the Blind and for the Visually Impaired
date: 2019-05-15  +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
image: lighthouse.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Deep learning, Computer vision, YOLO]
---

`LightHouse` is an application designed to offer a unique experience to the visually impaired or blind person and contribute to their well-being. It leverages the latest advances in Artificial Intelligence (AI), especially in one of its subfields, namely, deep learning. The application would allow anyone in possession of a smartphone and without the use of one’s eyesight to know approximately what's in front of them, what object to look for and provide instructions on how to find it. The innovation of the algorithm stands not only on the latest object detection and localization algorithm, i.e., YOLO, but more importantly, on the many features (modes) that would assist the visually impaired people in their daily life and inside their home. 

# Motivation
It is observed that more than 1.7 million people in France experience some trouble in their vision. 207000 are highly visually impaired, amongst them 65000 are blind. 90% of people older than 65 are visually impaired. These numbers are expeted to double in 2050. An application for detecting, localizing and finally guiding the end user towards an object . The application will eventually run on a smartphone and the way it works will be explained. 
The first thing that comes to mind when designing this kind of application is to use a camera as a replacement to our human vision. One can think of the ouput of a camera (video) as a succesion of images that can be utilized to detect, in real time, different objects in the image . One of the object detection algorithm that offers a good tradeoffs in terms of inference time and precision is called `YOLO`, which stands for You Only Look Once. The figure below is extracted from a paper by Redmon and Farhadi on their reference paper of YOLOv3. The benchmark figure depicts a significant performance gain given in terms of inference time, when compared to the other object detection methods.

![yolo-benchmark]({{site.baseurl}}/images/benchmark-yolo.png){:height="50%" width="50%"}

*Figure highlighting the performance of YOLOv3 in function of inference time(ms) and mean average precision (mAP)*
# Choice of the object detection neural architecture 
You Only look Once (YOLO) is one of the fastest object detection algorithms. It represents a good tradeoff in terms of real-time detection and accuracy. It uses only convolutional layers, making it a fully convolutional network (FCN). YOLO v3 is based on a deeper architecture of feature extractor called Darknet-53. The number outlines the 53 convolutional layers, each followed by a batch normalization layer and Leaky ReLU activation. In order to prevent a loss of low-level features, often present with pooling, a stride 2 is used to downsample the feature maps. Detection algorithm does not only predict class labels but detects objects's location as well. It therefore not only classifies the image into a category, but it also detects multiple objects within an image.

![darknet-53]({{site.baseurl}}/images/darknet-53.png){:height="40%" width="40%"}

*Darknet-53 architecture*

### Interpretation of the output
I am not going to go through the details of how `YOLO` works, this can be found in the litterature, I will however focus on a key step to understand the rest of the project. To make it simple, one can think of `YOLO` as a bloack box, where :
* the input is a batch of images of shape `(m, 416, 416, 3)`, where `m` is the number of images and 
* the output is a list of bounding boxes along with the recognized classes. Each bounding box of the detected object is represented by `[c, x, y, w, h, s]`. The detected object class `c`, its tag center `(x, y)`, the width and height of the box given as `(w, h)`, as well as the probability `s` of belonging to a class. 
* YOLOv3 has been trained to recognise 80 category of objects.

![livingroom]({{site.baseurl}}/images/livingroom.jpg){:height="60%" width="60%"}