[![hackmd-github-sync-badge](https://hackmd.io/S-zvJu8gQ36HTEUXZuONFA/badge)](https://hackmd.io/S-zvJu8gQ36HTEUXZuONFA)

# Simulating molecules(LiH) using VQE


###### Qiskit Summer School Final Project: Designing your implementation of a variational quantum eigensolver (VQE) algorithm that simulates the ground state energy of the Lithium Hydride (LiH) molecule.

![](https://i.imgur.com/3kqdmwI.png)
> By science focus

## INTRODUCTION
Fundamentally, light was believed to be a wave. However, Albert Einstein found that light consisted of photons called quanta, which have energy determined by its frequency. Photons of visible light can be absorbed by an electron, thereby causing the electron to move from a low energy orbital to a high energy orbital. 

Determining these properties is difficult for a classical computer since the electrons may be highly entangled. To model this accurately, we require more computational power, and this is where quantum computers come into play. Compared to classical computers, they are more efficient at handling entanglement.

LiH is a 12 body molecule containing 4 protons, 4 electrons, and 4 neutrons. This creates a 12 body model, which becomes intractable when simulating it both with a classical and quantum computer. So, this model reduces the First Quantized Molecular Hamiltonian to one two body interaction between two electrons in the hybridized p orbital and four one body interactions with their respective nuclei.

![](https://i.imgur.com/anvTZWN.png)
> By Wikipedia

One of the essential properties is the ground state energy of a molecule. Finding the ground state energy of these molecules becomes harder as the size of the molecule increases, which is why until now, the largest simulated molecule is Beryllium Hydride. Molecular simulations' problems grow exponentially as the size of molecules increase. Some of the applications of these problems are in drug discoveries.

> Richard Feynman said, "Nature isn't classical, dammit, and if you want to make a simulation of nature, you'd better make it quantum mechanical, and by golly, it's a wonderful problem because it doesn't look so easy."

## PROCESS
The variational principle explains how the energy of any trial wave function is greater than or equal to the system's exact ground state energy.

It is important to find the minimum eigenvalue of a matrix; in Chemistry, the minimum eigenvalue of a Hermitian matrix is the ground state energy.

Given a Hermitian matrix  H  with an unknown minimum eigenvalue <img src="https://i.upmath.me/svg/%5Clambda%20min" alt="\lambda min" />, associated with the eigenstate <img src="https://i.upmath.me/svg/%5Cmid%20%5Cpsi%20min%20%5Crangle%20" alt="\mid \psi min \rangle " />, VQE provides an estimate  <img src="https://i.upmath.me/svg/%5Clambda_%7B%5Ctheta%7D" alt="\lambda_{\theta}" />  bounding  <img src="https://i.upmath.me/svg/%5Clambda%20min" alt="\lambda min" /> λmin≤λθ≡⟨ψ(θ)|H|ψ(θ)⟩
 


#### 1. We map the molecular Hamiltonians into qubit Hamiltonians. 
We find the fermionic operators and map the fermionic Hamiltonians to qubit Hamiltonians. For this transformation we use
1. Jordan Wigner transformation leads to N-local Hamiltonians
2. Bravyi-Kotoev transformation leads to log(N) local Hamiltonians
3. Partity transformation

We then copy the electron orbitals interactions to qubits.

#### 2. We create an ansatz.
If we want to vary the ansatz then we need a parametrized circuit with a fixed form. There are various variational forms that exist, Ry, RyRz and UCCSD which utilize domain specific knowledge. By applying a parameterized circuit represented by a linear transform <img src="https://i.upmath.me/svg/%20U(%5Ctheta)" alt=" U(\theta)" /> to some intial state, Vacuum state <img src="https://i.upmath.me/svg/%7C%200%5Crangle" alt="|\0\rangle" /> or Hartree Fock state, <img src="https://i.upmath.me/svg/%7C%5Cpsi%5Crangle" alt="|\psi\rangle" /> the output generated is <img src="https://i.upmath.me/svg/U(%5Ctheta)%7C%5Cpsi%5Crangle%20%5Cequiv%20%7C%5Cpsi(%5Ctheta)%5Crangle" alt="U(\theta)|\psi\rangle \equiv |\psi(\theta)\rangle" />. This estimate is optimized by a classical optimizer changing the parameter <img src="https://i.upmath.me/svg/%5Ctheta" alt="\theta" /> thus minimizing the expectation value of <img src="https://i.upmath.me/svg/%5Clangle%20%5Cpsi(%5Ctheta)%20%7CH%7C%5Cpsi(%5Ctheta)%20%5Crangle" alt="\langle \psi(\theta) |H|\psi(\theta) \rangle" />. This is the variational principle.

Iterative optimization over <img src="https://i.upmath.me/svg/%7C%5Cpsi(%5Ctheta)%5Crangle" alt="|\psi(\theta)\rangle" /> aims to yield an expectation value  ⟨ψ(θ)|H|ψ(θ)⟩≈Egs≡λmin . Ideally,  <img src="https://i.upmath.me/svg/%5Clambda_%7B%5Ctheta%7D" alt="\lambda_{\theta}" />  will be close to  <img src="https://i.upmath.me/svg/%5Clambda%20min" alt="\lambda min" />  although in practice, useful bounds on  <img src="https://i.upmath.me/svg/E_%7Bgs%7D" alt="E_{gs}" />  can be obtained even if this is not the case. As the circuit depth increases so does the noise increase. In this case we use heuristic variational form RYRZ rather than the UCCSD as it is shallow and uses few gates.

> Hermitian matrix <img src="https://i.upmath.me/svg/H" alt="H" /> is equal to its own conjugate transpose
> 
> <img src="https://i.upmath.me/svg/%20H%20%3D%20H%5E%7B%5Cdagger%7D%20" alt=" H = H^{\dagger} " />

> By Spectral theorem the expectation value of the Hermitian matrix must be real and any expectation value has the property
> 
> <img src="https://i.upmath.me/svg/%20%5Clambda_i%20%3D%20%5Clambda_i%5E*" alt=" \lambda_i = \lambda_i^*" />

> 


The expectation value of any wave function will be at least the minimum eigenvalue associated with the wave function.
The ground state of the Hamiltonian system is the smallest eigenvalue associated with the Hermitian matrix.


Moreover, <img src="https://i.upmath.me/svg/H" alt="H" /> may be expressed as <img src="https://i.upmath.me/svg/%20H%20%3D%20%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%20%5Clambda_i%20%7C%5Cpsi_i%5Crangle%20%5Clangle%20%5Cpsi_i%20%7C" alt=" H = \sum_{i = 1}^{N} \lambda_i |\psi_i\rangle \langle \psi_i |" /> where each <img src="https://i.upmath.me/svg/%20%5Clambda_i%20" alt=" \lambda_i " />  is the eigenvalue corresponding to the eigenvector <img src="https://i.upmath.me/svg/%7C%5Cpsi_i%5Crangle" alt="|\psi_i\rangle" />. Furthermore, the expectation value of the observable  <img src="https://i.upmath.me/svg/H" alt="H" />  on an arbitrary quantum state <img src="https://i.upmath.me/svg/%20%7C%5Cpsi%5Crangle%20" alt=" |\psi\rangle " /> is given by <img src="https://i.upmath.me/svg/%20%5Clangle%20H%20%5Crangle_%7B%5Cpsi%7D%20%26%5Cequiv%20%5Clangle%20%5Cpsi%20%7C%20H%20%7C%20%5Cpsi%20%5Crangle%20" alt=" \langle H \rangle_{\psi} &amp;\equiv \langle \psi | H | \psi \rangle " />. Substituting <img src="https://i.upmath.me/svg/H" alt="H" /> with its representation as a weighted sum of its eigenvectors,


<img src="https://i.upmath.me/svg/%20%5Clangle%20H%20%5Crangle_%7B%5Cpsi%7D%20%3D%20%5Clangle%20%5Cpsi%20%7C%20H%20%7C%20%5Cpsi%20%5Crangle%20%26%3D%20%5Clangle%20%5Cpsi%20%7C%20%5Cleft(%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%20%5Clambda_i%20%7C%5Cpsi_i%5Crangle%20%5Clangle%20%5Cpsi_i%20%7C%5Cright)%20%7C%5Cpsi%5Crangle" alt=" \langle H \rangle_{\psi} = \langle \psi | H | \psi \rangle &amp;= \langle \psi | \left(\sum_{i = 1}^{N} \lambda_i |\psi_i\rangle \langle \psi_i |\right) |\psi\rangle" />

<img src="https://i.upmath.me/svg/%5Clangle%20H%20%5Crangle_%7B%5Cpsi%7D" alt="\langle H \rangle_{\psi}" /> = <img src="https://i.upmath.me/svg/%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%20%5Clambda_i%20%7C%20%5Clangle%20%5Cpsi_i%20%7C%20%5Cpsi%5Crangle%20%7C%5E2" alt="\sum_{i = 1}^{N} \lambda_i | \langle \psi_i | \psi\rangle |^2" />

Each of the weights in the linear combination is greater than or equal to 0, as and so it is clear that:

<img src="https://i.upmath.me/svg/%20%5Clambda_%7Bmin%7D%20%5Cle%20%5Clangle%20H%20%5Crangle_%7B%5Cpsi%7D%20%3D%20%5Clangle%20%5Cpsi%20%7C%20H%20%7C%20%5Cpsi%20%5Crangle%20%3D%20%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%20%5Clambda_i%20%7C%20%5Clangle%20%5Cpsi_i%20%7C%20%5Cpsi%5Crangle%20%7C%5E2" alt=" \lambda_{min} \le \langle H \rangle_{\psi} = \langle \psi | H | \psi \rangle = \sum_{i = 1}^{N} \lambda_i | \langle \psi_i | \psi\rangle |^2" />

The expectation value of the eigenstate <img src="https://i.upmath.me/svg/%5Cmid%20%5Cpsi%20min%20%5Crangle%20" alt="\mid \psi min \rangle " /> is given by  <img src="https://i.upmath.me/svg/%5Clangle%20%5Cpsi_%7Bmin%7D%7CH%7C%5Cpsi_%7Bmin%7D%5Crangle%20%3D%20%5Clangle%20%5Cpsi_%7Bmin%7D%7C%5Clambda_%7Bmin%7D%7C%5Cpsi_%7Bmin%7D%5Crangle%20%3D%20%5Clambda_%7Bmin%7D" alt="\langle \psi_{min}|H|\psi_{min}\rangle = \langle \psi_{min}|\lambda_{min}|\psi_{min}\rangle = \lambda_{min}" />. Thus, as expected, <img src="https://i.upmath.me/svg/%5Clangle%20H%20%5Crangle_%7B%5Cpsi_%7Bmin%7D%7D%3D%5Clambda_%7Bmin%7D" alt="\langle H \rangle_{\psi_{min}}=\lambda_{min}" />.


#### 3. Parameter optimisation
One problem that arises is noise, meaning the energy calculations may not be true. We try and overcome this by gradient descent, which also has its problems.
We use Simultaneous Perteburation Stochastic Approximation (SPSA) as the ideal optimizer. It works by perturbing all the parameters in random. 
Under noise, Sequential Least Squares Programming (SLSQP) and Constraint Optimisation by Linear Approximation (COBYLA) are preferred.






## IMPLEMENTATION

Notebook is [here](https://gist.github.com/0x6f736f646f/698ee32dde649ace70ad1152d276f748)
```python
from qiskit import BasicAer, Aer, IBMQ
from qiskit.aqua import QuantumInstance, aqua_globals
from qiskit.aqua.algorithms import VQE, ExactEigensolver
from qiskit.aqua.components.initial_states import Zero
from qiskit.aqua.components.optimizers import COBYLA, L_BFGS_B, SLSQP, SPSA
from qiskit.aqua.components.variational_forms import RY, RYRZ, SwapRZ
from qiskit.aqua.operators import WeightedPauliOperator, Z2Symmetries
from qiskit.chemistry import FermionicOperator
from qiskit.chemistry.drivers import PySCFDriver, UnitsType
from qiskit.chemistry.components.variational_forms import UCCSD
from qiskit.chemistry.components.initial_states import HartreeFock
from qiskit.circuit.library import EfficientSU2


from qiskit.providers.aer import QasmSimulator
from qiskit.providers.aer.noise import NoiseModel
from qiskit.providers.aer.noise.errors import QuantumError, ReadoutError
from qiskit.providers.aer.noise.errors import pauli_error
from qiskit.providers.aer.noise.errors import depolarizing_error
from qiskit.providers.aer.noise.errors import thermal_relaxation_error
from qiskit.ignis.mitigation.measurement import CompleteMeasFitter

from qiskit.providers.aer import noise
provider = IBMQ.load_account()

import numpy as np
import matplotlib.pyplot as plt
from functools import partial
```

    ibmqfactory.load_account:WARNING:2020-08-21 22:24:01,457: Credentials are already in use. The existing account in the session will be replaced.



```python
import warnings
warnings.filterwarnings("ignore", category=DeprecationWarning)
```

# Defining your molecule:
In this challenge, we will focus on LiH using the sto3g basis with the PySCF driver, which can be described in Qiskit as follows: 'inter_dist' is the interatomic distance.



```python
inter_dist = 1.6
```


```python
driver = PySCFDriver(atom='Li .0 .0 .0; H .0 .0' + str(inter_dist), unit=UnitsType.ANGSTROM, charge=0, spin=0, basis='sto3g')
```

We also set up the molecular orbitals to be considered and can reduce the problem size when we map to the qubit Hamiltonian. Hence, the amount of time required for the simulations is reasonable for a laptop computer.


```python
# please be aware that the idx here with respective to original idx
freeze_list = [0]
remove_list = [-3, -2] # negative number denotes the reverse order
```

#### Once you have computed the qubit operations for LiH, you can use the following function to classical solve for the exact solution. This is used to compare how well your VQE approximation is performing.


```python
#Classically solve for the lowest eigenvalue
def exact_solver(qubitOp):
    ee = ExactEigensolver(qubitOp)
    result = ee.run()
    ref = result['energy']
    print('Reference value: {}'.format(ref))
    return ref
```

Here we ask you to use the `statevector_simulator` as the simulation backend for your VQE algorithm.


```python
backend = BasicAer.get_backend('statevector_simulator')
```

### Now, you can start choosing the components that make up your VQE algorithm!

#### 1. Optimizers
The most commonly used optimizers are `COBYLA,` `L_BFGS_B,` `SLSQP,` and `SPSA.` 

#### 2. Qubit mapping
There are several different mappings for your qubit Hamiltonian, `parity,` `bravyi_kitaev,` `jordan_wigner,` which in some cases can allow you to reduce the problem size further.

#### 3. Initial state
There are different initial states that you can choose to start your simulation. Typically people choose from the zero state 
`init_state = Zero(qubitOp.num_qubits)` 
and the UCCSD initial state
`HartreeFock(qubitOp.num_qubits, num_spin_orbitals, num_particles, map_type, qubit_reduction)`

#### 4. Parameterized circuit
There are different choices you can make in the form of variational forms of your parameterized circuit.

`UCCSD_var_form = UCCSD(num_qubits, depth=depth, num_orbitals=num_spin_orbitals, num_particles=num_particles)`
    
`RY_var_form = RY(num_qubits, depth=depth)`
    
`RYRZ_var_form = RYRZ(num_qubits, depth=depth)`
    
`swaprz_var_form = SwapRZ(num_qubits, depth=depth)`

#### 5. Simulation backend
There are different simulation backends that you can use to perform your simulation.

`backend = BasicAer.get_backend('statevector_simulator')`

`backend=Aer.get_backend('qasm_simulator')`

### Compare the convergence of different choices for building your VQE algorithm

Among the above choices, which combination do you think would outperform others and give you the lowest estimation of LiH ground state energy with the quickest convergence? Compare the results of different combinations against each other and the classically computed exact solution at a fixed interatomic distance, for example, `inter_dist=1.6`. 

To access the intermediate data during the optimization, you would need to utilize the `callback` option in the VQE function:

`def store_intermediate_result(eval_count, parameters, mean, std):
            counts.append(eval_count)
            values.append(mean)
            params.append(parameters)
            deviation.append(std)`
            
`algo = VQE(qubitOp, var_form, optimizer, callback=store_intermediate_result)`

`algo_result = algo.run(quantum_instance)`



### Compute the ground state energy of LiH at various interatomic distances
By changing the parameter `inter_dist,` you can use your VQE algorithm to calculate the ground state energy of LiH at various interatomic distances, and potentially produce a plot as you see here. Note that the VQE results are very close to the exact results, and so the exact energy curve is hidden by the VQE curve.

### How does your VQE algorithm perform in the presence of noise?
Try importing the noise model and qubit coupling map of a real IBM quantum device into your simulation. You can use the imported noise model in your simulation by passing it into your quantum instance. You can also try enabling error mitigation to lower the effect of noise on your simulation results.


```python
#Define our noise model based on the ibmq_essex chip
chip_name = 'ibmq_essex'
device = provider.get_backend(chip_name)
coupling_map = device.configuration().coupling_map
noise_model = noise.device.basic_device_noise_model(device.properties())
basis_gates = noise_model.basis_gates
```


### You can find an example of using Qiskit to simulate molecules with VQE [here](https://qiskit.org/textbook/ch-applications/vqe-molecules.html).

```python
# Classically solve for the lowest eigenvalue
# This is used just to compare how well you VQE approximation is performing
def exact_solver(qubitOp):
    ee = ExactEigensolver(qubitOp)
    result = ee.run()
    ref = result['energy']
    print('Reference value: {}'.format(ref))
    return ref

# Define your function for computing the qubit operations of LiH
def compute_LiH_qubitOp(map_type, inter_dist, basis='sto3g'):
    
    # Specify details of our molecule
    driver = PySCFDriver(atom='Li .0 .0 .0; H .0 .0 ' + str(inter_dist), unit=UnitsType.ANGSTROM, charge=0, spin=0, basis=basis)

    # Compute relevant 1 and 2 body integrals.
    molecule = driver.run()
    h1 = molecule.one_body_integrals
    h2 = molecule.two_body_integrals
    nuclear_repulsion_energy = molecule.nuclear_repulsion_energy
    
    num_particles = molecule.num_alpha + molecule.num_beta
    num_spin_orbitals = molecule.num_orbitals * 2
    print("HF energy: {}".format(molecule.hf_energy - molecule.nuclear_repulsion_energy))
    print("# of electrons: {}".format(num_particles))
    print("# of spin orbitals: {}".format(num_spin_orbitals))

    # Please be aware that the idx here with respective to original idx
    freeze_list = [0]
    remove_list = [-3, -2] # negative number denotes the reverse order
    
    # Prepare full idx of freeze_list and remove_list
    # Convert all negative idx to positive
    remove_list = [x % molecule.num_orbitals for x in remove_list]
    freeze_list = [x % molecule.num_orbitals for x in freeze_list]
    
    # Update the idx in remove_list of the idx after frozen, since the idx of orbitals are changed after freezing
    remove_list = [x - len(freeze_list) for x in remove_list]
    remove_list += [x + molecule.num_orbitals - len(freeze_list)  for x in remove_list]
    freeze_list += [x + molecule.num_orbitals for x in freeze_list]

    # Prepare fermionic hamiltonian with orbital freezing and eliminating, and then map to qubit hamiltonian
    # and if PARITY mapping is selected, reduction qubits
    energy_shift = 0.0
    qubit_reduction = True if map_type == 'parity' else False

    ferOp = FermionicOperator(h1=h1, h2=h2)
    if len(freeze_list) > 0:
        ferOp, energy_shift = ferOp.fermion_mode_freezing(freeze_list)
        num_spin_orbitals -= len(freeze_list)
        num_particles -= len(freeze_list)
    if len(remove_list) > 0:
        ferOp = ferOp.fermion_mode_elimination(remove_list)
        num_spin_orbitals -= len(remove_list)

    qubitOp = ferOp.mapping(map_type=map_type)
    qubitOp = Z2Symmetries.two_qubit_reduction(qubitOp, num_particles) if qubit_reduction else qubitOp
    qubitOp.chop(10**-10)
    shift = energy_shift + nuclear_repulsion_energy 
    return qubitOp, num_spin_orbitals, num_particles, qubit_reduction, shift
```

```python
counts = []
values = []
params = []
deviation = []
def store_intermediate_result(eval_count, parameters, mean, std):
    counts.append(eval_count)
    values.append(mean)
    params.append(parameters)
    deviation.append(std)
```

### NOTE: I am not using the noisy model

```python
distances = np.arange(0.5, 4.0, 0.1)
exact_energies = []
vqe_energies = []
for dist in distances:
    qubitOp, num_spin_orbitals, num_particles, qubit_reduction, shift = compute_LiH_qubitOp(map_type='parity', inter_dist=dist)
    # Classically solve for the exact solution and use that as your reference value
    ref = exact_solver(qubitOp) + shift

    # Specify your initial state
    init_state = HartreeFock(num_spin_orbitals, num_particles, "parity", qubit_reduction)

    # Select a state preparation ansatz
    # Equivalently, choose a parameterization for our trial wave function.
    var_form = UCCSD(num_orbitals=num_spin_orbitals,
                     num_particles=num_particles,
                     initial_state=init_state,
                     qubit_mapping="parity")

    # Choose where to run/simulate our circuit
    quantum_instance = Aer.get_backend('statevector_simulator')

    # Choose the classical optimizer
    optimizer = SPSA()

    # Run your VQE instance
    vqe = VQE(qubitOp, var_form, optimizer, callback=store_intermediate_result)

    # Now compare the results of different compositions of your VQE algorithm!
    
    ret = vqe.run(quantum_instance)
    vqe_result = np.real(ret['eigenvalue']) + shift
    print("Interatomic Distance:", np.round(1.4, 2), "VQE Result:", vqe_result, "Exact Energy:", ref)
    exact_energies.append(ref)
    vqe_energies.append(vqe_result)
```

```python
plt.plot(np.arange(0.5, 4.0, 0.1), exact_energies, label="exact")
plt.plot(np.arange(0.5, 4.0, 0.1), vqe_energies, label="vqe")
plt.xlabel('Interatomic Distance (Angstrom)')
plt.ylabel('Bond Energy')
plt.legend()
plt.title('Bond energy vs interatomic distance')
plt.show()
```
![](https://i.imgur.com/BNDTgE8.png)

```python
# Dictionary of optimizers:
opt_dict = {'SPSA' , 'SLSQP' , 'COBYLA' , 'L_BFGS_B'}

for opt in opt_dict:
    print('Testing', str(opt) , 'optimizer')
    qubitOp, num_spin_orbitals, num_particles, qubit_reduction, shift = compute_LiH_qubitOp(map_type , 1.5)
    # Classically solve for the exact solution and use that as your reference value
    ref = exact_solver(qubitOp) + shift

    # Specify your initial state
    init_state = HartreeFock(num_spin_orbitals,num_particles, qubit_mapping='parity') 

    # Select a state preparation ansatz
    # Equivalently, choose a parameterization for our trial wave function.
    var_form = UCCSD(num_orbitals=num_spin_orbitals, num_particles=num_particles, qubit_mapping='parity')

    # Choose where to run/simulate our circuit
    quantum_instance = Aer.get_backend('statevector_simulator')

    # Choose the classical optimizer
    if opt == 'SPSA':
        optimizer = SPSA(max_trials = 500)
    elif opt == 'SLSQP':
        optimizer = SLSQP(maxiter = 1000)
    elif opt == 'L_BFGS_B':
        optimizer = L_BFGS_B(maxfun = 1000 ,  maxiter = 1000)
    elif opt == 'COBYLA':
        optimizer = COBYLA(maxiter = 1000)
    
    counts =[]
    values =[]
    params =[]
    deviation =[]

    # Run your VQE instance
    vqe = VQE(qubitOp, var_form, optimizer , callback = store_intermediate_result)
    vqe_results = vqe.run(quantum_instance)
    
    #Printing error in final value:
    ground_state_energy = vqe_results['eigenvalue'] + shift
    energy_error_ground = np.abs(np.real(ref) - ground_state_energy)
    print('Error:', str(energy_error_ground))
    # Calculating energy error
    vqe_energies = np.real(values) + shift
    energy_error = np.abs(np.real(ref) - vqe_energies)
    plt.plot(counts , energy_error , label=str(opt))
plt.legend()
plt.xlabel('Counts')
plt.ylabel('Energy Error/ Hartree')
plt.title('Energy Convergence of VQE: UCCSD Ansatz')
```
![](https://i.imgur.com/fPGYHjy.png)


```python
# Trying all combinations to get the most efficiency
maps_types = ['parity'] # For 'bravyi_kitaev', 'jordan_wigner' we need another noise model eg ibmq_16_melbourne or ibmq_qasm_simulator

init_states = ["HartreeFock(num_spin_orbitals, num_particles, map_type, qubit_reduction)",
              "Zero(qubitOp.num_qubits)"]

var_forms = ["UCCSD(num_orbitals=num_spin_orbitals, num_particles=num_particles, active_occupied=[0], active_unoccupied=[0, 1], initial_state=init_state, qubit_mapping=map_type, two_qubit_reduction=qubit_reduction)",
             "RY(qubitOp.num_qubits, depth=depth)",
             "RYRZ(qubitOp.num_qubits, depth=depth)",
             "SwapRZ(qubitOp.num_qubits, depth=depth)"]

backends = [
           "Aer.get_backend('qasm_simulator')"]

optimizers = ["COBYLA(maxiter=opt_max_eval)",
            "L_BFGS_B(maxiter=opt_max_eval)",
            "SLSQP(maxiter=opt_max_eval)",
            "SPSA()"]
depth = 1
opt_max_eval = 200
for map_type in maps_types:
    for i_state in init_states:
        for v_form in var_forms:
            for be in backends:
                for opt in optimizers:
                    print("map_type: ", map_type)
                    print("i_state: ", i_state)
                    print("v_form: ", v_form)
                    print("be: ", be)
                    print("opt: ", opt)

                    qubitOp, num_spin_orbitals, num_particles, qubit_reduction = compute_LiH_qubitOp(map_type, inter_dist)
                    # Classically solve for the exact solution and use that as your reference value
                    ref = exact_solver(qubitOp)

                    # Specify your initial state
                    init_state = eval(i_state)

                    # Select a state preparation ansatz
                    # Equivalently, choose a parameterization for our trial wave function.
                    var_form = eval(v_form)

                    # Choose where to run/simulate our circuit
                    quantum_instance = QuantumInstance(backend=eval(be),
                                                        noise_model=noise_model,
                                                        measurement_error_mitigation_cls=CompleteMeasFitter,
                                                        seed_simulator=167, seed_transpiler=167,)

                    # Choose the classical optimizer
                    optimizer = eval(opt)

                    # Run your VQE instance
                    counts = []
                    values = []
                    params = []
                    deviation = []
                    algo = VQE(qubitOp, var_form, optimizer, callback=store_intermediate_result)
                    results = algo.run(quantum_instance)
                    print('VQE Results: {:.12f}'.format(results.eigenvalue.real))
                    print('The total ground state energy is: {:.12f}'.format(results.eigenvalue.real))
                    print("Parameters: {}".format(results.optimal_point))

                    #plt.plot(counts, label="count")
                    plt.plot(values, label="Zero")
                    plt.xlabel('Eval count')
                    plt.ylabel('energy minimisation for various optimizers ')
                    plt.legend()
                    plt.show()

                    #print(counts)
                    #print(values)
                    #print(params)
                    #print(deviation)
                    # Now compare the results of different compositions of your VQE algorithm!

                    ## 8
                    print("---")
```


```python
import qiskit.tools.jupyter
%qiskit_version_table
%qiskit_copyright
```

<h3>Version Information</h3><table><tr><th>Qiskit Software</th><th>Version</th></tr><tr><td>Qiskit</td><td>0.19.0</td></tr><tr><td>Terra</td><td>0.14.0</td></tr><tr><td>Aer</td><td>0.5.1</td></tr><tr><td>Ignis</td><td>0.3.0</td></tr><tr><td>Aqua</td><td>0.7.0</td></tr><tr><td>IBM Q Provider</td><td>0.7.0</td></tr><tr><th>System information</th></tr><tr><td>Python</td><td>3.6.9 (default, Nov  7 2019, 10:44:02) 
[GCC 8.3.0]</td></tr><tr><td>OS</td><td>Linux</td></tr><tr><td>CPUs</td><td>2</td></tr><tr><td>Memory (Gb)</td><td>11.59366226196289</td></tr><tr><td colspan='2'>Sat Aug 22 00:12:58 2020 EAT</td></tr></table>


<div style='width: 100%; background-color:#d5d9e0;padding-left: 10px; padding-bottom: 10px; padding-right: 10px; padding-top: 5px'><h3>This code is a part of Qiskit</h3><p>&copy; Copyright IBM 2017, 2020.</p><p>This code is licensed under the Apache License, Version 2.0. You may<br>obtain a copy of this license in the LICENSE.txt file in the root directory<br> of this source tree or at http://www.apache.org/licenses/LICENSE-2.0.<p>Any modifications or derivative works of this code must retain this<br>copyright notice, and modified files need to carry a notice indicating<br>that they have been altered from the originals.</p></div>


Hopefully, you have gained a firm understanding of how VQE works and can now implement one on your own. Leave some claps and follow me for more content. I worked with [Victor](https://github.com/VictorCarlquist)
