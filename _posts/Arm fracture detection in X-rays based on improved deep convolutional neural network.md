---
title: Arm fracture detection in X-rays based on improved deep convolutional neural network
category: Review
tag: Detection, Fracture
---


>  ## ** Goal ** 

This paper's goal is to propose a novel deep learning method to detect arm fracture in X-rays.


>  ## ** Contribution ** 

**1. New backbone network based on feature pyramid architecture**

**2. Image preprocessing procedure**

**3. Receptive field adjustment with anchor scale reduction and tiny ROIs expansion

>  ## ** Method ** 

#### ** Backbone network**
  FPN + Fast R-CNN +RPN with <u?Gaussian non-local attention module (refine integrated features)</u>
    Integration of features (Novel method)



<a href="https://imgur.com/LxkaNVB"><img src="https://imgur.com/LxkaNVB.JPG"  title="source: imgur.com" /></a>




#### ** Preprocessing **
  Noise removal --> Morphological method
  Brightening --> Cumulative distribution function
  
<a href="https://imgur.com/MJbsSfi"><img src="https://imgur.com/MJbsSfi.JPG"  title="source: imgur.com" /></a>

  
  
#### ** Anchor scales reduction **
  {P2; P3; P4; P5; P6} : {512; 256; 128; 64; 32}  {256; 128; 64; 32; 16}
    Guarantees more foreground RoIs for RPN training because GT bounding boxes are too small

#### ** Expanding receptive field to fine tiny fracture **
  Adding pixels to width and height for small ROIs (length adjustment)
    Extract useful info from tiny ROIs

<a href="https://imgur.com/4WYn82P"><img src="https://imgur.com/4WYn82P.JPG"  title="source: imgur.com" /></a>

