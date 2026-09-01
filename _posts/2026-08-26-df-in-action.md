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

$$0 \le P(A) \le 1$$

where $P(A)=0$: The Event A will never occur and $P(A)=1$ The Event A absolutely certainty will occur.
The probability of not occurring: $P(\neg A) = 1 - P(A)$

Example: Single coin toss, where  the set of possible events is $S=\lbrace H,T \rbrace$,  with $P(H) = 0.5$ and $P(T)=0.5$.

Sum of all probabilities of each event or outcome in set $S$ must equal 1:
$$\sum_{e\in S} P(e) = 1$$
Probability of event A occurring, **assuming each event is equally likely**:
$$P(A) = \frac {\text{Number of ways EventA occurs}} {\text{Total number of outcomes}}$$
**Example:** Double coin toss events has the following space $S=\lbrace HH,HT,TH,TT \rbrace$, where $P(HH)=\frac 1 4$ and in general $P(A)=\frac {N(A)} {N(S)}$

## Mutual Exclusivity

Mutual exclusive events mean that only one event from the given events set $S=\lbrace A,B \rbrace$ can occur at the same time $P(A \cap B) = 0$.

__In general probability of mutual exclusive events:__

$$P(A \cup B) = P(A) + P(B) = 1$$

Example: Single coin toss, where  the set of possible events is $S=\lbrace H,T \rbrace$, events $H$ and $T$ are mutually exclusive.

Events do not have to be mutually exclusive, it is possible to get different events occurring at the same time. In this case events are not mutually exclusive: $P(\text{A and B}) \neq 0$.

Example: Given a deck of card with $S_{number}=\lbrace A,1,2,3,4,5,6,7,8,9,10,J,Q,K\rbrace$ and $S_{suit} = \lbrace Spades,Hearts,Diamonds,Clubs \rbrace$. According to the rules $P(\text{King and Hearts}) \neq 0$ therefore $P(\text{King or Heart}) = P(King) + P(Heart) - P(\text{King and Heart})$.

__In general probability of non-mutually exclusive events:__

$$P(A \cup B) = P(A) + P(B) - P(A \cap B) = 1$$

## Conditional Probability

* Events can be considered **independent** if the likelihood of one event does not affect the likelihood of another occurring (e.g. roll of a dice, toss a coin)
* **Dependent** events are the opposite. When one event occurs it changes the probability of the other events.

Example: 
A deck of card where $P(red) = 26/52 = 0.5$ and $P(black) = 26/52 = 0.5$.
When we pick any card from a full deck of card at random, there is a 50/50 chance that it is a black or red card. Let's say we pull out a three of hearts. And if we put if back and pull out a different card, this time we pull out a black seven of clubs. We still have 50/50 chance of being black or red. In this case events are considered as independent events.
If we don't replace a card, so we leave it pulled out, the probability is changed: $P(red) = \frac {26} {52-1} = 0.51$ and $P(black) = \frac {26-1} {52-1} = 0.49$. In this case events are considered as dependent events.

__In general probability of dependent events:__

$$\boxed{P(A \cap B) = P(A) P(B \mid A) = P(B) P(A \mid B)}$$

Using this formula the probability of drawing a red card followed by a black card without replacing the cards after they are drawn are:
$$P(red \cap black) = P(red)P(black|red)=(26/52)(26/51)=0.255$$

**Conditional probability:**

$$\boxed{P(A\mid B) = \frac {P(A \cap B)} {P(B)}}$$


(probability of A occurring given probability B has occurred is equal probability of events A and B occurring divided by probability of event B occurring).

Where $P(A\mid B)$ is referring as **conditional probability**, $P(\text{A and B})$ - **joint probability**, $P(B)$ - **marginal probability**.

Independent events:

$$P(A \cap B) = P(A)P(B)$$

$$P(A\mid B) = P(A)$$

$$P(B \mid A) = P(B)$$

 
Example:

![df]({{site.utl}}/assets/img/posts/DF13.png)

## Bayes Theorem

* One of the most important concepts used in Bayesian Estimation (e.g. Probabilistic Estimation)
* It allow to calculate the likelihood or bounds on an unknown parameter or event based on prior information related to that event (Bayesian inference)

From conditional probability:

$P(A\mid B) = \frac {P(A \cap B)} {P(B)}$ and $P(B\mid A) = \frac {P(B \cap A)} {P(A)}$ 

Joint probability gives us:

$P(\text{A and B}) = P(\text{B and A})$


**Bayes Theorem:**

$$\boxed{P(A\mid B) = \frac {P(B\mid A) P(A)} {P(B)}}$$

