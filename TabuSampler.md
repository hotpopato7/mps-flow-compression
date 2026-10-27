Here, I use TabuSampler from Dwave samplers: classical optimization from dwave

goal:
<img width="880" height="272" alt="Screenshot 2026-10-27 233458" src="https://github.com/user-attachments/assets/b824cc66-fcfb-499d-84c1-6f82968618f9" />

code:

```
import numpy as np
import dimod
from dwave.samplers import TabuSampler

# 1. Define the symmetric Q matrix from the problem
Q_matrix = np.array([
    [ 1,  -3,  -3,  -3],
    [ -3,  1,  0,   0],
    [ -3,  0,  1,  -3],
    [ -3,  0,  -3,  1],
])

# 2. Convert Maximization to Minimization by negating the Q matrix
# (Since D-Wave samplers find minimums, minimizing -Q maximizes Q)
Q_min = -Q_matrix

# 3. Create the Binary Quadratic Model (BQM)
bqm = dimod.BinaryQuadraticModel(Q_min, "BINARY")

# 4. Instantiate the classical TabuSampler
sampler = TabuSampler()

# 5. Sample the solution
# num_reads: number of independent tabu searches/runs to perform
sampleset = sampler.sample(bqm, num_reads=10)

# 6. Print the best solution found
print("--- Best Solution Found ---")
best_sample = sampleset.first
print(f"Variables (x vector): {best_sample.sample}")

# Since we minimized -Q, negate the energy back to get the maximum y value
max_y = -best_sample.energy
print(f"Maximum y (x^T Q x):    {max_y}")
```

code breakdown

maximize the objective function. However, D-Wave samplers (TabuSampler, ExactSolver, etc.) are strictly designed to minimize objective functions.
solution: negate the Q matrix (-Q_matrix)

or since dimod reads values in upper triangular matrices only, sometimes the optimization may go worng. better write Q in dictionary:

```
import numpy as np
import dimod
from dwave.samplers import TabuSampler

# Negated dictionary to convert Maximization into Minimization for D-Wave
Q_dict = {
    (0, 0): -1,  (1, 1): -1,  (2, 2): -1,  (3, 3): -1,   # Negated diagonal terms
    (0, 1):  3,  (0, 2):  3,  (0, 3):  3,              # Negated quadratic terms
    (2, 3):  3                                       # Negated quadratic terms
}

# Create the BQM from the QUBO dictionary
bqm = dimod.BinaryQuadraticModel.from_qubo(Q_dict)

# 4. Instantiate the classical TabuSampler
sampler = TabuSampler()

# 5. Sample the solution
# num_reads: number of independent tabu searches/runs to perform
sampleset = sampler.sample(bqm, num_reads=10)

# 6. Print the best solution found
print("--- Best Solution Found ---")
best_sample = sampleset.first
print(f"Variables (x vector): {best_sample.sample}")

# Since we minimized -Q, negate the energy back to get the maximum y value
max_y = -best_sample.energy
print(f"Maximum y (x^T Q x):    {max_y}")
```

which yields such results
```
--- Best Solution Found ---
Variables (x vector): {0: np.int8(0), 1: np.int8(1), 2: np.int8(1), 3: np.int8(0)}
Maximum y (x^T Q x):    2.0
```

which matches the solution <img width="893" height="368" alt="Screenshot 2026-10-27 235208" src="https://github.com/user-attachments/assets/0818a4e3-a141-43cc-846c-a037a92f9a01" />

