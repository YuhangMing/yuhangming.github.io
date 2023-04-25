---
layout: post
title:  Feature Detector and Descriptor
date:   2023-03-15 13:40:16
description: A review on common image features.
tags: image_processing
categories: research-posts
---

# Table of Content
- [Table of Content](#table-of-content)
- [Harris](#harris)
- [Shi-Tomasi](#shi-tomasi)
- [SIFT](#sift)
- [SURF](#surf)
- [FAST](#fast)
- [BRIEF](#brief)
- [BRISK](#brisk)
- [ORB](#orb)
- [FREAK](#freak)


# Harris 
Gradient-based corner detector.
[bmvc88](http://www.bmva.org/bmvc/1988/avc-88-023.pdf)


# Shi-Tomasi
Gradient-based corner detector.
[cvpr94](https://ieeexplore.ieee.org/document/323794)
[paper](https://users.cs.duke.edu/~tomasi/papers/shi/TR_93-1399_Cornell.pdf)


# SIFT 
(scale invariant feature transform)
Gradient-based corner detector and descriptor.
[iccv99](https://www.cs.ubc.ca/~lowe/papers/iccv99.pdf)
[ijcv04](https://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf)
**Main steps**:
1. Scale-space extrema detection: 
   The image is convolved with Gaussian filters at different scales, and then the difference of successive Gaussian-blurred images are taken. Keypoints are then taken as maxima/minima of the Difference of Gaussians (DoG) that occur at multiple scales.
2. Keypoint localization:
   2.1 Interpolation of nearby data for accurate position
   2.2 Discarding low-contrast keypoints
   2.3 Eliminating edge responses
3. Orientation assignment
   Each keypoint is assigned one or more orientations based on local image gradient directions, and the gradient magnitude and orientation are precomputed using pixel differences.
4. 128-d Keypoint descriptor
   Create a set of orientation histograms on 4×4 pixel neighborhoods with 8 bins each


# SURF 
(speeded-up robust features)
Gradient-based
[eccv06](https://people.ee.ethz.ch/~surf/eccv06.pdf)


# FAST 
(features from accelerated segment test)
Intensity-based corner detector.
[eccv06](https://link.springer.com/chapter/10.1007/11744023_34)
[procedure](https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/AV1011/AV1FeaturefromAcceleratedSegmentTest.pdf)
**Remarks**:
1. Check 1,5,9,13 for fast detection;
2. Non-maximum suppression.


# BRIEF 
(binary robust independent elementary features)
[eccv10](https://www.cs.ubc.ca/~lowe/525/papers/calonder_eccv10.pdf)


# BRISK 
(binary robust invariant scalable keypoints)
[iccv11](https://ieeexplore.ieee.org/document/6126542)


# ORB 
(oriented FAST and rotated BRIEF)
[iccv11](https://ieeexplore.ieee.org/document/6126544)


# FREAK 
(fast retina keypoint)
[cvpr12](https://ieeexplore.ieee.org/document/6247715)


[Back to Top](#table-of-content)


