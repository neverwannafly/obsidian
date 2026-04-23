# Kalman Filters — Intuition + Math

---

## 1. What problem does a Kalman Filter solve?

Kalman Filters solve **state estimation**.

You have:
- A system that evolves over time (car, drone, robot, stock price)
- Noisy sensors (GPS, accelerometer, camera, price feed)

You want:
> The best possible estimate of the true state of the system right now.

Example (Uber):
GPS is noisy and delayed, but the car moves smoothly and follows physics.  
Kalman filter combines both.

---

## 2. The hidden state

We do NOT track only what we measure.

We track the full physical state:

x =  
\[ position\_x, position\_y, velocity\_x, velocity\_y \]

This is the **hidden state**.  
We never observe it directly.

We only observe:

z = \[ measured\_x, measured\_y \]  (GPS)

---

## 3. Two models

Kalman filter runs two models at every time step.

### A) Motion model (physics)

Predict how the system moves:

position = position + velocity * dt  
velocity = velocity

Written as a matrix:

xₜ = A xₜ₋₁

A encodes Newton’s laws.

---

### B) Measurement model (sensor)

GPS sees only position:

zₜ = H xₜ + noise

H selects the position from the full state.

---

## 4. Uncertainty is part of the state

Kalman filter also tracks:

P = covariance matrix of uncertainty

This tells:
- how uncertain position is
- how uncertain velocity is
- how errors are correlated

Big P = low confidence  
Small P = high confidence

---

## 5. The Kalman Filter loop

At every time step:

---

### Step 1: Predict

Using physics:

x̂ = A x  
P̂ = A P Aᵀ + Q

Q = how noisy the motion is (bumps, turns, braking)

Now we have:
> Where we think the object is without looking at sensors

---

### Step 2: Compute trust in GPS

K = P̂ Hᵀ ( H P̂ Hᵀ + R )⁻¹

K = Kalman Gain  
R = sensor noise (GPS accuracy)

K decides:
> How much should we trust GPS vs physics?

---

### Step 3: Update using GPS

x = x̂ + K ( z − H x̂ )  
P = ( I − K H ) P̂

This means:
> Move the predicted state toward the GPS measurement, weighted by confidence.

---

## 6. Why this works

If GPS suddenly jumps 30m:
- Physics model says “impossible”
- Kalman gain becomes small
- GPS is ignored

If GPS keeps drifting:
- Uncertainty grows
- GPS is trusted more

The filter automatically balances the two.

---

## 7. What Uber uses this for

Uber tracks:
- position
- velocity
- sometimes acceleration

Even when GPS is bad:
- the filter predicts where the car must be
- map matching snaps it to roads

This produces smooth, realistic movement on the app.

---

## 8. One-line definition

A Kalman Filter is:

A real-time Bayesian estimator that fuses a physical model with noisy measurements to infer the true hidden state.

---

## 9. Why it is powerful

Kalman filters are used in:
- Uber and Google Maps
- Airplanes and missiles
- Self-driving cars
- Robotics
- Finance

Anywhere reality is noisy but structured.
