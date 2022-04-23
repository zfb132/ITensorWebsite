# <img src="docs/VERSION/formulas/icon.png" class="largeicon">  Code Formulas #

Below are code "formulas" for common tasks that come up when using ITensor. 
Please <a href="/about.html">email us</a> to suggest a formula you would
like to see.

## General Formulas 

* [[Creating your own "driver" code that builds against ITensor|formulas/driver]]
* [[Running your program in debug mode|formulas/debug_mode]]
* [[Reading input from a file|formulas/input]]
* [[Writing and reading ITensor objects to and from HDF5 files|formulas/hdf5]]

## ITensor Formulas
Formulas about working with ITensors, Index, etc.

* [[Evaluate a Function on Each Element of a Tensor|formulas/visit]]
* [[Extract the Storage of a Dense ITensor|formulas/extractdense]]
* [[Create a set of Site indices to use as a lattice|formulas/index_sites]]
* [[Make a single-site operator (no quantum numbers)|formulas/itensor_single_site_op]]
* [[Exactly Compute the Ground State of a Small Hamiltonian|formulas/simple_ed]]
* [[Replace an ITensor Index with Another Index|formulas/itensor_replace_index]]

## MPS and DMRG Formulas
Formulas involving matrix product states and DMRG.

* **MPS Basics & Updating MPS**
    - [[Obtaining Elements or Components of a Tensor Represented as an MPS|formulas/mps_element]]
    - [[Applying a one-site operator to an MPS|formulas/mps_onesite_op]]
    - [[Applying a two-site 'gate' to an MPS|formulas/gate]]

* **Running DMRG**
    - [[Perform a basic DMRG calculation|formulas/basic_dmrg]]
    - [[Perform a quantum number (QN) conserving DMRG calculation|formulas/qn_dmrg]]
    - [[Make a Ladder Hamiltonian for DMRG|formulas/ladder]]
    - [[Make a 2D Hamiltonian for DMRG|formulas/2d_dmrg]]
    - [[Compute excited states using DMRG|formulas/excited_dmrg]]
    - [[Read and write an MPS or MPO to and from disk|formulas/readwrite_mps]]
    - [[Stopping a DMRG Run "Gracefully"|formulas/stopping_dmrg]]
    - [[Making a Custom DMRGObserver|formulas/dmrg_observer]]
    - [[Using a Mixed Site Set (Holstein Polaron Example)|formulas/gs_holst_polaron]]

* **Measuring MPS**
    - [[Measure local properties of an MPS wavefunction|formulas/measure_mps]]
    - [[Measure two-point correlator from an MPS wavefunction|formulas/correlator_mps]]
    - [[Compute entanglement entropy|formulas/entanglement_mps]]
    - [[Measure spinless fermion two-point correlator|formulas/spinless_correlator_mps]]
    - [[Compute a two-site reduced density matrix from an MPS|formulas/mps_two_rdm]]
    - [[Operator matrix elements involving two MPS|formulas/two_mps]]

* **Time Evolution & Quantum Circuits**
    - [[Time-evolving an MPS with Trotter Gates|formulas/tevol_trotter]]
    - [[Time-evolving an MPS with an MPO (matrix product operator)|formulas/tevol_mps_mpo]]
    - [[Turning a set of gates into an MPO|formulas/gates_to_mpo]]

