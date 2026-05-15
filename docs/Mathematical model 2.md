# Model 2: Selective Multi-Product VRPTW for Wominjeka Coffee Co

This document shows **`gurobi model 2.ipynb`** as a mathematical model. I keep the main notation from `math_modelling.md`,

$$
x_{ijk},\ y_{ik},\ z_i,\ q_{ipk},\ R_k,\ g_k,
$$

This model aimed to upgrade the previous selective multi-product CVRP into a **selective multi-product vehicle routing problem with time windows (VRPTW)** by adding departure times, cafe arrival times, soft lateness variables, inventory constraints, vehicle-product compatibility, and time-propagation constraints.

---

# 1. Sets

| Symbol | Description |
|---|---|
| $C$ | Set of cafes |
| $0$ | Depot node |
| $N=\{0\}\cup C$ | Set of all nodes |
| $K$ | Set of vans |
| $P$ | Set of products |
| $P^{milk}\subseteq P$ | Set of milk products |
| $A=\{(i,j):i,j\in N,\ i\neq j\}$ | Set of feasible directed arcs |

---

# 2. Parameters

## 2.1 Product and Demand Parameters

| Symbol | Description |
|---|---|
| $d_{ip}$ | Daily demand of product $p$ at cafe $i$ |
| $w_p$ | Weight per unit of product $p$ |
| $r_p$ | Revenue per unit of product $p$ |
| $c_p$ | Cost per unit of product $p$ |
| $m_p=r_p-c_p$ | Margin per unit of product $p$ |
| $I_p$ | Available inventory of product $p$ |

## 2.2 Vehicle Parameters

| Symbol | Description |
|---|---|
| $Q_k$ | Capacity of van $k$ |
| $F_k$ | Fuel or operating cost per kilometre for van $k$ |
| $\alpha_{kp}$ | 1 if van $k$ is compatible with product $p$, and 0 otherwise |
| $\phi$ | Fixed cost of activating one van |

## 2.3 Network and Time Parameters

| Symbol | Description |
|---|---|
| $D_{ij}$ | Distance from node $i$ to node $j$ |
| $T_{ij}$ | Travel time from node $i$ to node $j$ |
| $L$ | Loading time at the depot |
| $s$ | Service/unloading time per cafe |
| $a_i$ | Start of the receiving time window of cafe $i$ |
| $b_i$ | End of the receiving time window of cafe $i$ |
| $H^0$ | Earliest allowed depot departure time |
| $H^1$ | Latest allowed return time / end of working day |
| $T^{max}$ | Maximum allowable route duration for vans carrying milk |
| $M^T$ | A sufficiently large time-based Big-M constant |
| $M^{milk}$ | A sufficiently large milk-volume Big-M constant |
| $\beta$ | Penalty coefficient for long route duration |
| $\rho$ | Penalty coefficient for late delivery |
| $\theta$ | Minimum service fraction if partial delivery is allowed |
| $n^{min}$ | Minimum number of cafes that must be selected if cafes are optional |

---

# 3. Decision Variables

## 3.1 Routing Variables

$$
x_{ijk}=\begin{cases}
1, & \text{if van }k\text{ travels directly from node }i\text{ to node }j,\\
0, & \text{otherwise,}
\end{cases}
\qquad \forall (i,j)\in A,\ k\in K.
$$

## 3.2 Service and Selection Variables

$$
y_{ik}=\begin{cases}
1, & \text{if cafe }i\text{ is served by van }k,\\
0, & \text{otherwise,}
\end{cases}
\qquad \forall i\in C,\ k\in K.
$$

$$
z_i=\begin{cases}
1, & \text{if cafe }i\text{ is selected and served,}\\
0, & \text{otherwise,}
\end{cases}
\qquad \forall i\in C.
$$

## 3.3 Delivery Quantity Variables

$$
q_{ipk}\ge 0,
\qquad \forall i\in C,\ p\in P,\ k\in K,
$$

where $q_{ipk}$ is the quantity of product $p$ delivered by van $k$ to cafe $i$.

## 3.4 Route Duration and Milk Indicator Variables

$$
R_k\ge0,
\qquad \forall k\in K,
$$

where $R_k$ is the total route duration of van $k$.

$$
g_k=\begin{cases}
1, & \text{if van }k\text{ carries any milk product,}\\
0, & \text{otherwise,}
\end{cases}
\qquad \forall k\in K.
$$

## 3.5 Time-Window Variables

$$
\delta_k\ge0,\qquad \forall k\in K,
$$

where $\delta_k$ is the departure time of van $k$ from the depot.

$$
\tau_{ik}\ge0,\qquad \forall i\in C,\ k\in K,
$$

