# Vehicule routing Problem

By Alexis DAVID

In this problem we consider trucks with capacities, routes with lengths and demand constraints

# Variables :

Lets consider the following variables : 

- $N :$  Number of delivery points
- $T :$  Number of trucks
- $x_{ij}^t = 1$  if edge $(i,j)$ used by $t$ ; 0 otherwise
- $d_{ij} :$  distance between delivery point $i$ ****and $j$
- $D_i :$ demand in packages of delivery point $i$
- $c_i^t :$ packages carried by truck $t$ for delivery point $i$
- $u_i^t :$ the rank at which delivery point $i$ is visited by truck $t$

$d_{ij}$ , N and $D_j$ are given by the problem and can be considered as constants. $T$  is the number of trucks available by the company but it can be refined to reduce the search space.

Then, decision variables are : $c_j^t$  and $x_{ij}^t$ . It means the company can only choose for each truck  which route it uses and how many packages it carries for each delivery point. 

$u_i^t$  are variables only used to eliminate sub tours

# Objective function :

$$
minimize \sum_{t=1}^T \sum_{i=1}^N \sum_{j=1}^N d_{ij}x_{ij}^t
$$

We want to minimize the total sum of distances traveled by trucks. I suggest to consider each edge of the delivery network and add its distance to the objective each time a truck uses it. 

# Constraints :

This problem looks like the traveling salesman problem but we do not impose that each truck got to every delivery point. So lets relaxe the TSP formulation for each truck and add constraint of capacity and distance specific to this problem. 

$$
\forall t \in \llbracket 1, T 
\rrbracket\ , \newline 
\left\{
\begin{array}{l}
\sum_{i=1}^Nx_{ij}^t \leq 1 \textit{ (arrive at i at most once)} \newline
\sum_{j=1}^Nx_{ij}^t \leq 1 \textit{ (leave j at most once)} 
\end{array}
\right.
\newline
\sum_{j=1}^Nc_{j}^t \leq 50 \textit{ (capacity constraints)}
\newline
\sum_{i=1}^N \sum_{j=1}^N d_{ij} x_{ij}^t \leq 100 \textit{ (distance constraints)}
\newline
\forall i,j \in \llbracket 1; N \rrbracket , i\neq j, \newline
0\leq c_{j}^t, x_{ij}^t ,   \textit{ (Positivity constraints)} 
\newline
x_{ii}^t = 0 

$$

Since we are not including the minimization of the number of packages carried in the objective, we set the total number of packages carried at each delivery point equal to its demand. This avoid the feasible (but not desired) solution where all trucks are full on the way in and come back with residual packages. 

$$
\forall j \in \llbracket 1, N \rrbracket\ , \newline \sum_{t=1}^T c_{j}^t = D_j \textit{ (demand satisfaction constraints)} \newline
\forall j \in \llbracket 1, N \rrbracket\ , \forall i \in \llbracket 1, N \rrbracket\ , i \neq j \newline \forall t \in \llbracket 1, T \rrbracket\ , \newline  
c_j^t \leq 50*x_{ij}^t \textit{ (}x_{ij}^t =0 => c_j^t=0\textit)
\newline 
$$

Now we add a conservation constraint that assures continuity of the route (no teleportation). 

$$
\forall t \in \llbracket 1 ; T\rrbracket, \forall k \in \llbracket 1 ; N\rrbracket, i\neq j, 
\newline
\sum_{i=1}^N x_{ik}^t = \sum_{j=1}^N x_{kj}^t 
$$

For each truck, we add the MTZ formulation for sub-tour elimination. This avoid ending up with unconnected tours for a given truck. 

$$
    u_{i}^t - u_{j}^t + N \cdot x_{i,j}^t \leq N - 1, \quad \forall t \in \{1, \dots, T\}, \, \forall i, j \in \{1, \dots, N-1\}, \, i \neq j 
\newline
u_1^t = 0,  \forall t \in \llbracket 1:T\rrbracket \textit{ (departure of trucks have rank 0)}
$$

We could also add a constraint on the departure. This constraint is not present in standard traveling salesman problem because it provides a complete tour and thus a particular point (departure) can be defined a posteriori. But here, since we will have several tours, we need to guarantee that all the trucks start at position $i=1$ . 

$$
\forall t \in \llbracket 1;T\rrbracket , \sum_{j=2}^N x_{1j}^t = 1
$$

There is no constraint on the number of time a delivery point is being visited because if its demand exceeds 50 packages , it would not be satisfied. 

Hence, trucks can work together to fulfill a demand by adding their contribution for a delivery point. 

# Implementation and scalability:

We could implement this linear integer problem with Julia and packages like JuMP or in python with PuLP for instance. 

Since the number of constraints are $O(N*(N-1)*T)$ it cannot scale easily for a high number of delivery points. However it may be possible for reasonable network size with a reasonable number of trucks. To reduce the complexity, we can start solving with the minimum number of trucks required ( $total\_demand //50 +1$  ) and increase gradualy to meet feasability.