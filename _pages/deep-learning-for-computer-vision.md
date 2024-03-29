---
title: "Deep Learning for Computer Vision"
permalink: /notes/deep-learning-for-computer-vision/
author_profile: true
---

## Image Classification

* Assign a label to an image
* ImageNet Large Scale Visual Recognition Challenge
    - 1000 categories

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
* [Microsoft COCO](https://arxiv.org/abs/1405.0312)
    - Complex everyday scenes containing common objects in their natural context
    - 80 object categories
    - 1.5 million object instances in 330k images
* [PASCAL Visual Object Classes Challenge](http://host.robots.ox.ac.uk/pascal/VOC/pubs/everingham15.pdf)
    - 20 object categories

### [Faster R-CNN](https://arxiv.org/abs/1506.01497)

* A CNN is pretrained for image classifications (e.g. ImageNet) and one of its intermediate layers is used as a feature extractor
* A region proposal network (RPN) finds up to a predefined number of regions
* Initially all combinations of given sizes (e.g. 64, 128, 256 pixels) and given ratios between width and height (e.g. 0.5, 1.0, 1.5) are used as reference regions
* RPN learns to predict a probability that a reference region contains an object, and x, y, width, and height offset from the reference
* The extracted features are reused for classifying the regions using a region-based CNN (R-CNN)
* R-CNN consists of two fully-connected layers of size 4096, followed by two output layers—one predicting the class (including "background") and one predicting the region offset

### [YOLO](https://arxiv.org/abs/1506.02640)

* A single convolutional network extracts features and predicts the bounding boxes
* The input image is conceptually divided into a grid of cells (for example, 7 × 7)
* For each cell, the model predicts multiple bounding boxes (for example, two), whose center falls into the cell, and one set of class probabilities
* The model predicts 5 values for each bounding box:
    - x and y coordinates (between 0 and 1, relative to the cell)
    - width and height (between 0 and 1, relative to the image)
    - confidence score (between 0 and 1)
* At test time, the confidences are multiplied by the class probabilities to get a set of class scores for each bounding box
* For every bounding box, keeps only the class with the highest score
* Overlapping bounding boxes are filtered using non-maximum suppression
* At training time, the bounding box with the highest Intersection over Union with a ground truth bounding box is "responsible for predicting the object", i.e. takes part in the loss calculation
* Target for the confidence score is the Intersection over Union between the predicted and ground truth bounding boxes
* The loss of a bounding box is the sum of squared errors from its coordinates and dimensions, confidence score, and class probabilities

<figure>
  <img src="https://raw.githubusercontent.com/D3lt4lph4/papers/master/docs/images/imagedetection/yolo/network.png">
  <figcaption>
    YOLO architecture.
    <a href="https://dx.doi.org/10.1109/CVPR.2016.91">Redmon et al. 2016</a>
  </figcaption>
</figure>

### Non-Maximum Suppression

* Bounding boxes that predict the same class, are sorted by probability, and iterated starting from the one with the highest probability
* At each step, calculates Intersection over Union (IoU) with the next bounding box
* A high IoU means that there is considerable overlap between the bounding boxes
* If the IoU is higher than a threshold, any remaining bounding boxes are discarded

### [YOLO9000](https://arxiv.org/abs/1612.08242) and [YOLOv3](https://pjreddie.com/media/files/papers/YOLOv3.pdf)

* There are multiple detection layers at different levels of the model (for example, 3)
* Each grid cell at each detection layer predicts multiple bounding boxes (for example, 3) and a separate set of class probabilities for every box (as opposed to YOLO)
* Boxes are predicted relative to predefined anchor boxes, like in Faster R-CNN
* The anchor boxes are centered at the cell center, i.e. the bounding box location is still predicted relative to the grid cell and the anchor box only defines a prior width and height
* The three boxes predicted at a cell each have a separate prior width and height
* The three detection layers use separate priors, totaling nine anchor box shapes
* The anchor box dimensions are determined by running k-means clustering on the training set bounding boxes
* At each detection layer, the bounding box that is responsible for a training target is the one whose anchor box best matches the object
    - The grid cell is determined by the object center
    - The predictor within the cell is the one whose prior dimensions give the ighest Intersection over Union with the target
* Confidence score is now called objectness, and YOLOv3 uses binary targets
    - The target is one for the bounding boxes that are responsible for predicting any objects
    - Bounding boxes that are not responsible for predicting any objects, but whose anchor box overlaps an object (high enough IoU), are ignored from the objectness loss
    - Objectness loss is calculated as the binary cross entropy from those boxes that are not ignored


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
