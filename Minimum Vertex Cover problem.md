Condition:

<img width="342" height="121" alt="image" src="https://github.com/user-attachments/assets/3d8c17ec-f823-4185-b8d5-771ac75044b7" />

Goal: write down this QUBO matrix into QAOA

<img width="645" height="230" alt="image" src="https://github.com/user-attachments/assets/161cf07c-a80c-4a6f-836b-f78debcdb4e8" />

```
import numpy as np
from openqaoa.problems import QUBO

# 1. Define the 5x5 symmetric Q matrix from the image
Q = np.array(
    [
        [-15, 4, 4, 0, 0],
        [4, -15, 0, 4, 0],
        [4, 0, -23, 4, 4],
        [0, 4, 4, -23, 4],
        [0, 0, 4, 4, -15],
    ]
)

n_qubits = Q.shape[0]
terms = []
weights = []

# 2. Parse the Q matrix into OpenQAOA terms and weights
for i in range(n_qubits):
  # Diagonal elements correspond to linear (bias) terms: x_i
  if Q[i, i] != 0:
    terms.append([i])
    weights.append(Q[i, i])

  # Off-diagonal elements correspond to quadratic coupling terms: x_i * x_j
  # Since Q is symmetric (Q_ij = Q_ji), x^T Q x sums both, so we multiply by 2 for i < j
  for j in range(i + 1, n_qubits):
    if Q[i, j] != 0:
      terms.append([i, j])
      weights.append(2 * Q[i, j])

# 3. Create the OpenQAOA QUBO object
qubo_problem = QUBO(n=n_qubits, terms=terms, weights=weights)

# 4. Extract Ising model terms (h and J) exactly like in your original code
ising_hamiltonian = qubo_problem.ising_model

h_new = {
    tuple(i): w
    for i, w in zip(ising_hamiltonian.terms, ising_hamiltonian.weights)
    if len(i) == 1
}
J_new = {
    tuple(i): w
    for i, w in zip(ising_hamiltonian.terms, ising_hamiltonian.weights)
    if len(i) == 2
}
```

Code breakdown
