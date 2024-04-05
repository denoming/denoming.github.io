---
title: "Digital image processing"
date: 2024-03-28
categories: [Development,ComputerVision]
tags: [cv]
math: true
mermaid: true
---
# Introduction

## Basic Mathematical Operations and Intensity Transformations

* Distance Measures
  * The Euclidean distance
  * The $D_4$ (city-block distance) between two pixels $p$ and $q$
  * The $D_8$ (chessboard distance) between two pixels $p$ and $q$
* Element-wise and matrix Operations
* Linear and non-linear operations $H[F(x,y)] = g(x,y)$ (e.g. mean operator)
* Arithmetic operations
* Set operations (union, intersection, complement, difference)
* Logical operations
* Spatial operations
  * Single-pixel operations $s = T(x)$, where $z$ - input pixel, $s$ - output pixel, $T$ - transformation function
  * Neighborhood operations
  * Geometric spatial transformations
    * Spatial transformation of coordinates (e.g. affine transformation)
    * Intensity interpolation (assigning intensity values to spatially transformed pixels)
* Image transforms (e.g. Fourier transformation)

### Intensity Transformations

#### Intensity transformation

  * operations on single pixels (e.g. threshold and contrast stretching)
  * the transformation is independent of locations, determined only by the intensity values

Procedure:
  * check the pixels in a raster scanning method
  * calculate the new value for each pixel
  * update the intensity
  * repeat for every pixel

### Spatial filtering

Involves neighbor pixels, typically a square shape (e.g. intensity averaging):

$$g(x,y) = T[f(x,y)]$$

Procedure:
* check the pixels in a raster scanning method
* calculate the new value for each pixel
* update the intensity
* repeat for every pixel

### Basic transformation functions

* image inversion: $s = (L - 1) - r$, where $L$ - maximum intensity value (good to visualize the features when black areas are dominant)
* log transformation: $s = c * log(1 + r)$ (enhancing/stretching dark areas and compressing bright areas)
* power law (gamma) transformation:  $s = cr^y$ ($y < 1$ - enhancing dark areas, $y > 1$ - enhancing bright areas, $y = c = 1$ - identity transformation)
* contrast stretching
* intensity-level slicing
* bit-plane slicing
