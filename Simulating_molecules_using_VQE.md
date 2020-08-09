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
3. Partity transformation

It is important to find the minimum eigen value of a matrix, in Chemistry the minimum eigen value of a Hermitian matrix is the ground state energy. 
By applying a parameterized circuit represented by U(0) to some state |U> the outputs is 
The estimate is optimized by a classical optimizer changing the parameters thus minimizinf the expectation value of 
This is the variational principle.
> Hermitian matrix is equal to its own conjugate transpose
> By Spectral theorem the expectation value of the Hernitian matrix must be real and any expectation value has the property 

The expectation value corresponding to the Hermitian matrix to an arbitary state is geiven by

Expectation value of any wave function will be atleast the minimum eigen value associated with the wave function.
The ground state of the Hamiltonian system is the smallest eigen value associated with the Hermitian matrix.

## VQE
The ideal approach in varrying your ansatz is to use a parametrized circuit with a fixed form.
The circuit is represented by a linear transform. The circuit is applied to an initial state(Vacuum state or Hartree Fock state) and generates the output
Various variational circuits exist, Ry, RyRz and UCCSD. As the circuit depth increases so does the noise increase. In this case we use heuristic variational form RYRZ rather than the UCCSD as it is shallow and uses few gates.

## Parameter optimisation
One problem arises, there is noise so the energy calculations may not be true. We try and overcome this by gradient descent. But again gradient descent has it problems.
We use Simulatneous Perteburation Stochastic Approximation(SPSA) as the ideal optimuizer. It works by perteburing all the parameters in random. 
Under no oise Sequential Least Squares Programming(SLSQP) and Constarint Optimisation by Linear Approximation(COBYLA) are prefered.

## PROCESS
1. We map the molecular hamiltonians into qubit hamiltonians. We copy the electon orbitals interactions to qubits.
2. We create an ansatz.
If we want to vary the ansatz then we need a parametrized circuit with a fixed form.There are various variational forms that exist, Ry, RyRz and UCCSD which utilize domain specific knowledge

3. Parameter optimization



## IMPLEMENTATION

Hopefully you have gained a firm understadng on how VQE works and you can now implement one on your own. Leave some claps and follow me for more content

<img src="https://i.upmath.me/svg/%5Clambda%20min" alt="\lambda min" />

A

<img src="https://i.upmath.me/svg/%5Cmid%20%5Cpsi%20min%20%5Crangle%20" alt="\mid \psi min \rangle " />

b

<img src="https://i.upmath.me/svg/ax%5E2%2Bbx%2Bc%3D0" alt="ax^2+bx+c=0" />

c

<img src="https://i.upmath.me/svg/%20%5Cpsi_i%20%5Crangle%20%3D%20%5Clambda_i%20%7C%5Cpsi_i%5Crangle" alt=" \psi_i \rangle = \lambda_i |\psi_i\rangle" />

d

<img src="https://i.upmath.me/svg/%20H%20%3D%20H%5E%7B%5Cdagger%7D%20" alt=" H = H^{\dagger} " />


e

<img src="https://i.upmath.me/svg/%20%5Clambda_i%20%3D%20%5Clambda_i%5E*" alt=" \lambda_i = \lambda_i^*" />

f
<img src="https://i.upmath.me/svg/%20H%20%3D%20%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%20%5Clambda_i%20%7C%5Cpsi_i%5Crangle%20%5Clangle%20%5Cpsi_i%20%7C" alt=" H = \sum_{i = 1}^{N} \lambda_i |\psi_i\rangle \langle \psi_i |" />

g

<img src="https://i.upmath.me/svg/%20%5Clambda_i%20" alt=" \lambda_i " />

h