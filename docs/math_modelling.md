# Model 1: Profit-Maximising Multi-Product Vehicle Routing Problem

---

# Sets

| Symbol | Description |
|---|---|
| $C$ | Set of cafes |
| $0$ | Depot node |
| $N=\{0\}\cup C$ | Set of all nodes |
| $K$ | Set of vans |
| $P$ | Set of products |
| $P^{milk}\subseteq P$ | Set of milk/perishable products |
| $A=\{(i,j):i,j\in N,\ i\neq j\}$ | Set of arcs |

---

# Parameters

## Demand and Product Parameters

| Symbol | Description |
|---|---|
| $d_{ip}$ | Daily demand of product $p$ at cafe $i$ |
| $w_p$ | Weight per unit of product $p$ |
| $r_p$ | Revenue per unit of product $p$ |
| $c_p$ | Cost per unit of product $p$ |
| $m_p=r_p-c_p$ | Margin per unit of product $p$ |

---

## Vehicle Parameters

| Symbol | Description |
|---|---|
| $Q_k$ | Capacity of van $k$ |
| $F_k$ | Fuel cost per kilometre for van $k$ |

---

## Network Parameters

| Symbol | Description |
|---|---|
| $D_{ij}$ | Distance from node $i$ to node $j$ |
| $T_{ij}$ | Travel time from node $i$ to node $j$ |

---

## Service and Perishability Parameters

| Symbol | Description |
|---|---|
| $L$ | Loading time at depot |
| $s$ | Service time per cafe |
| $T^{max}$ | Maximum allowable route duration for vans carrying milk |
| $\beta$ | Penalty coefficient for long routes |

---

# Decision Variables

## Routing Variables

$$
x_{ijk}=
\begin{cases}
1 & \text{if van } k \text{ travels directly from } i \text{ to } j \\
0 & \text{otherwise}
\end{cases}
$$

This binary variable determines the routing structure of each van.

---

## Service Variables

$$
y_{ik}=
\begin{cases}
1 & \text{if cafe } i \text{ is served by van } k \\
0 & \text{otherwise}
\end{cases}
$$

Indicates whether van $k$ services cafe $i$.

$$
z_i=
\begin{cases}
1 & \text{if cafe } i \text{ is served} \\
0 & \text{otherwise}
\end{cases}
$$

Indicates whether cafe $i$ is included in the delivery plan.

---

## Delivery Quantity Variables

$$
q_{ipk}\ge0
$$

Amount of product $p$ delivered by van $k$ to cafe $i$.

---

## Route Duration Variables

$$
R_k\ge0
$$

Total route duration of van $k$.

---

## Milk Indicator Variables

$$
g_k=
\begin{cases}
1 & \text{if van } k \text{ carries milk products} \\
0 & \text{otherwise}
\end{cases}
$$

Used to activate perishability constraints.

---

## MTZ Subtour Elimination Variables

$$
u_{ik}\ge0
$$

Auxiliary variable used to eliminate disconnected subtours.

---

# Objective Function

$$
\max
\quad
\sum_{i\in C}\sum_{p\in P}\sum_{k\in K}
m_pq_{ipk}
-
\sum_{k\in K}\sum_{(i,j)\in A}
F_kD_{ij}x_{ijk}
-
\beta\sum_{k\in K}R_k
$$

## Objective Function Explanation

The objective maximises total daily profit.

The first term:

$$
\sum_{i\in C}\sum_{p\in P}\sum_{k\in K}
m_pq_{ipk}
$$

represents total product margin earned from deliveries.

The second term:

$$
\sum_{k\in K}\sum_{(i,j)\in A}
F_kD_{ij}x_{ijk}
$$

represents transportation cost, calculated using travelled distance and van-specific fuel costs.

The third term:

$$
\beta\sum_{k\in K}R_k
$$

penalises excessively long delivery routes. This approximates perishability effects and operational inefficiency associated with long travel times.

---

# Constraints

---

## 1. Each Cafe Served by At Most One Van

$$
\sum_{k\in K}y_{ik}=z_i,
\quad
\forall i\in C
$$

### Explanation

Each cafe can either:

- be unserved ($z_i=0$), or
- be served by exactly one van ($z_i=1$).

This prevents multiple vans from visiting the same cafe.

---

## 2. Demand Satisfaction

$$
\sum_{k\in K}q_{ipk}
=
d_{ip}z_i,
\quad
\forall i\in C,\ p\in P
$$

