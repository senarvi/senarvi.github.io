---
title: "Kalman filter equations and extended Kalman filter"
math: true
---

## Kalman filter

Kalman filter estimates the state of some quantities by combining two kinds of information: a dynamic model that describes our idea of how the world behaves, and measurements that provide noisy estimates of the state. The principle is to iterate the following steps:

*	Take measurement <span>$z[n]$</span>.
*	Estimate the current state <span>$x[n,n]$</span> based on the measurement and the values predicted by the dynamic model.
*	Predict the next state <span>$x[n+1,n]$</span> based on the dynamic model equations.

### Measurements

Each measurement <span>$z[n]$</span> has some uncertainty. Assume that we know the variance <span>$r$</span> of the measurement error from the specifications of the sensor or through calibration. Usually we have multiple sensors, and it’s convenient to store them in a vector. Then the measurement error is specified as a covariance matrix <span>$R$</span>.

In some applications we want to first transform the measurements into appropriate outputs. For example, we may want to combine the measurements from multiple sensors that measure the same physical quantity. In the standard Kalman filter, each output is a linear combination of the measurements, meaning that the transform can be represented as a matrix <span>$H$</span> that we call the observation matrix.

### State update equation

State update equations describe how to combine the measurements and the estimate that is obtainced using our dynamic model to producee a better state estimate.

The current state could be estimated as the average of all the previous measurements, but we don't want to store all the previous measurements. It's possible to derive the following recursive equation:

<div>$$
x[n,n] = \frac{1}{n} \sum z = x[n,n−1] + \frac{1}{n} (z[n] − x[n,n−1])
$$</div>

This is a weighted sum of the measurement and <span>$x[n,n−1]$</span>, which is our prediction of what the measurement would be. Here the measurement is weighted by <span>$1/n$</span>. We make this value a time-dependent variable and call it the Kalman gain <span>$K[n]$</span>. The state update equation interpolates between the predicted value and the measurement.

<div>$$
x[n,n] = x[n,n−1] + K[n] (z[n] – H x[n,n−1])
$$</div>

Kalman gain will be computed based on uncertainty in our measurements. A low value will smooth the uncertainty. A high value gives more weight to the measurement.

### State extrapolation equation

The state extrapolation equation predicts the next state according to our dynamic model. For example, a one-dimensional, constant velocity movement can be modeled using two variables:

<div>$$
\begin{align}
x[n,n-1] &= x[n,n] + \Delta t x’[n,n] \\
 x’[n,n] &= x’[n,n]
\end{align}
$$</div>

When considering acceleration, we would have three variables. In a three-dimensional model, there would be nine variables in total. For convenience, we want to place them in a nine-dimensional vector, so that we can represent the state extrapolation equation as a multiplication by a state transition matrix <span>$F$</span>.

We’ll make the state extrapolation equation more generic by including control input <span>$u[n]$</span>. This vector may contain other information that we don't predict using the dynamic model, for example steering of the vehicle.

In the standard Kalman filter, our dynamic model is linear, meaning that the transform can be described by matrix multiplication.

<div>$$
x[n+1,n] = F x[n,n] + B u[n]
$$</div>

### Kalman gain

Our predictions have uncertainty too, since our dynamic model is just an approximation. Kalman Filter provides an estimate for the prediction error, <span>$p$</span> (or generally, the covariance matrix <span>$P$</span>), which depends on the time instance and is updated using the covariance equations. It can be initialized to the variance of the initial estimate of <span>$x$</span>.

The current Kalman gain is large (more weight to the measurement), when the prediction error is large, or the measurement error is small.

<div>$$
K[n] = p[n,n-1] / (p[n,n-1] + r)
$$</div>

In matrix form, when including the observation matrix, the equation becomes more complex:

<div>$$
K[n] = P[n,n-1] H^T (H P[n,n-1] H^T + R)^{-1}
$$</div>

### Covariance extrapolation equation

In the same way that we extrapolate the state using the dynamic model, we extrapolate the prediction error using our uncertainty in the dynamic model. This uncertainty can be described, for example, using process noise variance <span>$q$</span>:

<div>$$
p[n+1,n] = p[n,n] + q
$$</div>

Set the process noise variance to a low value, e.g. 0.0001, if the model is accurate. If the model is not accurate (for example, modeling an accelerating motion with a constant velocity), a too low value will cause lag error.
With multiple variables, we describe the process noise as its covariance matrix <span>$Q$</span>.

<div>$$
P[n+1,n] = F P[n,n] F^T + Q
$$</div>

### Covariance update equation

The prediction uncertainty is updated based on the Kalman gain. The larger the Kalman gain, the smaller we’re going to make our next estimate.

<div>$$
p[n,n] = (1 − K[n]) p[n,n−1]
$$</div>

The matrix form considers the observation matrix.

<div>$$
P[n,n] = (I – K[n] H) P[n,n-1]
$$</div>


## Extended Kalman filter

Extended Kalman filter is an extension of this concept for nonlinear dynamic model and output functions.

*	Instead of transforming the state using the matrix <span>$F$</span>, use a possibly non-linear state transition function: <span>$x[n+1,n] = f(x[n,n], u[n])$</span>
*	Instead of transforming the measurements using the matrix <span>$H$</span>, use a possibly non-linear output function: <span>$z = h(x)$</span>

The idea is to linearize the dynamic model and output function around each estimate. This means numerically finding the partial derivatives of the functions at the estimate, and using the Kalman filter equations as if the functions were linear.

*	Replace <span>$H$</span> in the covariance update and Kalman gain equations with the first derivative, or the Jacobian, of <span>$h$</span>.
*	Replace <span>$F$</span> in the covariance extrapolation equation with the first derivative, or the Jacobian, of <span>$f$</span>.
