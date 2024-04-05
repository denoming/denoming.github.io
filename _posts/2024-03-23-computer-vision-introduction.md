---
title: "Computer vision introduction"
date: 2024-03-28
categories: [Development,ComputerVision]
tags: [cv]
---

# Image recognition and object detection

## Introduction

Traditional computer vision pipeline:

```mermaid
flowchart LR
    Input["Image\nImage"] --> Preprocessing
    Preprocessing --> Features["Features:\nHAAR, HOG, SIFT, SURF"]
    Features --> Learning["Learning Algorithm"]
    Learning --> Label["Label Assignment"]
```

* Step 1: Pre-processing image to normalize contrast and brightness effects
* Step 2: Extract features (e.g. HAAR like features, HOG - Histogram of Oriented Gradients, SIFT - Scale-Invariant Feature Transform, SURF - Speeded Up Robust Feature)
* Step 3: Learning algorithm (e.g. SVM - Support Vector Machine)
* Step 4: Assign label on input image
