# Mpox SEIRDC Epidemiological Model

This repository contains a continuous-time compartmental ordinary differential equation (ODE) model developed to analyze and simulate the spread of Mpox (Monkeypox). The implementation was created as part of a Master's course project within the Big Data & Machine Learning School at ITMO University.

The model integrates publicly available OWID surveillance data, isolates the acute outbreak phase, and performs parameter estimation through numerical optimization to recover key epidemiological characteristics.

---

## Overview

Traditional SEIR models typically map active infections to observed cumulative case data, which introduces structural mismatch between model states and surveillance reporting.

This implementation adopts an extended **SEIRDC formulation**, where an auxiliary cumulative state variable (C(t)) is introduced to represent cumulative detected infections. This enables consistent alignment between the continuous-time epidemiological process and discrete cumulative public health observations.

The resulting framework represents a **continuous-time inverse problem**: recover epidemiological parameters that best explain observed cumulative incidence and mortality trajectories.

---

## Mathematical Formulation

### 1. State Variables

The effective population (N) is partitioned into mutually exclusive compartments:

* $(S(t))$: Susceptible individuals
* $(E(t))$: Exposed individuals (infected but not yet infectious)
* $(I(t))$: Infectious individuals
* $(R(t))$: Recovered individuals (assumed immune within the window)
* $(D(t))$: Deceased individuals
* $(C(t))$: Cumulative detected infections (observable proxy)

The auxiliary state (C(t)) accumulates incidence flow and serves as the observable counterpart of reported cumulative case counts.



### 2. System of Ordinary Differential Equations

The disease dynamics are governed by the following continuous-time system:

$$
\frac{dS}{dt} = -\frac{\beta S I}{N}
$$

$$
\frac{dE}{dt} = \frac{\beta S I}{N} - \alpha E
$$

$$
\frac{dI}{dt} = \alpha E - (\gamma + \mu) I
$$

$$
\frac{dR}{dt} = \gamma I
$$

$$
\frac{dD}{dt} = \mu I
$$

$$
\frac{dC}{dt} = \alpha E
$$

where cumulative incidence is defined as the flow of individuals transitioning from exposed to infectious.

---

### 3. Parameter Set

The model parameters

$$
\Theta = {\beta, \alpha, \gamma, \mu, N, E_{0_{frac}}}
$$

define transition rates:

* $(\beta)$: Effective transmission rate
* $(\alpha)$: Incubation progression rate
* $(\gamma)$: Recovery rate
* $(\mu)$: Disease-induced mortality rate
* $(N)$: Effective susceptible population
* $(E_{0_{frac}})$: Initial exposed fraction

---

### 4. Initial Conditions

Given observed cumulative cases (C_{obs}(0)) and deaths (D_{obs}(0)):

$$
C(0) = C_{obs}(0)
$$

$$
D(0) = D_{obs}(0)
$$

$$
I(0) = 1
$$

$$
E(0) = E_{0_{frac}} \cdot N
$$

$$
R(0) = \max(C(0) - I(0) - D(0), 0)
$$

$$
S(0) = \max(N - E(0) - I(0) - R(0) - D(0), 1)
$$

---

### 5. Objective Function

Parameter estimation is performed via minimization of log-space discrepancy between model outputs and observations.

Let (\epsilon = 10^{-6}). The loss function is:

$$
\mathcal{L}(\Theta) =
\frac{1}{T+1}\sum_{t=0}^{T}
\left[\log(C(t,\Theta)+\epsilon) - \log(C_{obs}(t)+\epsilon)\right]^2
+
\frac{1}{T+1}\sum_{t=0}^{T}
\left[\log(D(t,\Theta)+\epsilon) - \log(D_{obs}(t)+\epsilon)\right]^2
$$

Log-space fitting stabilizes optimization under exponential growth dynamics.

---

### 6. Optimization Constraints

The inverse problem

$$
\min_{\Theta}\mathcal{L}(\Theta)
$$

is solved under biologically motivated bounds:

* $(0.01 \leq \beta \leq 3.0)$
* $(0.07 \leq \alpha \leq 0.2)$
* $(0.04 \leq \gamma \leq 0.15)$
* $(10^{-5} \leq \mu \leq 0.05)$
* $(10^5 \leq N \leq 2 \times 10^9)$
* $(0.001 \leq E_{0_{frac}} \leq 0.05)$

A hybrid optimization strategy is used:

* **Differential Evolution** for global exploration
* **L-BFGS-B** for local refinement

---

### 7. Derived Epidemiological Quantities

Post-estimation metrics include:

* Incubation period: $(\frac{1}{\alpha})$
* Infectious period: $(\frac{1}{\gamma})$
* Effective reproduction number:
  $$
  R_0 = \frac{\beta}{\gamma+\mu}
  $$
* Case fatality ratio:
  $$
  \text{CFR} = \frac{\mu}{\gamma+\mu}
  $$

---

## Modeling Assumptions

* Homogeneous mixing within effective population (N)
* Time-invariant parameters within the selected acute-phase window
* Reported cumulative cases and deaths serve as observational proxies
* Demographic turnover is negligible over the modeling horizon

---

## Data Processing and Window Selection

To preserve validity of homogeneous assumptions, the model automatically isolates the primary outbreak wave:

1. Daily incidence is derived from cumulative data
2. A centered 7-day rolling mean removes reporting artifacts
3. The epidemic peak is detected
4. A localized window (30 days before peak, 180 days after) is selected

This restricts inference to a dynamically coherent outbreak phase.

---

## Key Features

* Continuous-time compartmental modeling
* Explicit cumulative observable state
* Log-space inverse calibration
* Automatic acute-phase detection
* Hybrid global–local optimization
* Residual diagnostics and visualization

---

## Requirements

* numpy
* pandas
* matplotlib
* scipy

---

## Usage

Run the file:

```
mpox_model.ipynb
```

Execution will:

1. Download OWID Mpox dataset
2. Detect outbreak window
3. Perform global and local optimization
4. Output parameter estimates and epidemiological metrics
5. Generate model fit and residual plots

---

## Interpretation Note

Estimated parameters represent phenomenological descriptors of the selected outbreak phase rather than uniquely identifiable biological constants. The model should be interpreted as a mechanistic explanatory approximation rather than a predictive surveillance tool.

---
