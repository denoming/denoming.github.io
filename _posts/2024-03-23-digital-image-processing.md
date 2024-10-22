---
title: Digital image processing
date: 2024-03-28
categories:
  - Development
  - ComputerVision
tags:
  - cv
math: true
mermaid: true
---
# Introduction

A digital image is an image composed of picture elements, also knows as pixels, each with finite, discrete quantities of numeric representation for its intensity or gray level that is an output from its 2D functions fed as input by its special coordinates denoted by x and y on the x-axis and y-axis respectively.

The amplitude of $f$ is called intensity or gray level of the point $(x,y)$.

The function $f(x,y)$ may be characterized by 2 components: 1) the amount of source illumination incident on the scene being viewed; 2) the amount of illumination reflected by the objects in the scene. These illumination and reflected components can be denoted by $i(x,y)$ and $r(x,y)$ respectively. These two functions combine to form $f(x,y)$:
$$f(x,y) = i(x,y) * r(x,y)$$
	where $0 \le i(x,y)\lt \infty$, $0\le r(x,y) \le 1 \text{(0 - total absorption), (1 - total reflection)}$ 

Three types of image processing:
* low-level (input: image, output: image) - involves primitive operations (e.g. image pre-processing to reduce noise, contrast enhancement, and image sharpening).
* mid-level (input: image, output: extracted attributes, like edges and contours) - involves segmentation of image in order to partition it into regions or object.
* higher-level - involves "making sense" of an ensemble of recognized objects, performing the cognitive functions normally associated with human vision.

Key stages in digital image processing:
* Image acquisition
* Image filtering and enhancement
* Image restoration
* Color image processing
* Wavelets and other image transforms
* Compression and watermarking
* Morphological processing
* Segmentation
* Feature extraction
* Image pattern classification

Digital image processing requires digital image. Converting analog image to digital image involves two steps:
1) sampling (digitizing the coordinate values)
2) quantization (digitizing the amplitude values)

## Relationship

![spatial-filtering](/assets/img/posts/DIP19.png)
	(a) - $N_4(p)$ (4-neighborhood); (b) - $N_d(p)$ (diagonal neighborhood); (c) - $N_8(p)$ (8-neighborhood)

$$N_8(p) = N_4(p) + N_d(p)$$
## Connectivity (adjacency)

Two pixel are connected if they are in the same class (i.e. the same color or the same range of intensity) and thy are neighbors of one another.
* 4-connectivity: $p$ and $q$ are 4-connected if $q\in N_4(p)$
* 8-connectivity: $p$ and $q$ are 8-connected if $q\in N_8(p)$
* mixed-connectivity (m-connectivity): $p$ and $q$ are m-connected if $q\in N_4(p)$ or $q\in N_D(p)$ and $N_4(p)\cap N_4(q) = \emptyset$
## Basic math operations and intensity transformations

* Distance Measures
  * The Euclidean distance: $De(p,q) = [(x-s)^2 + (y-t)^2]^{1/2}$
  * The $D_4 (p,q) = |x-s| + |y-t|$ (city-block distance) between two pixels $p$ and $q$
  * The $D_8(p,q)=max(|x-s|,|y-t|)$ (chessboard distance) between two pixels $p$ and $q$
  * The $D_m$ (depends on the path)
* Element-wise and matrix Operations
* Linear and non-linear operations: $H[F(x,y)] = g(x,y)$ (e.g. mean operator)
* Arithmetic operations ($+,-,\times,\div$)
	* $+$: addition of noisy images for noise reduction
	* $-$: enhancement of differences between images
	* $\times$: shading correction and masking or region of interest operations
	* $\div$: shading correction
* Set operations (union, intersection, complement, difference)
* Logical operations ($\land, \lor, \lnot$)
* Spatial operations
  * Single-pixel (point) operations: $s = T(x)$, where $z$ - input pixel, $s$ - output pixel, $T$ - transformation function
  * Neighborhood operations
  * Geometric spatial transformations
    * Spatial transformation of coordinates (e.g. affine transformation)
    * Intensity interpolation (assigning intensity values to spatially transformed pixels)
* Image transforms (e.g. Fourier transformation)
## Intensity transformations

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

![spatial-filtering](/assets/img/posts/DIP20.png)

Procedure:
* check the pixels in a raster scanning method
* calculate the new value for each pixel
* update the intensity
* repeat for every pixel

### Basic intensity transformation functions

Transformation function:
$$S = T(r)$$
![trans-function](/assets/img/posts/DIP21.png)
Types of transformation function:
* image inversion (digital negative): $s = (L - 1) - r$, where $L$ - maximum intensity value (good to visualize the features when black areas are dominant)
* thresholding: $s=(L-1), r\ge \text{threshold }; s=0,r\lt \text{threshold}$
* log transformation: $s = c * log(1 + r)$ (enhancing/stretching dark areas and compressing bright areas)
* power law (gamma) transformation:  $s = cr^y$ ($y < 1$ - enhancing dark areas, $y > 1$ - enhancing bright areas, $y = c = 1$ - identity transformation)
	* Application  I: Gamma correction
	* Application II: Contrast Enhancement
![power-law-transofrm](/assets/img/posts/DIP22.png)

* contrast stretching (changing the location of $(r1,s1$) and $(r2,s2)$ we can control the shape of the transformation function)
![intensity-level-slicing](/assets/img/posts/DIP18.png)

* intensity-level slicing (left - no background, right - with background)
![intensity-level-slicing](/assets/img/posts/DIP23.png)
* bit-plane slicing
![intensity-level-slicing](/assets/img/posts/DIP24.png)

## Histogram processing

**Unnormalized** histogram of $f(x, y)$ image:
$$h(r_k) = n_k,\text{for k=0,1,3,...,L-1}$$
	where $n_k$ is the number of pixels of $f$ with intensity $r_k$ which represents the intensities of an L-level image $f(x, y)$

**Normalized** histogram of $f(x,y)$ image:
$$p(r_k) = {h(r_k)\over MN} = {n_k\over MN}$$
	where $M$ and $N$ are the number of image rows and columns

The sum of $p(r_k)$ for all values of k is always 1:
$$\sum_{k=0}^{L-1}p(r_k)=1 $$
$p(r_k)$ are the probabilities of intensity levels in the image.

In an ideal image every intensity level corresponds to the same number of pixels. Histogram equalization aims to do this.

### Histogram equalization

Transformation function (intensity mapping) function:
$$s=T(r), 0\le r \le L-1$$
	where $r$ - input intensity range and $s$ - output intensity range

Function conditions:
 a) $T(r)$ is a monotonic increasing function in the interval $[0,L-1]$
 b) $0\le T(r) \le L -1,\text{for }0\le r \le L-1$.

Function $T$ is one-to-one and reversible if it is strictly monotonic ($a^1$).
![intensity-level-slicing](/assets/img/posts/DIP25.png)
($a$) - output intensity values will never be less than input values, preventing artifacts and intensity reversals
($b$) - guarantees that the range of the output intensities is the same as the input image
($a^1$) - guarantees that the mapping function from $r$ to $s$ is one-to-one and reversible

After applying histogram equalization we will obtain uniform distribution of intensity values in output images.  

The discrete transformation can be defined as follows:
$$s_k=T(r_k)=(L-1)\sum_{j=0}^{k}p_r(r_j)={L-1\over MN}\sum_{j=0}^{k}n_j,\text{for }k=0,1,2,...,L-1 $$
	where $L$ is the number of intensity levels

### Histogram matching

