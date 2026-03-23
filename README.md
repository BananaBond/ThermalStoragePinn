# Physics-Informed Neural Networks for Thermal Energy Storage

## Overview

This repository implements Physics-Informed Neural Networks (PINNs) to model heat transfer in a thermal energy storage system. The project includes:

1. Forward solution of coupled PDEs (Task 1)  
2. PDE-constrained inverse problem (Task 2)  

The system represents a 1D thermal storage device where a fluid exchanges heat with a solid medium.

---

## Physical Model

We consider two temperature fields:

- Fluid temperature: \( T_f(x,t) \)  
- Solid temperature: \( T_s(x,t) \)  

The system evolves through charging, idle, discharging, and idle phases.

---

## Governing Equations

### Fluid
\[
\rho_f C_f \frac{\partial T_f}{\partial t} + \rho_f C_f u_f(t)\frac{\partial T_f}{\partial x}
= \lambda_f \frac{\partial^2 T_f}{\partial x^2} - h_v (T_f - T_s)
\]

### Solid
\[
(1-\varepsilon)\rho_s C_s \frac{\partial T_s}{\partial t}
= \lambda_s \frac{\partial^2 T_s}{\partial x^2} + h_v (T_f - T_s)
\]

---

## Initial and Boundary Conditions

### Initial
\[
T_f(x,0) = T_s(x,0) = T_0
\]

### Solid (Neumann)
\[
\frac{\partial T_s}{\partial x}(0,t) = \frac{\partial T_s}{\partial x}(L,t) = 0
\]

### Fluid

- Charging:
\[
T_f(0,t) = T_{hot}, \quad \frac{\partial T_f}{\partial x}(L,t) = 0
\]

- Discharging:
\[
\frac{\partial T_f}{\partial x}(0,t) = 0, \quad T_f(L,t) = T_{cold}
\]

- Idle:
\[
\frac{\partial T_f}{\partial x}(0,t) = \frac{\partial T_f}{\partial x}(L,t) = 0
\]

---

# Task 1: PINNs for PDE Solution

## Objective

Approximate the solution of the coupled system during the charging phase.

---

## Non-Dimensional System

### Fluid
\[
\frac{\partial \bar{T}_f}{\partial t} + U_f \frac{\partial \bar{T}_f}{\partial x}
= \alpha_f \frac{\partial^2 \bar{T}_f}{\partial x^2} - h_f (\bar{T}_f - \bar{T}_s)
\]

### Solid
\[
\frac{\partial \bar{T}_s}{\partial t}
= \alpha_s \frac{\partial^2 \bar{T}_s}{\partial x^2} + h_s (\bar{T}_f - \bar{T}_s)
\]

---

## Parameters

- \( \alpha_f = 0.05 \), \( \alpha_s = 0.08 \)  
- \( h_f = 5 \), \( h_s = 6 \)  
- \( U_f = 1 \)  
- \( T_{hot} = 4 \), \( T_0 = 1 \)  

---

## Boundary Conditions

\[
\bar{T}_f(x,0) = \bar{T}_s(x,0) = T_0
\]

\[
\frac{\partial \bar{T}_s}{\partial x}(0,t) =
\frac{\partial \bar{T}_s}{\partial x}(1,t) =
\frac{\partial \bar{T}_f}{\partial x}(1,t) = 0
\]

\[
\bar{T}_f(0,t) =
\frac{T_{hot} - T_0}{1 + \exp(-200(t-0.25))} + T_0
\]

---

## PINN Formulation

Neural network approximations:

- Single network:  
\[
(x,t) \rightarrow (\bar{T}_f, \bar{T}_s)
\]

- Or two networks:  
\[
(x,t) \rightarrow \bar{T}_f, \quad (x,t) \rightarrow \bar{T}_s
\]

Loss:
\[
\mathcal{L} = \mathcal{L}_{PDE} + \mathcal{L}_{IC} + \mathcal{L}_{BC}
\]

---

# Task 2: PDE-Constrained Inverse Problem

## Objective

Infer \( \bar{T}_s(x,t) \) from known \( \bar{T}_f(x,t) \) using the governing PDE.

---

## Governing Equation

\[
\frac{\partial \bar{T}_f}{\partial t} + U_f(t)\frac{\partial \bar{T}_f}{\partial x}
= \alpha_f \frac{\partial^2 \bar{T}_f}{\partial x^2} - h_f (\bar{T}_f - \bar{T}_s)
\]

---

## Time-Dependent Velocity

Two cycles over \( t \in [0,8] \), each of length 4:

- Charging: \( U_f = 1 \)  
- Idle: \( U_f = 0 \)  
- Discharging: \( U_f = -1 \)  
- Idle: \( U_f = 0 \)  

Each phase lasts 1 time unit.

---

## Parameters

- \( \alpha_f = 0.005 \)  
- \( h_f = 5 \)  
- \( T_{hot} = 4 \), \( T_{cold} = T_0 = 1 \)  

---

