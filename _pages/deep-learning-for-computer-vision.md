---
title: "Deep Learning for Computer Vision"
permalink: /notes/deep-learning-for-computer-vision/
author_profile: true
math: true
---

## Image Classification

* Assign a label to an image
* ImageNet Large Scale Visual Recognition Challenge: 1000 different classes

### [AlexNet](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks)

* Five convolutional layers
* Convolution kernels are 3-dimensional, depth matching the input depth
* Convolution is calculated in two directions (x, y), producing a 2-dimensional output for each kernel
* Kernel sizes: 11x11, 5x5, and three layers of 3x3
* Two position-wise fully connected layers (i.e. 1x1 convolutions) with 4096 outputs and one with 1000 outputs before softmax
* Max pooling to reduce resolution after first, second, and the last convolutional layer
* Data augmentation by mirroring and cropping
* Dropout

<figure>
  <img src="https://www.mdpi.com/applsci/applsci-07-00992/article_deploy/html/images/applsci-07-00992-g001.png">
  <figcaption>
    AlexNet architecture.
    <a href="https://dx.doi.org/10.3390/app7100992">Llamas et al. 2017</a>
  </figcaption>
</figure>

### [VGGNet](https://arxiv.org/abs/1409.1556)

* More layers, e.g. 13 convolutional and 3 fully connected layers
* Smaller kernels, all are sized 3x3

<figure>
  <img src="https://www.researchgate.net/profile/Debaditya_Acharya/publication/323796590/figure/fig1/AS:606593536258049@1521634577318/The-architecture-of-a-VGGNet-CNN-after-Wang-et-al-2017_W640.jpg">
  <figcaption>
    VGGNet architecture.
    <a href="https://doi.org/10.1145/3038912.3052638">Wang et al. 2017</a>
  </figcaption>
</figure>

### [ResNet](https://arxiv.org/abs/1512.03385)

* Even deeper, typically 50 or 101 layers
* Consists of residual blocks that contain two convolutional layers and a bypass connection
* A residual block can achieve identity mapping by zeroing out the layer weights

### [Inception](https://arxiv.org/abs/1409.4842)

* Consists of nine inception modules
* Each module performs 1x1, 3x3, and 5x5 convolutions and concatenates them in depth dimension
* 1x1 convolutions are also performed to reduce the depth before the 3x3 and 5x5 convolutions

<figure>
  <img src="https://www.researchgate.net/profile/Ahmed_Abdelbaki5/publication/335443279/figure/fig12/AS:796779360559116@1566978411258/Inception-module-with-dimension-reductions-79_W640.jpg">
  <figcaption>
    Inception module.
    <a href="https://arxiv.org/abs/1409.4842">Szegedy et al. 2014</a>
  </figcaption>
</figure>


## Object Detection

* Finding and classifying a variable number of objects in an image
* Variable number of outputs: a list of bounding boxes with associated labels (and probabilities for the labels and bounding boxes)

### [Faster R-CNN](https://arxiv.org/abs/1506.01497)

* A CNN is pretrained for image classifications (e.g. ImageNet) and one of its intermediate layers is used as a feature extractor
* A region proposal network (RPN) finds up to a predefined number of regions
* Initially all combinations of given sizes (e.g. 64, 128, 256 pixels) and given ratios between width and height (e.g. 0.5, 1.0, 1.5) are used as reference regions
* RPN learns to predict a probability that a reference region contains an object, and x, y, width, and height offset from the reference
* The extracted features are reused for classifying the regions using a region-based CNN (R-CNN)
* R-CNN consists of two fully-connected layers of size 4096, followed by two output layers—one predicting the class (including "background") and one predicting the region offset


## Image Segmentation

* Predict a segmentation map given an image
* The output is the same size as the input

### [U-Net](https://arxiv.org/abs/1505.04597)

* After reducing resolutiong using the usual convolutional layers, increase the resolution by upsampling
* Cross connections (not part of the original U-Net architecture) from the downsampling part of the network to the same-sized image in the upsampling part

<figure>
  <img src="https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/u-net-architecture.png">
  <figcaption>
    U-Net architecture.
    <a href="https://arxiv.org/abs/1505.04597">Ronneberger et al. 2015</a>
  </figcaption>
</figure>