where $\tau_{ik}$ is the arrival time of van $k$ at cafe $i$.

$$
\ell_{ik}\ge0,\qquad \forall i\in C,\ k\in K,
$$

where $\ell_{ik}$ is the lateness of van $k$ at cafe $i$ under soft time windows.

---

# 4. Objective Function

$$
\begin{aligned}
\max Z ={}& \sum_{i\in C}\sum_{p\in P}\sum_{k\in K} m_p q_{ipk} \\
&- \sum_{k\in K}\sum_{(i,j)\in A} F_k D_{ij} x_{ijk} \\
&- \phi \sum_{k\in K}\sum_{j\in C} x_{0jk} \\
&- \beta \sum_{k\in K} R_k \\
&- \rho \sum_{i\in C}\sum_{k\in K} \ell_{ik}.
\end{aligned}
$$

The objective maximises WCC's daily net profit.  The first term is the product margin.  The second term is the van-specific travel cost.  The third term charges a fixed cost when a van leaves the depot.  The fourth term penalises long routes.  The final term penalises late delivery under soft time windows.

---

# 5. Constraints

## 5.1 Cafe Selection and Van Assignment

$$
\sum_{k\in K}y_{ik}=z_i,
\qquad \forall i\in C.
$$

Each cafe is either unserved, or served by exactly one van.

---

## 5.2 Demand Satisfaction

Default full-delivery version:

$$
\sum_{k\in K}q_{ipk}=d_{ip}z_i,
\qquad \forall i\in C,\ p\in P.
$$

If cafe $i$ is selected, all of its demand is delivered. If it is not selected, no delivery is made.

Optional partial-delivery version:

$$
\sum_{k\in K}q_{ipk}\le d_{ip}z_i,
\qquad \forall i\in C,\ p\in P,
$$

$$
\sum_{k\in K}q_{ipk}\ge \theta d_{ip}z_i,
\qquad \forall i\in C,\ p\in P.
$$

The partial-delivery version allows a selected cafe to receive between $\theta$ and 100 percent of demand.

---

## 5.3 Delivery Linking and Vehicle-Product Compatibility

$$
q_{ipk}\le d_{ip}\alpha_{kp}y_{ik},
\qquad \forall i\in C,\ p\in P,\ k\in K.
$$

A van can deliver product $p$ to cafe $i$ only if it serves cafe $i$ and is compatible with product $p$.

---

## 5.4 Vehicle Capacity

$$
\sum_{i\in C}\sum_{p\in P}w_pq_{ipk}\le Q_k,
\qquad \forall k\in K.
$$

The total loaded weight of all products on van $k$ cannot exceed its capacity.

---

## 5.5 Inventory Constraint

$$
\sum_{i\in C}\sum_{k\in K}q_{ipk}\le I_p,
\qquad \forall p\in P.
$$

The total delivered quantity of product $p$ cannot exceed WCC's available inventory.

---

## 5.6 Flow Conservation

$$
\sum_{j\in N:j\neq i}x_{ijk}=y_{ik},
\qquad \forall i\in C,\ k\in K,
$$

$$
\sum_{j\in N:j\neq i}x_{jik}=y_{ik},
\qquad \forall i\in C,\ k\in K.
$$

If van $k$ serves cafe $i$, it enters exactly once and leaves exactly once.

---

## 5.7 Depot Departure and Return

$$
\sum_{j\in C}x_{0jk}\le1,
\qquad \forall k\in K,
$$

$$
\sum_{i\in C}x_{i0k}\le1,
\qquad \forall k\in K,
$$

$$
\sum_{j\in C}x_{0jk}=\sum_{i\in C}x_{i0k},
\qquad \forall k\in K.
$$

Each van may remain unused, or depart from the depot once and return once.

---

## 5.8 Route Duration Definition

$$
R_k=
L\sum_{j\in C}x_{0jk}
+
\sum_{(i,j)\in A}T_{ij}x_{ijk}
+
s\sum_{i\in C}y_{ik},
\qquad \forall k\in K.
$$

Route duration includes depot loading time, travel time, and cafe service time.

---

## 5.9 Milk-Carrying Indicator

$$
\sum_{i\in C}\sum_{p\in P^{milk}}q_{ipk}\le M^{milk}g_k,
\qquad \forall k\in K,
$$

$$
g_k\le \sum_{j\in C}x_{0jk},
\qquad \forall k\in K.
$$

The first constraint activates $g_k$ when any milk product is loaded.  The second keeps $g_k=0$ for unused vans.

---

## 5.10 Milk Freshness / Maximum Route Duration

$$
R_k\le T^{max}+M^T(1-g_k),
\qquad \forall k\in K.
$$

