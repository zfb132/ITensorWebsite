
# Quantum Number Conserving DMRG

An important technique in DMRG calculations of quantum Hamiltonians
is the conservation of _quantum numbers_. Examples of these are the
total number of particles of a model of fermions, or the total of all
@@S^z@@ components of a system of spins. Not only can conserving quantum
numbers make DMRG calculations run more quickly and use less memory, but
it can be important for simulating physical systems with conservation
laws and for obtaining ground states in different symmetry sectors.
Note that ITensor currently only supports Abelian quantum numbers.

#### Necessary Changes

Setting up a quantum-number conserving DMRG calculation in ITensor requires
only very small changes to a DMRG code. The main changes are:

1. using tensor indices (`Index` objects) which carry quantum number (QN) information to build your Hamiltonian and  initial state
2. initializing your MPS to have well-defined total quantum numbers

Importantly, _the total QN of your state throughout the calculation will 
remain the same as the initial state passed to DMRG_.
The total QN of your state is not set separately, but determined 
implicitly from the initial QN of the state when it is first constructed.

Of course, your Hamiltonian should conserve all of the QN's that you would
like to use. If it doesn't, you will get an error when you try to construct
it out of the QN-enabled tensor indices.

#### Making the Changes

Let's see how to make these two changes to the 
[[Basic DMRG Formula|formulas/basic_dmrg]] code formula.
At the end, we will put together these changes for a complete, working code.

**Change 1: QN Site Indices**

To make change (1), we will change the line

    auto sites = SpinOne(N,{"ConserveQNs=",false});

by setting the `ConserveQNs` keyword argument to `true`:

    auto sites = SpinOne(N,{"ConserveQNs=",true});

Setting `ConserveQNs` to true makes a SpinOne site set with
every possible quantum number associated to the site
type. For S=1 spins, this will turn on total-``S^z`` conservation.
(For other site types that conserve multiple QNs, there are specific keyword 
arguments available to track just a subset of conservable QNs.)
We can check this by printing out some of the site indices, and seeing that the
subspaces of each `Index` are labeled by QN values:

    println("site 1 =",sites(1));
    println("site 2 =",sites(2));

Sample output:

    site 1 =(dim=3|id=489|"S=1,n=1,Site") <Out>
      1: 1 QN({"Sz",2})
      2: 1 QN({"Sz",0})
      3: 1 QN({"Sz",-2})
    site 2 =(dim=3|id=115|"S=1,n=2,Site") <Out>
      1: 1 QN({"Sz",2})
      2: 1 QN({"Sz",0})
      3: 1 QN({"Sz",-2})

In the sample output above, note than in addition to the dimension of these indices being 3, each of the three settings of the Index have a unique QN associated to them. The number after the QN on each line is the dimension of that subspace, which is 1 for each subspace of the Index objects above. Note also that `"Sz"` quantum numbers in ITensor are measured in units of ``1/2``, so `QN({"Sz",2})` corresponds to @@S^z=1@@ in conventional physics units.

**Change 2: Initial State**

To make change (2), instead of constructing the initial MPS `psi0` to be an arbitrary, random MPS, we will make it a specific state with a well-defined total @@S^z@@. 
So we will replace the line

    auto psi0 = randomMPS(sites);

by the lines

    auto state = InitState(sites);
    for(auto i : range1(N))
        {
        if(i%2 == 1) state.set(i,"Up");
        else         state.set(i,"Dn");
        }
    auto psi0 = MPS(state);

The variable `state` is a container that holds information about what
local state we want each site to be on, to make an MPS which is a product
state. We fill up the state container with the strings "Up" or "Dn"
to make an alternating pattern with a total @@S^z@@ of zero.
Finally, passing the state container to the MPS constructor makes
`psi0` to be the product state we want.

We can check that `psi0` has the right total quantum number as follows

    println("totalQN = ",totalQN(psi0));
    // Output: totalQN = QN({"Sz",0})

#### Putting it All Together

Let's take the [[Basic DMRG Formula|formulas/basic_dmrg]] code
from the previous section and make the changes discussed above, 
to turn it into a code which conserves the total @@S^z@@ quantum 
number throughout the DMRG calculation. The resulting code is:

    #include "itensor/util/print_macro.h"
    using namespace itensor;

    int main()
        {
        int N = 100;

        auto sites = SpinOne(N,{"ConserveQNs=",true});

        println("site 1 =",sites(1));
        println("site 2 =",sites(2));

        auto ampo = AutoMPO(sites);
        for(int j = 1; j < N; ++j)
            {
            ampo += 0.5,"S+",j,"S-",j+1;
            ampo += 0.5,"S-",j,"S+",j+1;
            ampo +=     "Sz",j,"Sz",j+1;
            }
        auto H = toMPO(ampo);

        auto sweeps = Sweeps(5); //number of sweeps is 5
        sweeps.maxdim() = 10,20,100,100,200;
        sweeps.cutoff() = 1E-10;

        auto psi0 = randomMPS(sites);

        println("totalQN = ",totalQN(psi0));

        auto [energy,psi] = dmrg(H,psi0,sweeps);

        println("Ground State Energy = ",energy);

        return 0;
        }
