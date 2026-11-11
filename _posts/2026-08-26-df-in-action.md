---
title: "Data Fusion In Action"
date: 2026-08-26
categories:
  - Development
tags:
  - datafusion, kalman
mermaid: true
math: true
---
# Introduction

Data Fusion is the process of integrating **multiple** data source to form useful information that is more consistent and more accurate than the original source.
Combining sensor measurements together in such a way that the resulted fusion produces a more confident estimate than the individual measurements alone could provide.

![df]({{site.utl}}/assets/img/posts/DF01.png)

![df]({{site.utl}}/assets/img/posts/DF02.png)

## Probabilistic Data Fusion

This is where the state of the dynamic system is encapsulated as a probability distribution. This distribution is updated by the distributions of the sensor measurements or other source of information to form most optimal estimate of the state of the system.


| Step                                                                                                                                                                                                                                                                       | Picture                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Probability function where the car might be.                                                                                                                                                                                                                               | ![df]({{site.utl}}/assets/img/posts/DF03.png)                                                  |
| Where the car might be after a while the initial probability<br>distribution grows with time to form predicted probability distribution.                                                                                                                                   | ![df]({{site.utl}}/assets/img/posts/DF04.png)                                                  |
| Now we get a bit of information from a censor<br>(such as GPS) which tells us an estimate of the<br>car position that the car is located somewhere<br>within measurement probability distribution.<br>Measurement distribution comes form the<br>properties of the sensor. | ![df]({{site.utl}}/assets/img/posts/DF05.png)                                                  |
| After combination the measurement distribution<br>(which comes from the censor properties) with the<br>current best estimate distribution gives updated<br>distribution.                                                                                                   | ![df]({{site.utl}}/assets/img/posts/DF06.png)                                                  |
| Keep repeating a process, evolving the distribution<br>of time while _Prediction Stage_ and then constraining it<br>with measurements<br>after each _Update Stage_.<br>Each update gives always smaller distribution than before.                                          | ![df]({{site.utl}}/assets/img/posts/DF07.png)<br>![df]({{site.utl}}/assets/img/posts/DF08.png) |
|                                                                                                                                                                                                                                                                            |                                                                                                |

![df]({{site.utl}}/assets/img/posts/DF09.png)

**The prediction step** ("time update") is used to calculate the best estimate of the state for the current time using all the available information we have at that specific time. This is where the uncertainty in the estimates grow. This step runs continuously over time driven by the system clock and sensors ticks, regardless of whether external more accurate sensor measurements arrive. Multiple prediction steps are executed sequentially overt time.
**The update state** ("measurement correction") is where the current best state estimate is update of measurements or information when they become available. This is is where the uncertainty in the estimate shrinks. This step is run when measurements from more accurate sensor arrive to correct the drift accumulated during predictions steps.

![df]({{site.utl}}/assets/img/posts/DF10.png)

In Kalman filters we are making a few assumption about the probability:

![df]({{site.utl}}/assets/img/posts/DF11.png)
![df]({{site.utl}}/assets/img/posts/DF12.png)

* Instead of the system stating: "The car is exactly at 20 meters" it says "The car is most likely at 20 meter, but it could be anywhere between 18 and 22 meters". This is modeled as a bell curve, highest point (the mean $\mu$) and the width of the curve (the variance $\sigma^2$) represents uncertainty.
* When a GPS sensor reads "24 meters", the system doesn't accept this as a perfect fact. It threads the reading as its own probability curve centered around 24 meters, shaped by the sensor's known hardware error rate.
* When the update state triggers, the Kalman filter mathematically multiplies these two overlapping bell curves together giving the third curve which is taller and narrower. This narrower variance proves mathematically that fusing data makes certain of the car's position than using either the prediction or the GPS alone.

## Key Ideas

