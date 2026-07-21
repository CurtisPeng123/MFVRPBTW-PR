# Mixed Fleet Vehicle Routing Problem with Backhauls, Time Windows, and Partial Recharging (MFVRPBTW-PR)
This repository provides the benchmark instances, mathematical programming implementation, and reproducible computational experiments accompanying the paper.

## Overview

This repository contains a Gurobi-based implementation for solving the **Mixed Fleet Vehicle Routing Problem with Backhauls and Time Windows (MFVRPBTW)**. This research addresses the complex optimization challenge of routing a heterogeneous fleet of electric and diesel vehicles while accommodating both delivery and pickup operations with time window constraints.

### Problem Description

The MFVRPBTW extends the classical Electric Vehicle Routing Problem (EVRP) by incorporating several real-world complexities:

- **Mixed Fleet Operations**: Heterogeneous fleet consisting of Battery Electric Trucks (BET) and Diesel Trucks (DT)
- **Backhaul Logistics**: Combined delivery and pickup operations with configurable backhaul ratios
- **Time Windows**: Strict service time constraints at customer locations
- **Electric Vehicle Constraints**: Battery capacity limitations and charging station requirements
- **Multi-objective Optimization**: Minimizing total travel cost while satisfying all operational constraints

## Features

### Core Capabilities

- **Mathematical Programming**: Formulated as a Mixed-Integer Linear Program (MILP) using Gurobi optimizer
- **Fleet Heterogeneity**: Supports multiple vehicle types with different characteristics
  - Electric vehicles with battery capacity constraints
  - Diesel vehicles with unlimited range
- **Backhaul Management**: Configurable pickup-to-delivery ratios (50%, 66%, 80%)
- **Time Window Enforcement**: Strict adherence to customer service time windows
- **Charging Infrastructure**: Electric vehicle charging station integration
- **Route Optimization**: Simultaneous route planning for mixed fleet operations


## Usage

### Basic Usage

The main solver function is implemented in `MFVRPBTW_Gurobi_v1.ipynb`:

```python
from mfsolver import mfsolver

# Solve an instance
filename = 'c101_21'
num_customers = 5
backhaul_ratio = 3  # 3 = 66% pickup rate (2=50%, 4=80%)

mfsolver(filename, num_customers, backhaul_ratio)
```

### Parameters

- **`file`**: Name of the instance file (without extension)
- **`max_c`**: Maximum number of customers to consider from the instance
- **`num_b_cus`**: Backhaul ratio denominator
  - `2`: 50% pickup customers
  - `3`: 66% pickup customers  
  - `4`: 80% pickup customers

### Example Execution

```python
# Solve a small instance with 5 customers and 50% backhaul rate
mfsolver('c101_21', max_c=5, num_b_cus=2)

# Solve a larger instance with 10 customers and 66% backhaul rate
mfsolver('r102_21', max_c=10, num_b_cus=3)
```

### Running the Jupyter Notebook

1. Start Jupyter Notebook:
   ```bash
   jupyter notebook MFVRPBTW_Gurobi_v1.ipynb
   ```

2. Execute cells sequentially or modify parameters as needed

3. View results including:
   - Optimal objective value
   - CPU time
   - Vehicle routes
   - Customer assignments

## Instance Format

### File Structure

Instance files are located in `E-VRPTW Instances/C100/` and follow a specific format:

```
StringID   Type       x          y          demand     ReadyTime  DueDate    ServiceTime
D0         d          40.0       50.0       0.0        0.0        1236.0     0.0
S0         f          40.0       50.0       0.0        0.0        1236.0     0.0
C1         c          45.0       68.0       10.0       78.0       140.0      90.0
```

### Location Types

- **`d`**: Depot - Starting and ending point for all vehicles
- **`f`**: Charging station - Recharging locations for electric vehicles
- **`c`**: Customer - Delivery or pickup locations

### Location Attributes

- **`x, y`**: Euclidean coordinates
- **`demand`**: Cargo requirement (positive for delivery, negative for pickup)
- **`ReadyTime`**: Earliest service time (time window start)
- **`DueDate`**: Latest service time (time window end)
- **`ServiceTime`**: Time required for loading/unloading operations

### Vehicle Parameters

Located at the end of each instance file:

```
Q Vehicle fuel tank capacity /79.69/
C Vehicle load capacity /200.0/
r fuel consumption rate /1.0/
g inverse refueling rate /3.39/
v average Velocity /1.0/
```

- **`Q`**: Battery capacity for electric vehicles
- **`C`**: Maximum cargo capacity
- **`r`**: Energy consumption rate per unit distance
- **`g`**: Time required to recharge one unit of energy
- **`v`**: Average vehicle velocity

## Mathematical Formulation

### Decision Variables

- **$x_{vij}$**: Binary variable indicating if vehicle $v$ travels from node $i$ to node $j$
- **$t_i$**: Arrival time at node $i$
- **$u_i$**: Remaining cargo capacity at node $i$
- **$b_{vi}$**: Battery level of electric vehicle $v$ at node $i$
- **$z_j$**: Binary variable indicating if charging station $j$ is visited
- **$y_v$**: Binary variable indicating if vehicle $v$ is used

### Objective Function

Minimize total travel distance + fleet utilization cost:

$$
\min \sum_{v \in V} \sum_{i \in N_0} \sum_{j \in N} d_{ij} x_{vij} + \sum_{v \in V} f_v \cdot y_v
$$

Where:
- $d_{ij}$: Travel distance from node $i$ to node $j$
- $x_{vij}$: Binary variable (1 if vehicle $v$ travels from $i$ to $j$, 0 otherwise)
- $f_v$: Fixed utilization cost for vehicle $v$ (may differ by vehicle type)
- $y_v$: Binary variable (1 if vehicle $v$ is used, 0 otherwise)

### Key Constraints

1. **Customer Service**: Each customer must be visited exactly once
2. **Flow Conservation**: Vehicles must enter and leave each node
3. **Time Windows**: Service must occur within specified time windows
4. **Cargo Capacity**: Vehicle load cannot exceed capacity
5. **Battery Management**: Electric vehicles must maintain sufficient battery levels
6. **Charging Station Access**: Electric vehicles can recharge at designated stations
7. **Backhaul Restrictions**: Pickup operations must follow delivery operations
8. **Fleet Constraints**: Each vehicle can perform at most one route

## Results and Output

### Solution Information

The solver provides:

- **Objective Value**: Total travel distance of optimal solution
- **CPU Time**: Computational time for solving
- **Solution Status**: Gurobi optimization status (optimal, feasible, etc.)
- **Vehicle Routes**: Detailed route for each vehicle in the fleet
- **Customer Assignments**: Which vehicle serves which customers

### Route Output Format

Routes are displayed as ordered sequences of node IDs:

```python
{
    0: [0, 1, 5, 3, 0],  # Electric vehicle route
    1: [0, 2, 4, 6, 0],  # Electric vehicle route
    2: [0, 7, 8, 0],     # Diesel vehicle route
}
```

Where:
- `0` represents the depot
- Values (1,2,3, ...) belongs to charging stations or backhaul/linehaul customer nodes
- Each list represents a complete route starting and ending at the depot


### Instance Selection

Available instances in `E-VRPTW Instances/C100/`:

- **c-series**: Clustered customer distributions
- **r-series**: Random customer distributions
- **rc-series**: Randomly clustered customer distributions

Each series includes various configurations with different numbers of customers and backhaul ratios.

## Performance Considerations

### Computational Complexity

The MFVRPBTW is NP-hard, and solution time scales exponentially with:
- Number of customers
- Number of vehicles
- Complexity of time windows
- Charging station locations

### Optimization Tips

1. **Start Small**: Begin with instances having fewer customers (5-10)
2. **Adjust Time Limits**: Set appropriate time limits based on instance size
3. **Fleet Sizing**: Use minimal number of vehicles necessary
4. **Instance Selection**: Choose simpler instances for initial testing

<!-- ## Acknowledgments

- Gurobi Optimization for providing the mathematical programming solver
- E-VRPTW instance library for benchmark datasets
- Research funding agencies and collaborators -->

## References
Schneider, M., Stenger, A., & Goeke, D. (2014). The electric vehicle-routing problem with time windows and recharging stations. Transportation science, 48(4), 500-520.

---
**Note**: The implementation includes several solver-oriented reformulations and auxiliary constraints for computational efficiency while remaining mathematically equivalent to the formulation presented in the paper.