Where $P(A\mid B)$ - posterior probability (the updated probability of event A occurring if given evidence B has occurred), $P(B\mid A)$ - likelihood (the probability of observing event B assuming event A is true), $P(A)$ - prior probability (the initial probability of event A before seeing any new evidence), $P(B)$ - marginal likelihood (the overall probability of observing evidence B across all possible events).

![df]({{site.utl}}/assets/img/posts/DF18.png)

Example:

Prior information: $P(\text{rain}) = 0.6$
(e.g. its rained 18 days out of the 30 days)

Evidence: $P(cloudy) = 0.48$
(e.g. you look outside and say there is a 48% change of it being cloudy this morning)

Likelihood: $P(\text{cloudy|rain}) = 0.68$
(e.g. on the days that it has rained, 68% of the time it was cloudy in the morning)

Posterior probability:
$P(\text{rain|cloudy}) = \frac {P(\text{cloudy|rain}) P(rain)} {P(cloudy)} = 0.85$

## Random Variable

* A random variable is a way to mathematically express stochastic outcomes as real number
* Function mapping that maps a set of experimental outcomes to a set of real numbers

$$X : S \to E$$

Where $X$ is a random variable, $S$ - set of outcomes and $E$ is a set of real number (e.g. $E= \lbrace ...1,2,3,... \rbrace$ for discrete case or $E= \lbrace ...1.01,3.14, ... \rbrace$ for continuous case).

![df]({{site.utl}}/assets/img/posts/DF14.png)


## Probability Density Functions

* Having a random variable $X$, we would to know how likely each outcome is to occur, Is each outcome equally as likely (e.g. toss of a coin) or are some outcomes more likely to occur than others.
* **Probability Density Functions** describes and quantify this.

Measure the relative likelihood or probability that a specific outcome will occur for different random variable types:
* Probability Density Function (PDF) for continuous random variable
* Probability Mass Function (PMF) for discrete random variable

Example:

|                    |                                                                                         |                                               |
| ------------------ | --------------------------------------------------------------------------------------- | --------------------------------------------- |
| Dice Roll          | $P(X=1)=1/6$<br>...<br>$P(X=6)=1/6$<br>$p_X(x_i)=P(X=x_i)$                              | ![df]({{site.utl}}/assets/img/posts/DF15.png) |
| Weighted Dice Roll | $P(X=6)=0.75$<br>$P(X=1)=0.05$<br>...<br>$P(X=5)=0.05$<br>$P(\neg 6) = 1 - P(X=6)=0.25$ | ![df]({{site.utl}}/assets/img/posts/DF16.png) |

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
* **PDF requires an interval:** there are infinitely many real numbers between any two points, the probability of measuring a continuous variable at _exactly_ a single value (e.g., exact height of 175.0000... cm) is effectively zero. Instead, integration the PDF over a range (e.g., between 174.5 cm and 175.5 cm) to find the probability.

## Expectation Operator

The expected value of a random variable is its long-run theoretical average. Let $X$ be a discrete random variable and we will test the value $N$ time. We observe the value $A_1$ occur $n_1$ times and the value $A_2$ occur $n_2$ times and so forth to value $A_m$ occurring $n_m$ times. The expected value $E(X)$ of the random variable $X$ is computed with:

| **Type**       | **Formula**                                         | **What It Means**                                                           |
| -------------- | --------------------------------------------------- | --------------------------------------------------------------------------- |
| **Discrete**   | $E[X] = \sum x \cdot P(X = x)$                      | Sum of each value weighted by its probability.                              |
| **Continuous** | $E[X] = \int_{-\infty}^{\infty} x \cdot f(x) \, dx$ | Integral of each value weighted by its probability density function $f(x)$. |

The expected value or mean of the random variable is usually for simplicity written as:
$$\overline{X} = \overline{x} = E(X)$$
## Distribution Statistical Properties

|                                    Measure                                     |                               Formula                                |                     Graph                     |
| :----------------------------------------------------------------------------: | :------------------------------------------------------------------: | :-------------------------------------------: |
|                  Mean aka `First moment`<br>(expected value)                   |                        $\overline{x} = E(X)$                         | ![df]({{site.utl}}/assets/img/posts/DF19.png) |
|   Variance aka `Second Moment`<br>(the measure of spread away from the mean)   |                       $\sigma_X^2=E[(X-x)^2]$                        | ![df]({{site.utl}}/assets/img/posts/DF20.png) |
| Skew aka `Third Moment`<br>(a measure of how asymmetrical the distribution is) | $skew=E[(X-\overline{x})^3]$<br>$skewness=\frac {skew} {\sigma^3_X}$ | ![df]({{site.utl}}/assets/img/posts/DF21.png) |