If van $k$ carries milk, then its route duration cannot exceed the freshness limit $T^{max}$.

---

## 5.11 Departure Time Bounds

$$
\delta_k\ge H^0\sum_{j\in C}x_{0jk},
\qquad \forall k\in K,
$$

$$
\delta_k\le H^1\sum_{j\in C}x_{0jk},
\qquad \forall k\in K.
$$

If van $k$ is used, it departs between $H^0$ and $H^1$. If it is unused, $\delta_k=0$.

---

## 5.12 Cafe Receiving Time Windows

$$
\tau_{ik}\ge a_iy_{ik},
\qquad \forall i\in C,\ k\in K,
$$

$$
\tau_{ik}\le b_i+\ell_{ik}+M^T(1-y_{ik}),
\qquad \forall i\in C,\ k\in K,
$$

$$
\tau_{ik}\le M^T y_{ik},
\qquad \forall i\in C,\ k\in K.
$$

If cafe $i$ is served by van $k$, its arrival time must be no earlier than $a_i$ and no later than $b_i$ plus lateness.  The third constraint keeps unused arrival times equal to zero.

For hard time windows, set

$$
\ell_{ik}=0,
\qquad \forall i\in C,\ k\in K.
$$

---

## 5.13 Time Propagation from the Depot

$$
\tau_{jk}\ge \delta_k+L+T_{0j}-M^T(1-x_{0jk}),
\qquad \forall j\in C,\ k\in K.
$$

If van $k$ travels directly from the depot to cafe $j$, then its arrival time must include departure time, depot loading time, and travel time.

---

## 5.14 Time Propagation Between Cafes

$$
\tau_{jk}\ge \tau_{ik}+s+T_{ij}-M^T(1-x_{ijk}),
\qquad \forall i,j\in C,\ i\neq j,\ k\in K.
$$

If van $k$ travels from cafe $i$ to cafe $j$, then it can only arrive at $j$ after serving cafe $i$ and travelling from $i$ to $j$.

---

## 5.15 Return-Time Consistency and Workday Limit

$$
\tau_{ik}+s+T_{i0}
\le
\delta_k+R_k+M^T(1-x_{i0k}),
\qquad \forall i\in C,\ k\in K,
$$

$$
\delta_k+R_k
\le
H^1+M^T\left(1-\sum_{j\in C}x_{0jk}\right),
\qquad \forall k\in K.
$$

The first constraint links the return-to-depot arc to the route duration.  The second ensures that every used van returns before the end of the working day.

---

## 5.16 Customer Selection Control

If all cafes must be served for comparison with Model 2, impose

$$
z_i=1,
\qquad \forall i\in C.
$$

Otherwise, if customer selection is enabled, impose

$$
\sum_{i\in C}z_i\ge n^{min}.
$$

The second version prevents the optimiser from selecting no cafe when costs are high.

---

## 5.17 Variable Domains

$$
x_{ijk}\in\{0,1\},\quad \forall (i,j)\in A,\ k\in K,
$$

$$
y_{ik}\in\{0,1\},\quad \forall i\in C,\ k\in K,
$$

$$
z_i\in\{0,1\},\quad \forall i\in C,
$$

$$
g_k\in\{0,1\},\quad \forall k\in K,
$$

$$
q_{ipk},R_k,\delta_k,\tau_{ik},\ell_{ik}\ge0.
$$

---

# 6. Subtour Elimination Note

This Model 2 does **not** use the MTZ variable $u_{ik}$ as the main subtour-elimination method.  Instead, cafe-only subtours are implicitly ruled out by the time-propagation constraint.

For example, suppose a disconnected cycle exists:

$$
A\to B\to C\to A.
$$

Then the time-propagation constraints imply

$$
\tau_B\ge\tau_A+s+T_{AB},
$$

$$
\tau_C\ge\tau_B+s+T_{BC},
$$

$$
\tau_A\ge\tau_C+s+T_{CA}.
$$

Adding them gives

$$
0\ge 3s+T_{AB}+T_{BC}+T_{CA},
$$

which is impossible when service and travel times are positive. Therefore, disconnected cafe-only cycles cannot be feasible.

---

# 7. Model Interpretation

Model 2 jointly decides:

1. which cafes should be selected;
2. which van serves each selected cafe;
3. how much of each product is delivered;
4. whether a van carries milk;
5. each van's departure time;
6. each cafe's arrival time;
7. the complete route of each van.

This model evolves the earlier multi-product CVRP into a more comprehensive VRPTW application. To better reflect the WCC delivery environment, it incorporates time-sensitive variables and profitability factors—specifically cafe receiving windows, milk freshness, and product margins.
