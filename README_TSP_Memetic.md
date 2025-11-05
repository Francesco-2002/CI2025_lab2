# Memetic Algorithm for the Traveling Salesperson Problem (TSP)

This repository contains a Python implementation of a high-performance **Memetic Algorithm (MA)** designed to solve the **Traveling Salesperson Problem (TSP)**.

A Memetic Algorithm is a hybrid approach that combines a traditional **Genetic Algorithm (GA)** with a **local search optimization**.

---

## Key Features

### Hybrid Approach
- It’s a **Memetic Algorithm**, not a simple GA.  
- Every child solution generated is immediately optimized using a **local search heuristic**.

### Asymmetric TSP (ATSP) Support
- Automatically detects if the problem is **symmetric** (standard TSP) or **asymmetric (ATSP)**.
- Chooses the appropriate **local search** and **mutation operators** accordingly.

### High Performance
- Core logic (cost calculation, local search) is accelerated using **Numba** (`@njit`).
- Mutation operators use **fast delta-based cost calculation** ($O(1)$) instead of full $O(N)$ re-calculation.

### Heuristic Initialization
- The initial population combines the **Nearest Neighbour heuristic** and random solutions.

---

## Algorithm Flow

The main `easy_algorithm()` function follows this logic:

1. **Problem Check**  
   Detects if the distance matrix is symmetric to determine the correct mutation/local search logic.

2. **Initialization**  
   Creates a population of `POPULATION_SIZE` individuals using:
   - Nearest Neighbour heuristic (starting from each city)
   - Randomly generated solutions

3. **Evolutionary Loop**
   Runs for `MAX_ITERATIONS` iterations, performing:
   - **Parent Selection:** Tournament selection of two parents  
   - **Offspring Generation:**  
     - 50% chance → crossover (`OX1` or `PMX`)  
     - 50% chance → cloning one parent  
   - **Mutation:** Always applied (equal 33% chance for each operator)  
     - Inverse Mutation (2-opt)  
     - Swap Mutation (2-city exchange)  
     - Insertion Mutation (city relocation)

4. **Local Search (Memetic Step)**
   Every child is immediately optimized until a local optimum is reached:
   - Symmetric TSP → `local_search_systematic` (2-opt search)
   - Asymmetric TSP → `local_search_systematic_insertion` (insertion search)

5. **Survival Selection (Elitism)**
   - Combine parents and offspring  
   - Sort by cost  
   - Keep top N best individuals for the next generation

6. **Restart Mechanism**
   If the best cost doesn’t improve for `MAX_ITERATIONS_WITHOUT_IMPROVEMENT`, the algorithm:
   - Re-initializes the population  
   - Preserves the best-found individual  
   - Repeats this process up to 5 times

---

## Core Components

### Genetic Operators

#### Crossover
- **Order Crossover (OX1):** A segment from the first parent is copied, remaining cities filled from second parent in order.  
- **Partially-Mapped Crossover (PMX):** A segment is swapped and a mapping created to avoid duplicates.

#### Mutation
Optimized to calculate **cost deltas** rather than full tour recalculation.
- **Inverse Mutation (2-opt):** Reverses a segment between two points.  
- **Insertion Mutation:** Moves one city to a different position.  
- **Swap Mutation:** Exchanges two cities.  
- For asymmetric problems, dedicated `*_asymmetric` variants handle direction-specific costs.

---

## Local Search (Hill-Climbing)

This is where the "memetic" improvement occurs — every child undergoes a local search:

- **`_local_search_core` (2-Opt):**  
  Used for symmetric TSPs. Repeatedly applies improving 2-opt moves.

- **`_local_search_insertion_core` (Insertion):**  
  Used for asymmetric TSPs. Repeatedly relocates cities until no improvement is possible.

Both are **Numba-accelerated** for speed.

---

## How to Run

### Dependencies
You need the following Python libraries:
```bash
pip install numpy numba
```

### Problem File
The script expects a NumPy file:
```
lab2/problem_r1_500.npy
```
This file should contain the $N \times N$ distance matrix.

### Execution
Run the algorithm by executing the final cell in the notebook:

```python
easy_algorithm()
```

During execution:
- The best cost found at initialization and every new improvement is printed.  
- The output indicates whether it’s solving a **symmetric** (“Real problem: True”) or **asymmetric** (“Real problem: False”) TSP.

---

## Summary

| Feature | Description |
|----------|-------------|
| Algorithm Type | Memetic (GA + Local Search) |
| Acceleration | Numba (`@njit`) |
| Mutation Cost | Delta-based ($O(1)$) |
| Initialization | Nearest Neighbour + Random |
| Restart | Adaptive, up to 5 cycles |

---
