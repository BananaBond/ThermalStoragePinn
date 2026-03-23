# Physics-Informed Neural Networks for Thermal Energy Storage

## Project Goal

This project uses Physics-Informed Neural Networks (PINNs) to model 1D heat transfer in a thermal energy storage device where a moving fluid exchanges heat with a packed solid medium.

The work is split into two tasks:

1. **Task 1 (forward problem):** solve the coupled PDE system for fluid and solid temperatures.
2. **Task 2 (inverse problem):** recover the solid temperature field from fluid data under PDE constraints.

## Problem Statement

Let $T_f(x,t)$ be the fluid temperature and $T_s(x,t)$ the solid temperature on $x \in [0,L]$.

The coupled model is:

$$
\rho_f C_f \frac{\partial T_f}{\partial t} + \rho_f C_f u_f(t)\frac{\partial T_f}{\partial x}
= \lambda_f \frac{\partial^2 T_f}{\partial x^2} - h_v (T_f - T_s)
$$

$$
(1-\varepsilon)\rho_s C_s \frac{\partial T_s}{\partial t}
= \lambda_s \frac{\partial^2 T_s}{\partial x^2} + h_v (T_f - T_s)
$$

with initial condition:

$$
T_f(x,0) = T_s(x,0) = T_0
$$

and boundary conditions:

Solid (Neumann):

$$
\frac{\partial T_s}{\partial x}(0,t) = \frac{\partial T_s}{\partial x}(L,t) = 0
$$

Fluid during charging:

$$
T_f(0,t) = T_{hot}, \qquad \frac{\partial T_f}{\partial x}(L,t) = 0
$$

Fluid during discharging:

$$
\frac{\partial T_f}{\partial x}(0,t) = 0, \qquad T_f(L,t) = T_{cold}
$$

Fluid during idle:

$$
\frac{\partial T_f}{\partial x}(0,t) = \frac{\partial T_f}{\partial x}(L,t) = 0
$$

---

## Task 1: PINN for the Forward PDE System

### Objective

Approximate the coupled temperature fields $(\bar{T}_f, \bar{T}_s)$ during charging.

### Non-Dimensional Equations

$$
\frac{\partial \bar{T}_f}{\partial t} + U_f \frac{\partial \bar{T}_f}{\partial x}
= \alpha_f \frac{\partial^2 \bar{T}_f}{\partial x^2} - h_f (\bar{T}_f - \bar{T}_s)
$$

$$
\frac{\partial \bar{T}_s}{\partial t}
= \alpha_s \frac{\partial^2 \bar{T}_s}{\partial x^2} + h_s (\bar{T}_f - \bar{T}_s)
$$

### Parameters

- $\alpha_f = 0.05$, $\alpha_s = 0.08$
- $h_f = 5$, $h_s = 6$
- $U_f = 1$
- $T_{hot} = 4$, $T_0 = 1$

### Initial and Boundary Conditions

$$
\bar{T}_f(x,0) = \bar{T}_s(x,0) = T_0
$$

$$
\frac{\partial \bar{T}_s}{\partial x}(0,t) =
\frac{\partial \bar{T}_s}{\partial x}(1,t) =
\frac{\partial \bar{T}_f}{\partial x}(1,t) = 0
$$

$$
\bar{T}_f(0,t) =
\frac{T_{hot} - T_0}{1 + \exp(-200(t-0.25))} + T_0
$$

### PINN Formulation

Network options:

- Single network: $(x,t) \to (\bar{T}_f, \bar{T}_s)$
- Two networks: $(x,t) \to \bar{T}_f$ and $(x,t) \to \bar{T}_s$

Training loss:

$$
\mathcal{L} = \mathcal{L}_{PDE} + \mathcal{L}_{IC} + \mathcal{L}_{BC}
$$

---

## Task 2: PDE-Constrained Inverse Problem

### Objective

Infer $\bar{T}_s(x,t)$ from known/observed $\bar{T}_f(x,t)$ using the physics constraint.

### Governing Equation

$$
\frac{\partial \bar{T}_f}{\partial t} + U_f(t)\frac{\partial \bar{T}_f}{\partial x}
= \alpha_f \frac{\partial^2 \bar{T}_f}{\partial x^2} - h_f (\bar{T}_f - \bar{T}_s)
$$

### Time-Dependent Flow Schedule

Over $t \in [0,8]$, there are two cycles of length 4. Each phase lasts 1 time unit:

- Charging: $U_f = 1$
- Idle: $U_f = 0$
- Discharging: $U_f = -1$
- Idle: $U_f = 0$

### Parameters

- $\alpha_f = 0.005$
- $h_f = 5$
- $T_{hot} = 4$, $T_{cold} = T_0 = 1$

---