Describing a distribution using mean and variance:

$$X \sim (\overline{x},\sigma^2_X)$$

## Uniform Distribution (Continuous)

* Notation: $X \sim U(a,b)$
* Mean: $\overline{x}=\frac 1 2 (a+b)$
* Variance: $\sigma^2_X=\frac 1 {12}(b - a)^2$
* Skew: $skewness = 0$ (distribution is symmetrical)

![df]({{site.utl}}/assets/img/posts/DF22.png)

## Gaussian Distribution (Continuous)

Notation: $X \sim N(\mu,\sigma^2)$

![df]({{site.utl}}/assets/img/posts/DF23.png)


**Important property: Any linear transformation of the Gaussian distribution is another Gaussian distribution.**

## Transformation of Random Variable

Random variables can be transformed through a function. Suppose we have a random variable $X$ and its associated PDF $f_X(x)$, it is possible to apply a mathematical function $y=g(x)$ and find the PDF $f_Y(y)$ of the transformed random variable $Y$.



If $g$ is not strictly monotonic (e.g. $y=0.25$ comes from $x_0=-0.5$ and $x_1=+0.5$):

$$f_Y​(y)= \sum_i f_X​(x_i​) \vert \frac d {dy}g^{-1}(x_i) \vert$$

where $x_i$ ranges overt all solutions of $g(x) = y$.

If $g$ is strictly monotonic:

$$x=g(y)^{-1}=h(y)$$

$$f_Y(y) = f_X(h(y)) |h'(y)|$$


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

So we have new variance scaled by $a$ and new mean $a\overline{x} + b$ but the PDF remains being Gaussian PDF:

$$Y \sim N(a\overline{x}+b, a^2\sigma^2_x)$$

## Multiple Random Variables


Let $X$ be a random variable with a PDF $f_X(x)$ and also let $Y$ be a random variable with a PDF $f_Y(y)$. It is possible to define the PDF of the joint probability (probability of $X$ and $Y$) as $f_{XY}(x,y)$.

$$P(a \le X \le b \space \text{and} \space c \le Y \le d) = \int_c^d \int_a^b f(x,y) \space dx \space dy$$

Marginal density functions:

$$f_X(x) = \int_{-\infty}^\infty f(x,y)\space dy$$

$$f_Y(y) = \int_{-\infty}^\infty f(x,y)\space dx$$

Expected value:

$$E[g(x,y)]=\int_{-\infty}^{\infty} \int_{-\infty}^{\infty} g(x,y) f(x,y) \space dx \space dy  $$

Density function condition ($X$ and $Y$ are independent random variables):

$$f_{XY}(x,y) = f_X(x) f_Y(y)$$

Expected value of multiplication ($X$ and $Y$ are independent random variables):

$$E[XY] = \int \int XYf(x,y)\text{ dx dy} $$

$$E[XY] = \int \int XYf_X(x) f_Y(y)\text{ dx dy} $$

$$E[XY] = \int XYf_X(x) \space dx \int f_Y(y)\space dy $$

$$E[XY] = E(X) E(Y)$$

Expected value of sum ($X$ and $Y$ are independent random variables):

$$z(x,y) = g(x) + h(y)$$

$$E[z(x,y)] = E[g(x) + h(y)]$$

$$E[z(x,y)] = E[g(x)] + E[h(y)]$$


The random variables might not be independent, there might be some correlation between the two.

* Covariance of two dependent variables: $C_{XY} = E[(X - \overline{x})(Y - \overline{y})] = E(XY) - \overline{x}\space\overline{y}$
* Correlation coefficient: $\rho = \frac {C_{XY}} {\sigma_x \sigma_y}$ ($-1$ negatively correlated, $+1$ positively correlated, $0$ - not correlated)


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


The man shifts the center of the distribution, the variance controls the spread in the different axes, while the cross-covariances control the orientation of the distribution:

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

$$f_Y(Y) = \frac 1 {(2\pi)^{n/2} |AC_XA^T|^{1/2}} \exp[-\frac 1 2(Y-\bar Y)^T(AC_XA^T)^{-1}(Y-\bar Y)] $$


Therefore, in case of random variable we end up with Gaussian PDF as well:

$$X\sim N(\bar X, C_X) \Longrightarrow Y=AX+b \Longrightarrow Y \sim N(A \bar X + b,AC_XA^T)$$

If $C_X$ represents the uncertainty covariance, then it can be transformed to another frame using the linear transformation $y=Ax$ where the transformed covariance is given by $C_Y = AC_XA^T$.

