[![hackmd-github-sync-badge](https://hackmd.io/S-zvJu8gQ36HTEUXZuONFA/badge)](https://hackmd.io/S-zvJu8gQ36HTEUXZuONFA)

# Simulating molecules(LiH) using VQE

###### Qiskit Summer School Final Project: Designing your own implementation of a variational quantum eigensolver (VQE) algorithm that simulates the ground state energy of the Lithium Hydride (LiH) molecule.

## INTRODUCTION
Fundamentally light was believed to be a wave. But Albert Einsten found out that light consisted of photons called quanta which have energy determines by its frequency. Photons of vissible light can be absorbed by an electron thereby causing the electron to moce from a low energy orbital to a high energy orbital.
Determining these properties is difficult for a classical computer since the electrons may be highly enetangled. To accurately model this we require high computational power. This is where quantum cmputers come in play. They are more efficient at handling entanglement compared to classical computers. One of the properties that is essential is the ground state energy of a molecule. Finding the ground state energy of these molecules becomes harder as the size of the molecule increases. This is why upto now the largest simulated molecule is Beryllium Hydride. Molecular simulations problems grow exponentially as the size of molecules increase. Some of the applications of these problems are in drug discoveries.
> Richard Feyman suggested Quantum Simulator aims to harness controlled Quantum evolution in order to simulate other quantum systems

## THEORY
The variational principle explains how the energy of any trial wave function is greater than or equal to the exact ground state energy of the systme.

We find the fermionic operator first and map the fermionic hamiltonians to qubit hamiltonians. For this transformation we use
1. Jordan Wigner transformation leads to N-local hamiltonians
2. Bravyi-Kotoev transformation leads to log(N) local hamiltonians