* Represent the **estimated state** as a *probability distribution*
* Represent the measurements or observations of the current state (or function of the states) as a probability distribution
* Fuse the two distribution to get a better estimate (Bayer's Theorem)
* Prediction process increase the uncertainty with time
* Update/Measurement process decreases the uncertainty
* Kalman Filter allows use to do this numerically and mathematically simply by making a few assumptions about the probability.

# Probability

## Basic Probability

Probability is a mathematical way to describe the likelihood of an event happening.
For any Event A, the probability is

$$\boxed{0 \le P(A) \le 1}$$

where $P(A)=0$: The Event A will never occur and $P(A)=1$ The Event A absolutely certainty will occur.
The probability of not occurring:

$$\boxed{P(\neg A) = 1 - P(A)}$$

Example: Single coin toss, where  the set of possible events is $S=\lbrace H,T \rbrace$,  with $P(H) = 0.5$ and $P(T)=0.5$.


Sum of all probabilities of each event or outcome in set $S$ must equal 1:
$$\boxed{\sum_{e\in S} P(e) = 1}$$

Probability of event A occurring, **assuming each event is equally likely**:
$$\boxed{P(A) = \frac {\text{Number of ways EventA occurs}} {\text{Total number of outcomes}}}$$

**Example:** Double coin toss events has the following space $S=\lbrace HH,HT,TH,TT \rbrace$, where $P(HH)=\frac 1 4$ and in general $P(A)=\frac {N(A)} {N(S)}$, where $N(A)$ is a number of possible ways event A can occur and $N(S)$ is a total number of events in event space.

## Mutual Exclusivity

**Mutual exclusive** events mean that only one event from the given events set $S=\lbrace A,B \rbrace$ can occur at the same time:

$$\boxed{P(A \cap B) = 0}$$
**Non-mutually exclusive** events mean that at the same more than one event can occur:

$$\boxed{P(A \cap B) \ne 0}$$

__In general probability of mutual exclusive events:__

$$\boxed{P(A \cup B) = P(A) + P(B) = 1}$$

Example: Single coin toss, where  the set of possible events is $S=\lbrace H,T \rbrace$, events $H$ and $T$ are mutually exclusive.

Events do not have to be mutually exclusive, it is possible to get different events occurring at the same time. In this case events are not mutually exclusive: $P(\text{A and B}) \neq 0$.


__In general  the probability of non-mutually exclusive events:__

$$\boxed{P(A \cup B) = P(A) + P(B) - P(A \cap B) = 1}$$

Example: Given a deck of card with $S_{number}=\lbrace A,1,2,3,4,5,6,7,8,9,10,J,Q,K\rbrace$ and $S_{suit} = \lbrace Spades,Hearts,Diamonds,Clubs \rbrace$. According to the rules $P(\text{King and Hearts}) \neq 0$ therefore $P(\text{King or Heart}) = P(King) + P(Heart) - P(\text{King and Heart})$.

## Conditional Probability

* Events can be considered **independent** if the likelihood of one event does not affect the likelihood of another occurring (e.g. roll of a dice, toss a coin)
* **Dependent** events are the opposite. When one event occurs it changes the probability of the other events.

Example: 
A deck of card where $P(red) = 26/52 = 0.5$ and $P(black) = 26/52 = 0.5$.
When we pick any card from a full deck of card at random, there is a 50/50 chance that it is a black or red card. Let's say we pull out a three of hearts. And if we put if back and pull out a different card, this time we pull out a black seven of clubs. We still have 50/50 chance of being black or red. In this case events are considered as independent events.
If we don't replace a card, so we leave it pulled out, the probability is changed: $P(red) = \frac {26} {52-1} = 0.51$ and $P(black) = \frac {26-1} {52-1} = 0.49$. In this case events are considered as dependent events.

__In general the probability of dependent events:__

$$\boxed{P(A \cap B) = P(A) P(B \mid A) = P(B) P(A \mid B)}$$

Using this formula the probability of drawing a red card followed by a black card without replacing the cards after they are drawn are:
$$P(red \cap black) = P(red)P(black|red)=(26/52)(26/51)=0.255$$

**Conditional probability:**

$$\boxed{P(A\mid B) = \frac {P(A \cap B)} {P(B)}}$$


(probability of A occurring given probability B has occurred is equal probability of events A and B occurring divided by probability of event B occurring).

Where $P(A\mid B)$ is referring as **conditional probability**, $P(\text{A and B})$ - **joint probability**, $P(B)$ - **marginal probability**.

**Independent events:**

$$\boxed{P(A \cap B) = P(A)P(B)}$$

* $P(A\mid B) = P(A)$ which mean there is no dependency on $B$
* $P(B \mid A) = P(B)$ which mean there is no dependency on $A$

 Example: 

![df]({{site.utl}}/assets/img/posts/DF13.png)

## Bayes Theorem

* One of the most important concepts used in Bayesian Estimation (i.e. Probabilistic Estimation)
* It allow to calculate the likelihood or bounds on an unknown parameter or event based on prior information related to that event (Bayesian inference)

From conditional probability:

$P(A\mid B) = \frac {P(A \cap B)} {P(B)}$ and $P(B\mid A) = \frac {P(B \cap A)} {P(A)}$ 

Given that $P(\text{A and B}) = P(\text{B and A})$ according to  joint probability it will give us **Bayes Theorem's** equation:

$$\boxed{P(A\mid B) = \frac {P(B\mid A) P(A)} {P(B)}}$$

Where $P(A\mid B)$ - conditional probability (likelihood of Event A given Event B occurs, $P(B\mid A)$ - likelihood of Event B given Event A occurring, $P(A)$ - marginal probability (likelihood of Event A occurring, $P(B)$ - marginal probability (likelihood of Event B occurring).

In context of Bayes Theorem $P(A\mid B)$ is a **posterior probability**, $P(B\mid A)$ is a **likelihood**, $P(A)$ is **prior probability**, $P(B)$ is a **evidence**.

![df]({{site.utl}}/assets/img/posts/DF18.png)

Example:

Prior information: $P(\text{rain}) = 0.6$
(i.e. its rained 18 days out of the 30 days)

Evidence: $P(cloudy) = 0.48$
(i.e. you look outside and say there is a 48% change of it being cloudy this morning)

Likelihood: $P(\text{cloudy|rain}) = 0.68$
(i.e. on the days that it has rained, 68% of the time it was cloudy in the morning)

Posterior probability:
$$P(\text{rain|cloudy}) = \frac {P(\text{cloudy|rain}) P(rain)} {P(cloudy)} = 0.85$$

## Random Variable

* A random variable is a way to mathematically express stochastic outcomes as real number.
* Function mapping that maps a set of experimental outcomes to a set of real numbers.

$$X : S \to E$$

Where $X$ is a **random variable**, $S$ - set of outcomes and $E$ is a set of real number (e.g. $E= \lbrace ...1,2,3,... \rbrace$ for discrete case or $E= \lbrace ...1.01,3.14, ... \rbrace$ for continuous case).

![df]({{site.utl}}/assets/img/posts/DF14.png)

## Probability Density Functions

* Having a random variable $X$, we would to know how likely each outcome is to occur, Is each outcome equally as likely (e.g. toss of a coin) or are some outcomes more likely to occur than others.
* **Probability Density Functions** describes and quantify this.

Measure the relative likelihood or probability that a specific outcome will occur for different random variable types:
* **Probability Density Function (PDF)** for continuous random variable
* **Probability Mass Function (PMF)** for discrete random variable

Example:

|                       |                                                                                         |                                               |
| --------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------- |
| Dice Roll             | $P(X=1)=1/6$<br>...<br>$P(X=6)=1/6$                                                     | ![df]({{site.utl}}/assets/img/posts/DF15.png) |
| Weighted<br>Dice Roll | $P(X=6)=0.75$<br>$P(X=1)=0.05$<br>...<br>$P(X=5)=0.05$<br>$P(\neg 6) = 1 - P(X=6)=0.25$ | ![df]({{site.utl}}/assets/img/posts/DF16.png) |

General form:
$$\boxed{p_X(x_i)=P(X=x_i)}$$


Difference between PDF and PMF:

![df]({{site.utl}}/assets/img/posts/DF16.png)

| **Core Feature**            | **Probability Mass Function (PMF)**                           | **Probability Density Function (PDF)**                                          |
| --------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Variable Type**           | **Discrete** (countable values, e.g., coin flips, dice rolls) | **Continuous** (uncountable range, e.g., height, temperature, time)             |
| **Output Meaning**          | Direct **Probability** at a single point: $P(X = x) = f(x)$   | **Probability Density** (relative likelihood per unit)                          |
| **Point Value**             | $0 \le P(X = x) \le 1$                                        | $P(X = x) = 0$ for any exact single point                                       |
| **Probability Calculation** | $p_X(x_i) = P(X=x_i)$<br>$\sum p_x(x_i) = 1$                  | $P(a \le X \le b) = \int_a^b f_X(x) dx$<br>$\int_{-\infty}^{\infty} f_X(x)dx=1$ |
| **Max Value Limit**         | Cannot exceed 1                                               | **Can exceed 1** (since it measures density, not probability)                   |
| **Visual Representation**   | Bar graph or spike plot                                       | Continuous curve (area under the curve equals probability)                      |

* **PMF gives direct probability:** the probability of rolling a 4 on a standard die, the PMF gives the exact answer directly ($1/6$).
* **PDF requires an interval:** since there are infinitely many real numbers between any two points. The probability of measuring a continuous variable at _exactly_ a single value (e.g., exact height of 175.0000... cm) is effectively zero. The PDF gives us a likelihood. Therefore, we should integrate the PDF over a range (e.g., between 174.5 cm and 175.5 cm) to find the probability.

## Expectation Operator

The **expected value** of a random variable is its **long-run theoretical average**. It represents the average value which we expect to receive after running an experiment $N$ times.

Let $X$ be a discrete random variable and we will test the value $N$ time. We observe the value $A_1$ occur $n_1$ times and the value $A_2$ occur $n_2$ times and so forth to value $A_m$ occurring $n_m$ times. The expected value $E(X)$ of the random variable $X$ is computed with:

$$\boxed{E(X) = \frac 1 N \sum_{i=1}^m A_i n_i}$$

Also, there is another term called **expected frequency** (count) which represents how many times a specific event or success will happen over $N$ trials. It might be calculated using **Binomial distribution** formula as $n \cdot p$. Where $p$ is the probability of this event and $n$ is the number of trials.

Example:
Rolling a dice $N$ times. The probability to receive each face is equal to $1/6$. Therefore, the expected frequency of each value is equal: $E[X_i]=N \cdot \frac 1 6$. Expected value of random value (long-run average):

$$E[X]=\frac 1 N [(1) \frac N 6 + (2) \frac N 6 + ... + (6) \frac N 6]$$


Expected value represented through the probability for different variable types:

| **Type**       | **Formula**                                                                                                                     | **What It Means**                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Discrete**   | $E(X) = \frac 1 N \sum_{i=1}^m A_i n_i$<br>or<br>$E[X] = \sum x \cdot P(X = x)$<br>where $x = A_i$ and $P(X=x) = \frac {n_i} N$ | Sum of each value weighted by its probability.                              |
| **Continuous** | $E[X] = \int_{-\infty}^{\infty} x \cdot f(x) \, dx$                                                                             | Integral of each value weighted by its probability density function $f(x)$. |

The expected value or mean of the random variable is usually for simplicity written as:
$$\boxed{\overline{X} = \overline{x} = E(X)}$$


## Distribution Statistical Properties

It is useful to be able to describe a PDF or distribution using some key properties:

|                                    Measure                                     |                               Formula                                |                     Graph                     |
| :----------------------------------------------------------------------------: | :------------------------------------------------------------------: | :-------------------------------------------: |
|                  Mean aka `First moment`<br>(expected value)                   |                        $\overline{x} = E(X)$                         | ![df]({{site.utl}}/assets/img/posts/DF19.png) |
|   Variance aka `Second Moment`<br>(the measure of spread away from the mean)   |    $\sigma_X^2=E[(X-\overline x)^2] = ... = E(X^2)-\overline x^2$    | ![df]({{site.utl}}/assets/img/posts/DF20.png) |
| Skew aka `Third Moment`<br>(a measure of how asymmetrical the distribution is) | $skew=E[(X-\overline{x})^3]$<br>$skewness=\frac {skew} {\sigma^3_X}$ | ![df]({{site.utl}}/assets/img/posts/DF21.png) |

* $\sigma_X$ is a standard deviation
* $\sigma_X^2$ is a variance

Describing a distribution using mean and variance:

$$\boxed{X \sim (\overline{x},\sigma^2_X)}$$

## Uniform Distribution (Continuous)

* Notation: $\boxed{X \sim U(a,b)}$
* Mean: $\overline{x}=\frac 1 2 (a+b)$
* Variance: $\sigma^2_X=\frac 1 {12}(b - a)^2$
* Skew: $skewness = 0$ (distribution is symmetrical)

![df]({{site.utl}}/assets/img/posts/DF22.png)

## Gaussian Distribution (Continuous)

Notation: $\boxed{X \sim N(\mu,\sigma^2)}$

![df]({{site.utl}}/assets/img/posts/DF23.png)


**Important property: Any linear transformation of the Gaussian distribution is another Gaussian distribution.**

## Transformation of Random Variable

Random variables can be transformed through a function. Suppose we have a random variable $X$ and its associated PDF $f_X(x)$, it is possible to apply a mathematical function $y=g(x)$ and find the PDF $f_Y(y)$ of the transformed random variable $Y$.



If $g$ is not strictly monotonic (e.g. $y=0.25$ comes from $x_0=-0.5$ and $x_1=+0.5$):

$$\boxed{f_Y​(y)= \sum_i f_X​(x_i​) \vert \frac d {dy}g^{-1}(x_i) \vert}$$

where $x_i$ ranges overt all solutions of $g(x) = y$.

If $g$ is strictly monotonic:

$$x=g(y)^{-1}=h(y)$$

$$\boxed{f_Y(y) = f_X(h(y)) |h'(y)|}$$


Intuitively: to find the density $Y$ at a point $y$, we find the corresponding point $x=g^{-1}(y) = h(y)$ in the original space, take the density there and then re-scale by how much $g$ "stretches" or "compresses" space near that point). The absolute value is there because density must stay non-negative event if $g$ is decreasing (which would otherwise flip the sign.


Example: 

Suppose we have the random variable: $X \sim N(\overline{x}, \sigma^2_x)$ and its PDF $f_X(x) = \frac 1 {\sigma_x \sqrt{2\pi}} \exp [-\frac 1 2(\frac {x-\overline{x}} {\sigma_x})^2]$

Suppose also we have the transformation:
$$Y = g(X) = aX + b$$

$$X=g^{-1}(Y)=\frac {Y-b} a$$

$$h(Y) = \frac {(Y-b)} a$$

$$h'(y) = \frac 1 a$$

As the transformation is monotonic we can find $f_Y(y)$ as:

$$f_Y(y) = f_X(h(y)) |h'(y)|  = \frac 1 {a\sigma_x \sqrt{2\pi}} \exp [-\frac 1 2(\frac {y-(a\overline{x}+b)} {a\sigma_x})^2] $$

So we have new variance scaled by $a$ and new mean $a\overline{x} + b$ but **the PDF remains being Gaussian PDF**:

$$Y \sim N(a\overline{x}+b, a^2\sigma^2_x)$$

## Multiple Random Variables


Let $X$ be a random variable with a PDF $f_X(x)$ and also let $Y$ be a random variable with a PDF $f_Y(y)$. It is possible to define the PDF of the joint probability (i.e. probability of $X$ and $Y$) as $f_{XY}(x,y)$ or $f(x,y)$ for short.

Joint probability:

$$P(a \le X \le b \space \text{and} \space c \le Y \le d) = \int_c^d \int_a^b f(x,y) \space dx \space dy$$

Marginal density functions:

$$f_X(x) = \int_{-\infty}^\infty f(x,y)\space dy$$

$$f_Y(y) = \int_{-\infty}^\infty f(x,y)\space dx$$

Expected value for multiple random variable:

$$E[g(x,y)]=\int_{-\infty}^{\infty} \int_{-\infty}^{\infty} g(x,y) f(x,y) \space dx \space dy  $$

Density function condition ($X$ and $Y$ are independent random variables, so $P(A \text{ and } B) = P(A)P(B)$):

$$f_{XY}(x,y) = f_X(x) f_Y(y)$$

Expected value of multiplication ($X$ and $Y$ are independent random variables):

$$E[XY] = \int \int XYf(x,y)\text{ dx dy} $$

$$E[XY] = \int \int XYf_X(x) f_Y(y)\text{ dx dy} $$

$$E[XY] = \int XYf_X(x) \space dx \int f_Y(y)\space dy $$

$$E[XY] = \int Xf_X(x) \space dx \int Yf_Y(y)\space dy $$

$$E[XY] = E(X) E(Y)$$

Expected value of sum ($X$ and $Y$ are independent random variables):

$$z(x,y) = g(x) + h(y)$$

$$E[z(x,y)] = E[g(x) + h(y)]$$
$$E[z] = \int \int [g(x) + h(x)] f(x,y) \space dx \space dy $$
$$E[z] = \int \int g(x)f_X(x)f_Y(y) \space dx \space dy + \int \int h(y)f_X(x)f_Y(y) \space dx \space dy $$

where $\int f_Y(y) \space dy = 1$ and $\int f_X(x) \space dx = 1$, therefore:

$$E[z(x,y)] = E[g(x)] + E[h(y)]$$

**The random variables might not be independent**, there might be some correlation between the two.

* Covariance of two dependent variables: $C_{XY} = E[(X - \overline{x})(Y - \overline{y})] = E(XY) - \overline{x}\space\overline{y}$
* Correlation coefficient: $\rho = \frac {C_{XY}} {\sigma_x \sigma_y}$ ($-1$ negatively correlated, $+1$ positively correlated, $0$ - not correlated)

![df]({{site.utl}}/assets/img/posts/DF30.png)

## Multivariate Statistics

Single random variables can be generalize into vector form:


|                  What                  |                                               |
| :------------------------------------: | --------------------------------------------- |
| Generalized single<br>random variables | ![df]({{site.utl}}/assets/img/posts/DF24.png) |
|           Expected<br>value            | ![df]({{site.utl}}/assets/img/posts/DF25.png) |
|       Multivariate<br>Covariance       | ![df]({{site.utl}}/assets/img/posts/DF26.png) |
|       Autocorrelation<br>Matrix        | ![df]({{site.utl}}/assets/img/posts/DF27.png) |

The covariance matrix (autocorrelation) properties:
* Symmetric: $\sigma_{ij} = \sigma_{ji}$
* Positive semi-definite: $z^TC_Xz\ge 0$
* $C_X = C_X^T$

## Multivariate Gaussian Distribution


![df]({{site.utl}}/assets/img/posts/DF28.png)


The mean shifts the center of the distribution, the variance controls the spread in the different axes, while the cross-covariances control the orientation of the distribution:

![df]({{site.utl}}/assets/img/posts/DF29.png)

## Linear Transformation of Uncertainties

One random variable:
$$X\sim N(\overline{x}, \sigma_x^2) \Longrightarrow Y=aX+b \Longrightarrow Y \sim N(a\overline{x} +b,a^2\sigma^2_x)$$

Random vector:

$$X\sim N(\overline{X}, C_X)$$

$$Y= g(X) = AX + b$$

$$X = g^{-1}(Y) = A^{-1}Y - A^{-1}b$$

$$h(Y) = A^{-1}Y - A^{-1}b$$

$$h'(y) = A^{-1}$$


where $\bar Y = A\bar X + b$ and $C_Y = AC_X A^T$, therefore:

$$f_Y(Y) = \frac 1 {(2\pi)^{n/2} |AC_XA^T|^{1/2}} \exp[-\frac 1 2(Y-\bar Y)^T(AC_XA^T)^{-1}(Y-\bar Y)] $$


Therefore, in case of random variable we end up with Gaussian PDF as well:

$$X\sim N(\bar X, C_X) \Longrightarrow Y=AX+b \Longrightarrow Y \sim N(A \bar X + b,AC_XA^T)$$

If $C_X$ represents the uncertainty covariance, then it can be transformed to another frame using the linear transformation $y=Ax$ where the transformed covariance is given by $C_Y = AC_XA^T$.

# Dynamic Systems

## Differential Equations

Types:
* Ordinary Differential Equations: $\frac d {dx} f(x)$ (e.g. $a = \frac {dv} {dt} = \frac {d^2p} {dt^2}$)
* Partial Differential Equations: $\frac {\partial} {\partial x} f(x, y)$ (e.g. $\frac {\partial^2 u} {\partial x^2} + \frac {\partial^2 u} {\partial y^2}$)

Order of differential equation:
* First order: $\frac {du} {dx} = a + u^2$
* Second order: $\frac {d^2 u} {dx^2} - \frac {d u} {dx} + u = 0$

Linear system (the system which output changes proportionally with the input) and has the following properties:
* Additivity (Superposition): If an input $u_1(t)$ produces an output $y_1(t)$, and an input $u_2(t)$ produces $y_2(t)$, then the input $u_1(t) + u_2(t)$ will produce the exact output $y_1(t) + y_2(t)$
* Homogeneity (Scaling): If an input $u(t)$ produces an output $y(t)$, then scaling the input by a constant $c$ scales the output by the same constant: $c \cdot u(t) \to c \cdot y(t)$.

Linear ODE is an equation where both sides are a **linear combination** of the dependent variables and its derivatives.

General form: $a_n(x) \frac {d^ny} {dx^n} + ... + a_1(x)\frac {dy} {dx} + a_0(x)y = b(x)$
Linear ODE: $\frac {dy} {dx} + cy + x^2 = 0$
Non-Linear ODE: $\frac {dy} {dx} + a\sin(y) = 0$ (dependent variable $y$ is inside non-linear sine function)

## State space representation

* A system is a collection of interrelated entities (or differential equations) that can be considered as a whole.
* Any system whose internal state, output or process change over time is defined as a **dynamic system**.
* Unlike **a static system**, where the output at any given moment depends strictly on the current input - **a dynamic system** has memory, meaning its present behavior is influenced by its history and previous states.
* The differential equations that make up the system are the **state equations** of the dynamic system. The **state variables** of the system are the **dependent variables** of the **state equations**.

Newton's 'Dot' Notation: $\dot x = \frac {dx} {dt}$ (derivative w.r.t. time)

* $\dot x_1 = f_1(t,x_1,...,x_n,u1,...,u_m)$
* $\dot x_2 = f_2(t,x_1,...,x_n,u1,...,u_m)$
* ...
* $\dot x_n = f_n(t,x_1,...,x_n,u1,...,u_m)$

Where $t$ - time varying, $\dot x_1...\dot x_n$ - n-state equations, $x_1,...,x_n$ - n-states (current system condition), $u1,...,u_m$- m-inputs (forces which drives changes).

This can be compactly written in a state-space form:

$$\boxed{\dot x(t) = f(t,x(t),u(t))}$$

Where $\dot x(t)$ is a column vector holding all the rates of change, $x(t) = [x_1(t),..., x_n(t)]^T$ is a **state vector**, $u(t) = [u_1(t),...,u_m(t)]^T$ is **input vector** (forces driving changes).

* $\dot x(t) = f(t,x(t),u(t))$ - the general (nonlinear) form ($f$ could involve things like $x^2$ or $sin(x)$)
* $\dot x(t) = Ax(t) + Bu(t)$ - the linear (special, restricted) form

* Many of the processes that occur in the world can be expressed as linear or non-linear differential equations (most common).
* Because real systems almost always are non-linear, Kalman filters heavily rely on **linearization** (suing Jacobians) to approximate a messy $f(t,x,u)$ into clean, manageable matrices $A$ and $B$ to run calculation efficiently on computers.
* We can describe these processes in the state-space form which then allow us to use different **mathematical tools** to extract useful information and perform various analysis of the system
* If we know the state of the system for the current time and all the current and future inputs to the system, then we can predict the values of the future states and future outputs of that system and a lot more.

## Dynamic Systems and State Space

* First-Order differential equations make up the **state equations**: $\dot x(t) = f(t,x(t),u(t))$
* The **state variables** of the system are the dependent variables of the state equations: $x(t)$
* How can we represent different ODEs in a state space form?
	* Newton's equations of motion: Force = Mass x Acceleration: $F(t) = m \ddot x(t)$
	* How does the position $x(t)$ evolve with time for a given force $F(t)$?
	* The problem is the second order of the Newton's equation (we have acceleration in the formula).
	

To represents Newton's equation $F(t) = m \ddot x(t)$ as state equation $\dot x(t) = f(t,x(t),u(t))$:

* State Rates Vector (a rate of changes vector):  $\dot x(t) = \begin{bmatrix} \dot v(t) \\ \dot x(t) \end{bmatrix}$
* State Vector: $x(t) = \begin{bmatrix} v(t) \\ x(t) \end{bmatrix}$ (current state at particular point in time: position and velocity)
* Input Vector: $u(t)=[F(t)]$

Non-linear state-space form of Newton's equation:

$$ \begin{bmatrix} \dot v(t) \\ \dot x(t) \end{bmatrix} = \begin{bmatrix} \frac 1 m F(t) \\ v(t) \end{bmatrix} $$

(the left part always represents the rates of change of dynamic system state - how fast things are shifting right now, the right side includes: current state/condition of the system and forces/input which drives these changes)

$$\begin{bmatrix} \text{How fast velocity is changing } (\dot{v}) \\ \text{How fast position is changing } (\dot{x}) \end{bmatrix} = \begin{bmatrix} \text{The forces driving acceleration} \\ \text{Your current velocity} \end{bmatrix}$$
Intuition:

"If you want to know how my state (speed and position) is going to evolve over the next fraction of a second, look at the left side. It tells you the speeds of change. The right side tells you the rules: change in position is driven by current speed, and change in speed is driven by external force."

## Continuous and discrete time

* The differential equations shown before have been shown in "continuous" time (defined w.r.t. an independent variable $t$ which varies continuously and smoothly)
* For practical purposes we need to know the state of the system at a discrete set of point in time: $t_0, t_1,...,t_{k-1},t_k,t_{k+1},...$, where $k$ - represents particular step in time equal to $0.1s$ or other amount of time interval (this is what is called "discrete" time).

![df]({{site.utl}}/assets/img/posts/DF31.png)

* Time steps forward in discrete blocks of time
* Timestamp/Sample time is how large the blocks of time are

* State equation where $t$ is continuous: $\dot x(t) = f(t,x(t),u(t))$
* Discrete time representation of a dynamic system: $\boxed{x(t_{k+1}) = f(t_k, x(t_k),u(t_k))}$ where $t_k=k\Delta t$ (it calculates not rate of changes, it calculates the next state)

## Mathematical models of dynamic systems


State space representation:
* Continuous
	* Non-linear (very general representation): $\boxed{\dot x(t) = f(t,x(t),u(t))}$ (represents a general function, it can be linear, it can be non-linear, it is time varying, input is current states and control inputs, outputs continuous state writes)
	* Linear (continuous time linear dynamic system representation): $\boxed{\dot x(t) = A(t)x(t) + B(t)u(t)}$ (it is also time varying, outputs continuous state writes, linear combination using matrix multiplication of system states and control inputs, matrices are time varying)
* Discrete:
	* Non-linear (general non-linear representation for discrete time): $\boxed{x_{k+1} = f(t_k,x(t_k),u(t_k))}$
	* Linear: $\boxed{x_{k+1} = F_kx_k + G_ku_k}$


Models classification:

![df]({{site.utl}}/assets/img/posts/DF32.png)

## Continuous to discrete model conversions

We have continuous, time invariant, linear system: $\dot x(t) = Ax(t) + Bu(t)$. From engineering purposes we want to know the system's state at particular points n time. Therefore, we need discrete, time invariant linear system: $x_{k+1} = F_kx_k + G_ku_k$.

$$(A,B) \rightarrow (F,G)$$

Continuous, time invariant, linear system:

$$\dot x(t) = Ax(t) + Bu(t)$$

Solution:  $x(t) = e^{A(t-t_0)}x(t_0) + \int_{t_0}^t e^{A(t-\tau)}Bu(\tau)d\tau$ where $x(t)$ is the state of the system in future, $e^{A(t-t_0)}x(t_0)$ is a free response and $\int_{t_0}^t e^{A(t-\tau)}Bu(\tau)d\tau$ is a force response, it might be present or not.
This solution is saying that we can work out the state of the system at some time in the future based on matrix exponential equation and integration.

Consider a discrete time steps:
* $t=t_k$
* $t_0 = t_{k-1}$
* $\Delta t = t_k - t_{k-1}$

If we take these discrete time representation and do a substitution we end up with the following equation: $\boxed{x(t_k) = e^{A\Delta t}x(t_{k-1}) + \int_0^{\Delta t} e^{-A\alpha} d\alpha \space Bu(t_{k-1})}$. Comparing with $x_{k+1} = F_kx_k + G_ku_k$ we can conclude:

* Free response: $\boxed{F=e^{A\Delta t}}$
* Forcing response: $\boxed{G=\int_0^{\Delta t} e^{-A\alpha} d\alpha \space B = F[I-e^{-A\Delta t}] A^{-1}B \text{  if} A^{-1} \text{exist}}$ 

How matrix exponential can be calculated?

$e^{At} = \sum_{j=0}^{\infty} \frac {(At)^j} {j!} = (At)^0 + (At)^1 + \frac {(At)^2} {2!} ...$

We can use first order approximation: $\boxed{e^{At} \approx I + At}$


## Least Squares Estimation

### Estimation of a scalar constant


![df]({{site.utl}}/assets/img/posts/DF33.png)

k-measurements ($y_k$ - measured temperature, $x$ - constant true temperature, $v_k$ - noise which corrupts true temperature):
* $y_1 = x + v_1$ 
* ...
* $y_k = x + v_k$

Assumptions: noise on the measurement is zero mean, uncorrelated white noise:
* $E(V) = 0$
* $E(v_iv^T_j) = 0$

$\bar y = \frac 1 k \sum_{i=1}^ky_i = \frac 1 k \sum_{i=1}^k (x+v_i) = x + \frac 1 k \sum^k_{i=1}v_i$, where $\frac 1 k \sum^k_{i=1}v_i = E(V) = 0$ 


Having enough measurements we can way that the best estimate is given by: $\hat x = \bar y$ because the expected value of all noise values is going to be zero. The number of measurements that need to be made will depend on the required accuracy of the estimates and the size of the noise distribution from the sensor.

### Least Squares

![df]({{site.utl}}/assets/img/posts/DF34.png)

k-measurements ($y_k$ - measured temperature, $x_1$ - slope parameter, $x_2$ - bias parameter, $r_k$ - RPM measurements, $v_k$ - noise which corrupts true temperature):

* $y_1 = x_1r_1 + x_2 \cdot 1 + v_1$ 
* $y_2 = x_1r_2 + x_2\cdot 1 + v_2$
* ...
* $y_k = x_1r_k + x_2\cdot 1 + v_k$

In general $x$ represents a vector of constant values, each value is a slope. The goal is to estimate these constant values. Matrix representation of the equations above is $y=Hx + v$

![df]({{site.utl}}/assets/img/posts/DF35.png)

Assumptions: noise on the measurement is zero mean, uncorrelated white noise:
* $E(v) = 0$
* $E(v_iv^T_j) = 0$

Instead of solving $y-Hx = v$ with respect to true state $x$, we can solve it for the estimated state $\hat x$. We do this, by looking at the measurement residual $\epsilon = y - H \hat x$ (differences between the real measurements $y$ and the expected measurements $H\hat x$ which based on estimated state). If estimated $\hat x$ is spot on, the residual is just the sensor's noise $v_k$. If the estimate is wrong, the residual contains both the noise and the error of the estimation.

Re-arranged matrix equation to work out different measurements residual for each of the different measurements:

![df]({{site.utl}}/assets/img/posts/DF36.png)

If we can make the vector $\epsilon$ as small as possible (in magnitude), then the estimate $\hat x$ should be as close to the true value of $x$ that we can get. To do this we define cost function: $J = \epsilon_1^2 + ... + \epsilon_k^2 = \epsilon^T\epsilon$. The larger residual, the larger the cost function, the smaller the cost function, the smaller residual.

* $J = \epsilon^T\epsilon$
* $J = (y - H\hat x)^T(y - H\hat x)$
* $J = y^Ty - \hat x^TH^Ty - y^TH\hat x + \hat x^TH^TH\hat x$

To minimize cost functions we need to differentiate w.r.t. to $\hat x$ and set the derivative to zero:

* $\frac {\partial J} {\partial \hat x} = -y^TH-y^TH+2\hat x^TH^TH = 0$ after re-arranging the equation:
* $\boxed{\hat x = (H^TH)^{-1}H^Ty}$

The matrix $H$ must be full rank and the $(H^TH)^{-1}$ must exist. So the number of measurements $k$ must be greater that the number of elements in the $\hat x$ that we want to estimate.

Example:

![df]({{site.utl}}/assets/img/posts/DF37.png)

* $(H^TH) = \begin{bmatrix} 55 & 15 \\ 15 & 5 \end{bmatrix}$
* $(H^TH)^{-1} = \begin{bmatrix} 1/10 & -3/10 \\ -3/10 & 11/10 \end{bmatrix}$
* $\hat x = (H^TH)^{-1}H^Ty = \begin{bmatrix} \frac {91} {10} \\ \frac {527} {10} \end{bmatrix}$
* $\boxed{t=9.1r + 52.7}$

### Weighted Least Squares


![df]({{site.utl}}/assets/img/posts/DF38.png)


Assumptions: noise on the measurement is zero mean, uncorrelated white noise, but each value can have a different variance:
* $E(V) = 0$
* $E(v_iv^T_j) = 0$
* $E(v_i^2) = \sigma_i^2$ (shows how much uncertainty we have on each measurements)
* $E(vv^T) = R =\begin{bmatrix} \sigma_1^2 & 0 & \cdots & 0 \\ 0 & \sigma_2^2 & \cdots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \cdots & \sigma_k^2 \end{bmatrix}$

Problem model:
* $y = Hx+v$
* $E(v_i^2) = \sigma_i^2$
* $E(vv^T) = R$

Measurement Residual:
* $\epsilon_i = y_i - H_i \hat x$

Cost function:
* $J = \frac {\epsilon_1^2} {\sigma_1^2} + ... + \frac {\epsilon_k^2} {\sigma_k^2} = \epsilon^T R^{-1} \epsilon$ 
* The more uncertainty we have on each measurement, the less effect it's going to have on cost function.

Best estimate:
* $\hat x = min J$

Expand const function:
* $J = \epsilon^T R^{-1} \epsilon = (y - H\hat x)^TR{-1}(y-H\hat x) = ...$

Find minimum:
* $\frac {\partial J} {\partial \hat x} = -y^TR^{-1}H + \hat x^TH^TR^{-1}H = 0$
* $\boxed{\hat x = Ay = (H^TR^{-1}H)^{-1}H^TR^{-1}y}$ (we want to work out how errors in $y$ affects errors in $x$)

Transformation of uncertainty:
* $f=Ax$
* $\sum_f = A\sum_x A^T$

Solution uncertainty:
* $P=ARA^T = (H^TR^{-1}H)^{-1}$ (get uncertainty in $y$ and transform it to the uncertainty of $x$)

The matrix $P$ is an error covariance matrix. The error represents the difference between the true value and an estimated value ($\tilde x = x - \hat x$). Therefore, the matrix contains co-variances (off-diagonal values) and variances (diagonal values) for all components of the state vector.

### Recursive Least Squares

Previous methods requires all the measurements to be available at time that we calculate the solution. What happens if we don't?

Measurement:
* $y_k = H_kx + v_k$ (single measurement)
* $E(v_kv_k^T) = R_k$ (we have certain co-variance of the noise)
* $E(V) = 0$ (zero mean)
* $E(v_iv_j^T) = 0$ (there is no correlation)

First we have initial estimate $\hat x_0$, then we gather a measurement we update our estimate and obtain $\hat x_1$, then we gather next measurement and again update our estimate to get $\hat x+2$ and so forth.

 $\boxed{\hat x_k = \hat x_{k-1} + K_k(y_k - H_k\hat x_{k-1})}$, where $\hat x_{k-1}$ - previous estimate and $(y_k - H_k\hat x_{k-1})$ - difference between actual measurement and predicted estimate (error term), $K_k$ - correction gain matrix that makes the minimizes the least square cost function.

Recursive estimation error:
* $\epsilon_k=x - \hat x_k$, where $x$ - true state and $\hat x_k$ is the estimate of current state
* $\epsilon_k=x - \hat x_{k-1} - K_k(y_k - H_k\hat x_{k-1})$
* $\epsilon_k=\epsilon_{k-1} - K_k(H_kx + v_k - H_k\hat x_{k-1})$
* $\epsilon_k = (I - K_kH_k)\epsilon_{k-1} - K_kv_k$

Recursive uncertainty:
* $P_k=\epsilon_k \epsilon_k^T$
* $P_k = [(I-K_kH_k)\epsilon_{k-1} - K_kv_k][...]^T$
* $P_k=(I-K_kH_k)P_{k-1}(I-K_kH_k)^T + K_kR_kK_k^T$

Cost function:
* $J_k = (x_1 - \hat x_1)^2 + ... + (x_k - \hat x_k)^2$
* $J_k = \epsilon_1^2 + ... + \epsilon_k^2$
* $J_k = \epsilon_k^T \epsilon_k$
* $J_k=Tr(P_k)$

Find minimum:
* $\frac {\partial J_k} {\partial \hat K_k} = -2(I-K_kH_k)P_{k-1}H_k^T + 2K_kR_k = 0$
* $\boxed{K_k = P_{k-1}H_k^T(H_kP_{k-1}H_k^T+R_k)^{-1}}$

Initial guess:
* $\hat x_0 = E(x)$
* $P_0 = E[(x - \hat x_0)(x - \hat x_0)^T]$

Recursive least squares:
* $K_k = P_{k-1}H_k^T(H_kP_{k-1}H_k^T+R_k)^{-1}$
* $\hat x_k = \hat x_{k-1} + K_k(y_k - H_k\hat x_{k-1})$
* $P_k = (I - K_kH_k)P_{k-1}(I-K_kH_k)^T + K_kR_kK_k^T$


### Least Squares Estimation Summary

![df]({{site.utl}}/assets/img/posts/DF39.png)

## Linear Kalman

What is the Kalman filter?
* Recursive estimator that solves the "linear-quadratic-estimation" problem
* $\hat x=E(x\mid z_1,z_2,...,z_k)$, where $z_1, z_2,...,z_k$ are noisy measurements, $x$ is a true state (unknown) and $\hat x$ is an estimated state
* $\hat x = min J$
* State estimation error: $\tilde x=x - \hat x$
* State estimation error covariance matrix: $P = [(x - \hat x)(x - \hat x)^T]$
* $J = [(x - \hat x)(x - \hat x)^T] = Tr(P)$ (minimizing estimation covariance, minimizes the cost function)

Main applications are:
* guidance, navigation and control of vehicle (cars, ships, aircraft)
* Robotics and manufacturing
* Any time-domain series analysis

### Types of filter

* Linear Kalman Filter (KF)
* Extended Kalman Filter (EKF)
* Unscented Kalman Filter (UKF)

Linear Kalman Filter:
* Assumes  linear discrete or continuous system dynamics
* Uses linear covariance prediction/update equations
* Does not work with non-linear systems

Extended Kalman Filter:
* Assumes non-linear discrete or continuous system dynamics
* Uses a linear covariance prediction/update equation, **but** the linear relationship is approximated from the non-linear dynamics
* Works well with non-linear systems that are **smooth** and the estimated state is **close to the true state**

Unscented Kalman Filter
* Assumes non-linear discrete or continuous system dynamics
* Uses the non-linear system to calculate the prediction/update equation, making it a better approximation than the EKF (instead of linearizing the system dynamic, it calculates approximation of the uncertainty covariances)
* Works on on-linear systems that are slightly more non-linear that what the EKF can handle, since it is better at approximating the non-linear systems.


### How it works?

![df]({{site.utl}}/assets/img/posts/DF40.png)

* "a priori" means **before**
* "a posteriori" means **after**
* The "+" and "-" superscripts are a compact shorthand for "before" and "after"
* $\hat x_k^-$ means the systems's best guess of the state $k$, calculated by applying the **kinematic equations** to the previous step. This is our prediction based on previous state estimate.
* $\hat x_k^+$ is the **refined** state estimate $k$ once the sensor reading has corrected the prediction
* The filter tracks two things for every state: the **mean** (the best guess of the state) and the **covariance** (how uncertain that guess is)


Discrete time system model:
* **Process (System) model:** $\boxed{x_k=F_{k-1}x_{k-1} + G_{k-1}u_{k-1}+L_{k-1}w_{k-1}}$ (master formula which describes how the system evolves with time from step $k-1$ to step $k$)
	* $x_k$ - target system state we want to calculate
	* $F_{k-1}x_{k-1}$ (transition part) - handle the natural continuation of current state if we do nothing and let physics take the "wheel".
		* **Intuition:** This answers the question: _"If I do nothing and just let physics take the wheel, where will my previous state ($\mathbf{x}_{k-1}$) carry me over time step $\Delta t$?"_ It handles the natural continuation of your current position and velocity.
	* $G_{k-1}u_{k-1}$ (control part) - "What did we actively tell the system to do?"
		* **Intuition:** This answers: _"What did we_ actively tell the system to do?" For example, if you are tracking a vehicle and you know the driver slammed on the brakes, this term injects that known deceleration into the prediction. _(Note: If your tracked object is uncooperative or autonomous and you have no access to its controls—like tracking a wild bird or another drone—this entire term is dropped or set to zero)._
	* $L_{k-1}w_{k-1}$ (noise part) - noise which influence
		* **Intuition:** Real life never follows pure math. A gust of wind hits the drone, a tire slips on gravel, or a gear vibrates. $F$ and $G$ assume a _perfect_ world; $L\mathbf{w}$ represents **everything the math missed**. It forces the filter to admit: _"My physics model is good, but it's not omniscient, so I'll add a little buffer of uncertainty."_
* **Measurement model:** $\boxed{z_k=H_kx_k + M_kv_k}$
* $x_k \sim$ state vector
* $u_k \sim$ control input vector
* $w_k \sim$ process model noise vector
* $v_k \sim$ measurement noise vector
* $F_k \sim$ state transition matrix (describes how the state changes with the input of the state)
* $G_k \sim$ control input matrix (describes how the state changes based on control input)
* $H_k \sim$ measurement matrix describes the linear combination that the measurement is based on the current state)
* $L_k \sim$ process model noise sensitivity matrix
* $M_k \sim$ measurement model noise sensitivity matrix

Assumption about the noise ($w_k$ and $v_k$):
* $w_k \sim N(0, Q_k)$
* $v_k \sim N(0, R_k)$
* $w_k$ and $v_k$ have Gaussian distribution with zero mean and given covariance matrix
* $E(w_kw_j^T) = Q_k \delta_{k-j}$ (not correlated with time)
* $E(v_kv_j^T) = R_k \delta_{k-j}$ (not correlated with time)
* $E(w_kv_j^T) = 0$ (process noise and measurement noise are independent)

**The goal is to estimate the state of the system based on our knowledge of the dynamic and the availability of noise measurements**

* $\hat x^+_0 = E(x_0)$ where $\hat x_0$ is initial state
* $\hat x^-_k = E(x_k \mid z_1, z_2,...,z_{k-1})$ where $\hat x^-_k$ is a priori estimated state
	* **Intuition:** _"Based on where it was, how fast it was going, and how it was accelerating, my physics equations say it should be right here right now."_ It is a prediction _before_ checking the radar or camera.
* $\hat x^+_k = E(x_k \mid z_1, z_2, ..., z_k)$ where $\hat x^+_k$ is a posterior estimated state, $x_k$ is a true state, $z_1,z_2,...,z_k$ are noisy measurements.
	* **Intuition:** _"My physics prediction said it was at point A, but the camera says it's at point B. The Kalman filter compromised and placed it at point C."_ That compromise value is the **a posteriori mean**.

**The Kalman filter also uses the covariance of the estimation error to probabillistically find the updated state estimate**

* $P^-_k = E[(x_k - \hat x^-_k) (x_k - \hat x^-_k)^T]$ where $P^-_k$ is a priori estimated covariance
	* **Intuition:** Because time has passed and objects can wobble, speed up unexpectedly, or drift, **uncertainty grows during the prediction step**. The a priori covariance is usually larger than the previous step's covariance because you are driving blindfolded by physics until the next measurement arrives.
* $P^+_k = E[(x_k - \hat x^+_k) (x_k - \hat x^+_k)^T]$ where $P^+_k$ a posteriori estimated covariance
	* **Intuition:** Getting a fresh measurement gives you new information, which **reduces your uncertainty**. Therefore, the a posteriori covariance is almost always smaller (more confident) than the a priori covariance because the sensor helped pin down where the object actually is.
* $\tilde x=x_k - \hat x_k$

### 2D Tracking Filter

Goal: To estimate the position and velocity of a moving object based on **remote** measurements.

Many applications:
* Object tracking in images
* Military surveillance
* Robotics
* GPS navigation and guidance

Problem:
* Assume an object travels at a constant speed in the 2D X-Y plane {Process Model}
* Assume we get position measurements of the objects location (i.e. Radar) {Sensor Model} 


![df]({{site.utl}}/assets/img/posts/DF41.png)

### 2D Tracking System Model

In a 2D Cartesian coordinate system, an object's position at any time $t$ is defined by a 2D vector: $p(t) = \begin{bmatrix}x(t) \\ y(y)\end{bmatrix}$. As the object moves, its coordinates change over time. The derivatives describe the rates of this changes: $v = \frac {dp} {dt} = \begin{bmatrix}v_x \\ v_y\end{bmatrix}$ (velocity) and $a = \frac {dv} {dt} = \begin{bmatrix}a_x \\ a_y\end{bmatrix}$ (acceleration).

Derive the system process model from the equations of motion (kinematic equations): $\boxed{a = \frac {dv} {dy} = \frac {d^2p} {dt^2}}$

Assuming **constant** acceleration, the relations are found using truncated (as acceleration is constant) Taylor series in discrete time steps term:
* $p = p_0 + t v_0 + \frac 1 2 a t ^2$
* $v = v_0 + a t$
* $p_0$ and $v_0$ are initial position and velocity

In a tracking filter, we rarely deal with continuous time directly; instead, we work in discrete time steps $k$ (i.e. frame-to-frame intervals $\triangle t$). Taylor series expansion is a bridge between continuous physics (kinematic equations) and discrete steps applicable to Kalman filter. The Taylor series expansion says "if we know an object's position and all its derivatives (velocity, acceleration, ...), we can mathematically predict its position at a future time $t + \triangle t$". As we assume that acceleration is constant our Taylor series expansion is truncated. Therefore, any derivatives higher than acceleration are zero or negligible over tiny time step $\triangle t$.


![df]({{site.utl}}/assets/img/posts/DF42.png)

Taylor series for position: $p(t + \Delta t) = p(t) + \frac{dp}{dt}\Delta t + \frac{1}{2}\frac{d^2p}{dt^2}\Delta t^2 + \frac{1}{6}\frac{d^3p}{dt^3}\Delta t^3 + \dots$

* Equation in matrix form:
$$\begin{bmatrix} p \\ v \end{bmatrix} = \begin{bmatrix} 1 & t \\ 0 & 1 \end{bmatrix} \begin{bmatrix} p_0 \\ v_0 \end{bmatrix} + \begin{bmatrix} \frac 1 2 t^2 \\ t \end{bmatrix} a$$

* Convert to discrete time: $t = \triangle t$;
$$\begin{bmatrix} p \\ v \end{bmatrix}_k = \begin{bmatrix} 1 & \triangle t \\ 0 & 1 \end{bmatrix} \begin{bmatrix} p \\ v \end{bmatrix}_{k-1} + \begin{bmatrix} \frac 1 2 \triangle t^2 \\ \triangle t \end{bmatrix} [a]$$

* State vector: 
$$x_k = \begin{bmatrix} p_x \\ p_y \\ v_x \\ v_y \end{bmatrix}$$
* Input vector:
$$u_k = \begin{bmatrix} a_x \\ a_y \end{bmatrix}$$
* Discrete process model: $x_k = Fx_{k_1} + Gu_{k-1}$

$$\begin{bmatrix} p_x \\ p_y \\ v_x \\ v_y \end{bmatrix}_k = \underbrace{\begin{bmatrix} 1 & 0 & \triangle t & 0 \\ 0 & 1 & 0 & \triangle t \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}}_F \begin{bmatrix} p_x \\ p_y \\ v_x \\ v_y \end{bmatrix}_{k-1} + \underbrace{\begin{bmatrix} \frac 1 2 \triangle t^2 & 0 \\ 0 & \frac 1 2 \triangle t^2 \\ \triangle t & 0 \\ 0 & \triangle t \end{bmatrix}}_G \begin{bmatrix} a_x \\ a_y \end{bmatrix}_{k-1}$$

Because we don't have a sensor on board to track acceleration, we don't have input vector. Instead of having a known deterministic input, we assume that the input is a noise vector:

* $w_k = \begin{bmatrix} a_x \\ a_y \end{bmatrix}$
* $a_x \sim N(0,\sigma^2_{a_x})$
* $a_y \sim N(0,\sigma^2_{a_y})$
* $\sigma^2_{a_x}$ and $\sigma^2_{a_y}$ are tuning parameters (they tell how much the acceleration is going to vary in X and Y directions)
* Discrete process model (after we drop the input and assume the only input into the system is going to be from the noise value): $\boxed{x_k = Fx_{k-1} + Lw_{k-1}}$
* $L$ matrix is the same as $G$ matrix
* $a_x$ and $a_y$ are random variables following normal distribution

### Prediction Step

* We being the estimation step with the initial condition $\hat x_0^+$ which is the best estimate of the initial state of the system.
* We want to propagate time forward to find $\hat x_1^- = E(x_1)$,so we can use the sate space model of the system: $\hat x_1^- = F_0\hat x_0^+ + G_0 u_0$

#### Prediction Step (State)

* This is the prediction step or time update step. We use the discrete process model of the system we  want to estimate to predict the estimated state (mean of the Gaussian of the PDF) forward in time.

$$\boxed{\hat x_k^- = F_{k-1}\hat x_{k-1}^+ + G_{k-1} u_{k-1}}$$

#### Prediction Step (Covariance)

* But we also want to propagate the covariance forward in time as well. To begin with $P_0^+$ is the initial covariance for the state estimate of $x_0$
	* Exact knowledge of $x_0$ then $P_0^+ = 0$ (there is no uncertainty)
	* No knowledge of $x_0$ then $P_0^+ \rightarrow \infty$ (there is large uncertainty)
* But we usually know an approximate value (or good assumption) for the value of $x_0$ so we set $P_0^+$ to be:

$$\boxed{P_0^+ = E[(x_0 - \hat x_0^+)(x_0 - \hat x_0^+)^T]}$$

* So given $P_0^+$ we can propagate this uncertainty with time using the linear transformation:

$$\boxed{P_1^- = F_0P_0^+ F_0^T}$$

* But this does not change the amount of the uncertainty in the system, it just shifts it between the states as the system dynamics dictate. The magnitude of $P$ before and after stays the constant. 
* We need to account for the new uncertainty in the state due to the process model noise: $L_{k-1}w_{k-1}$  (part in the discrete process model).
* To account for new uncertainty in the system due to process model noise we add a new covariance term:

$$\boxed{P_1^- = F_0P_0^+ F_0^T + L_0Q_0L_0^T}$$
* It is usually assumed that the process noise is additive, so that $L_k = I$, which means that the equation can be simplified to:

$$\boxed{P_1^- = F_0P_0^+ F_0^T + Q_0}$$

**Predicted (a priori) state estimate:** $\boxed{\hat x_k^- = F_{k-1}\hat x_{k-1}^+ + G_{k-1} u_{k-1}}$ (shifts the mean of the Gaussian probability distribution of the estimated state)
**Predicted (a priori) covariance estimate:** $\boxed{P_k^- = F_{k-1}P_{k-1}^+ F_{k-1}^T + Q_{k-1}}$ (shifts the covariance of the Gaussian probability distribution of the estimated state)

#### 2D Tracking System Model

* Process Model: $x_k = Fx_{k-1} + Lw_{k-1}$, $w_k \sim N(0,Q)$
* Process Noise Model: $P_k^- = FP_{k-1}^+ F^T + LQL^T$, $Q = \begin{bmatrix}\sigma^2_{a_x} & 0 \\ 0 & \sigma^2_{a_x}\end{bmatrix}$

$$\begin{bmatrix} p_x \\ p_y \\ v_x \\ v_y \end{bmatrix}_k = \underbrace{\begin{bmatrix} 1 & 0 & \triangle t & 0 \\ 0 & 1 & 0 & \triangle t \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}}_F \begin{bmatrix} p_x \\ p_y \\ v_x \\ v_y \end{bmatrix}_{k-1} + \underbrace{\begin{bmatrix} \frac 1 2 \triangle t^2 & 0 \\ 0 & \frac 1 2 \triangle t^2 \\ \triangle t & 0 \\ 0 & \triangle t \end{bmatrix}}_L \begin{bmatrix} a_x \\ a_y \end{bmatrix}_{k-1}$$

Looking at $LQL^T$ and expend it out we will obtain not valid Gaussian probability distribution and it is not valid PDF because it is ranked deficient (determinant is zero). Instead of threading acceleration value for $x$ and $y$ separately,we are just going to assume that it's just a constant variance.

Assumptions:
* $\sigma_a = \sigma_{a_x} = \sigma_{a_y}$ (change from 2D Gaussian distribution to 1D)
* $w^k \sim L \cdot N(0,\sigma^2_a)$ (use $L$ matrix as a scaling to the normal distribution in order to make additive noise)
* This gives us covariance $\hat Q$ with valid probability distribution:

$$\hat Q = \sigma^2_a \begin{bmatrix} \frac 1 2 \triangle t^2 & 0 & 0 & 0 \\ 0 & \frac 1 2 \triangle t^2 & 0 & 0 \\ 0 & 0 & \triangle t & 0 \\ 0 & 0 & 0 & \triangle t \end{bmatrix}$$

Now, process noise model become: $\boxed{P_k^- = FP_{k-1}^+ F^T + \hat Q}$ and process model: $\boxed{x_k = Fx_{k-1} + w_{k-1}}$ (with additive noise).

### Update Step

#### Update Step (State)

* Measurement Model: $z_k = H_kx_k + M_kv_k$
* $H_k$ is a measurement model matrix
* $M_k$ is a measurement model noise sensitivity matrix

* We now need to derive how to update the state **prediction** $\hat x_k^-$ (without the current information from the measurement) with the current measurement $z_k$ to form the **updated** state estimate $\hat x_k^+$ (with the current information from the measurement).
* This will be done as a weighted estimate based on the **error** between the **measurement** $z_k$ and the **predicted measurement** calculated from $H_k\hat x_k^-$.
* This **error** is called **innovation** and denoted as $\tilde y_k$

* Innovation: $\boxed{\tilde y_k = z_k - H_k\hat x_k^-}$
* Innovation Covariance: $S_k = H_kP_k^-H_k^T + R_k$
* $z_k\sim$ measurement vector
* $\hat z_k\sim$ predicted measurement vector
* $\tilde y_k \sim$ measurement innovation (error) vector 
* $S_k \sim$ innovation covariance matrix


* The state is updated based the size of the innovation $\hat y_k$, using the **Kalman Gain** matrix $K_k$, which itself is formed as a ration between the innovation uncertainty and the current state uncertainty.

* State Update: $\boxed{\hat x_k^+ = \hat x_k^- + K_k\tilde y_k}$
* Kalman Gain: $\boxed{K_k = P_k^-H_k^TS_k^{-1}}$

#### Update Step (Covariance)

* The whole point of the update process is to reduce the uncertainty in the estimates and improve the accuracy, so the covariance of the estimate must change

* Covariance Update: $\boxed{P_k^+ = (I - K_kH_k)P_K^-}$ ($P_k^+$ must be smaller than $P_k^-$)

#### 2D Tracking Measurement Model

* Measurement Model: $z_k = Hx_k + v_k$
* $v_k \sim N(0, R)$
* $R = \begin{bmatrix}\sigma^2_x & 0 \\ 0 & \sigma^2_y\end{bmatrix}$
* $\sigma_\text{meas} = \sigma_x = \sigma_y$


$$\begin{bmatrix}p_x \\ p_y\end{bmatrix}_k = \underbrace{\begin{bmatrix}1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0\end{bmatrix}}_H \begin{bmatrix}p_x \\ p_y \\ v_x \\ v_y\end{bmatrix}_k$$


### Best Practice

Filter state prediction model check:
* **{Zero-Noise Consistency Check}** Run the simulation without any noise from a **known initial condition** and run the Kalman filter prediction from the **same initial condition** to be sure that the two are consistent.
	* In this noise-free environment, the filter's state transition equation should track the true simulated physics with absolute precision.
	* If the filter's prediction diverges from the simulation event slightly when noise is zero, it proves there is a coding or formulation bug.
* **{Open-Loop Test}** Run without update step, so that errors in the model **aren't hidden** (sometime correction step hides errors in the model).
	* We allow process noise and system dynamics to run with completely disabled measurement correction/update step (ignore incoming sensor data)
	* This test checks that kinematic model doesn't have a hidden bias (e.g. assumes an object accelerates 1% too fast every second), the filter will accumulate an internal drift. High-rate sensor updates constantly force the error back to zero, masking the underlying model drift.
	* Any model bias or improper process noise covariance will immediately cause the filter's error covariance or state estimates to blow up or drift away from the true simulation.


Filter covariance prediction model check:
* The covariance must always represent the approximate level of uncertainty in the filter estimates (model and tuning need to be correct)
	* True State is within the uncertainty bounds around the estimated state
	* If this is not the case then the filter is **inconsistent**. This leads to bad and incorrect estimates.
	* A filter is statistically consistent wen the true error vector $\tilde x = x_k - \hat x_k$ matches the theoretical variance predicted by the covariance matrix $P_k$
	* According to $3\sigma$ rule for a Gaussian distribution, the true state errors should fall within the $\pm 3\sqrt P_{ii}$ bounds of estimates state on any given axis
		* If $P$ is too small (underestimated uncertainty), the filter thinks it has pinpoint accuracy and ignore measurement which leads to divergence and filter failure
		* If $P$ is too large (overestimated uncertainty), the filter is sluggish and fails to react quickly to real changes. 
* Check prediction model correctly transforms uncertainty
	* Uncertainty in the system should not be increasing (if $Q=0$) but it can be transformed with model: $P_k^- = F_{k-1}P_{k-1}^+F_{k-1}^T + Q_{k-1}$, where $Q_{k-1}$ inflates/increases uncertainty and $F_{k-1}P_{k-1}^+F_{k-1}^T$ transforms uncertainty from $k-1$ to $k$
* Check position uncertainty operating as expected
	* Set initial position uncertainty to non-zero value
	* Set initial velocity uncertainty to zero value
	* Run the filter and see that the position uncertainty stays the same
* Check velocity uncertainty operating as expected
	* Set initial position uncertainty to zero value
	* Set initial velocity uncertainty to non-zero value
	* Run the filter and see that the velocity uncertainty stays the same, but the position uncertainty grows linearly with time
		* 1m/s (1 sigma) velocity uncertainty will grow into a 120m (1 sigma) uncertainty after 120 seconds
* Check process model noise (acceleration uncertainty) operating as expected
	* Set initial position uncertainty to zero value
	* Set initial velocity uncertainty to zero value
	* Set non-zero acceleration std (so $Q$ process model noise is non-zero)
	* Run the filter and see that the total level of uncertainty is the system grows with time (velocity uncertainty increase linearly, position uncertainty increase quadratic)


	








 




