### Explanation

If cafe $i$ is served ($z_i=1$), then the full demand for every product must be delivered.

If cafe $i$ is not served ($z_i=0$), then no products are delivered.

This creates an all-or-nothing delivery structure.

---

## 3. Deliveries Only If Van Services Cafe

$$
q_{ipk}
\le
d_{ip}y_{ik},
\quad
\forall i\in C,\ p\in P,\ k\in K
$$

### Explanation

A van can only deliver products to a cafe if it actually visits that cafe.

If $y_{ik}=0$, then:

$$
q_{ipk}=0
$$

forcing zero delivery from van $k$ to cafe $i$.

---

## 4. Vehicle Capacity Constraints

$$
\sum_{i\in C}\sum_{p\in P}
w_pq_{ipk}
\le
Q_k,
\quad
\forall k\in K
$$

### Explanation

The total weight of products carried by van $k$ cannot exceed its capacity.

This includes all milk and coffee bean products loaded onto the van.

---

## 5. Flow Conservation Constraints

### One Outgoing Arc If Cafe Is Served

$$
\sum_{j\in N:j\neq i}
x_{ijk}
=
y_{ik},
\quad
\forall i\in C,\ k\in K
$$

### One Incoming Arc If Cafe Is Served

$$
\sum_{j\in N:j\neq i}
x_{jik}
=
y_{ik},
\quad
\forall i\in C,\ k\in K
$$

### Explanation

If van $k$ services cafe $i$, then:

- it must arrive exactly once, and
- it must depart exactly once.

These constraints ensure route continuity.

---

## 6. Depot Departure and Return Constraints

### At Most One Departure From Depot

$$
\sum_{j\in C}
x_{0jk}
\le1,
\quad
\forall k\in K
$$

### At Most One Return To Depot

$$
\sum_{i\in C}
x_{i0k}
\le1,
\quad
\forall k\in K
$$

### Flow Balance at Depot

$$
\sum_{j\in C}
x_{0jk}
=
\sum_{i\in C}
x_{i0k},
\quad
\forall k\in K
$$

### Explanation

Each van can:

- remain unused, or
- leave the depot once and return once.

The flow balance equation ensures that every used route returns to the depot.

---

# 7. MTZ Subtour Elimination Constraints

$$
u_{ik}
-
u_{jk}
+
|C|x_{ijk}
\le
|C|-1,
\quad
\forall i,j\in C,\ i\neq j,\ k\in K
$$

$$
1
\le
u_{ik}
\le
|C|y_{ik},
\quad
\forall i\in C,\ k\in K
$$

### Explanation

These constraints eliminate disconnected cycles (subtours).

Without them, the model could create invalid routes such as:

$$
\text{Cafe A} \rightarrow \text{Cafe B} \rightarrow \text{Cafe A}
$$

without connecting to the depot.

The MTZ formulation enforces a consistent visit order along each route.

---

# 8. Route Duration Definition

$$
R_k
=
L\sum_{j\in C}x_{0jk}
+
\sum_{(i,j)\in A}
T_{ij}x_{ijk}
+
s\sum_{i\in C}y_{ik},
\quad
\forall k\in K
$$

### Explanation

The total route duration for van $k$ includes:

1. depot loading time,
2. all travel times between nodes,
3. service time spent at cafes.

If the van is unused, then route duration becomes zero.

---

# 9. Milk-Carrying Indicator Constraints

$$
\sum_{i\in C}
\sum_{p\in P^{milk}}
q_{ipk}
\le
Mg_k,
\quad
\forall k\in K
$$

### Explanation

This constraint activates the milk indicator variable.

If any milk product is loaded onto van $k$, then:

$$
g_k=1
$$

Otherwise:

$$
g_k=0
$$

---

# 10. Maximum Route Duration for Milk Deliveries

$$
R_k
\le
T^{max}
+
M(1-g_k),
\quad
\forall k\in K
$$

### Explanation

If van $k$ carries milk products ($g_k=1$), then its total route duration cannot exceed the perishability limit:

$$
R_k\le T^{max}
$$

If the van does not carry milk, the constraint is relaxed using a Big-M formulation.

---

# Model Interpretation

The model jointly determines:

1. Which cafes should be served
2. Which van serves each cafe
3. The route taken by each van
4. The quantity of each product delivered
5. How to maximise total profit while accounting for:
   - product margins
   - transportation costs
   - van capacity
   - route duration penalties
   - perishability constraints