# Making a Custom DMRG Observer

The Observer system lets you define custom measurements to be performed within algorithms
such as DMRG and time evolution. For the DMRG algorithm, you can make a subclass of
the `DMRGObserver` type to implement custom measurements and printing of information
from the ITensor `dmrg` function.

Let's see how to make a custom DMRG Observer by way of example.

## Making our CustomObserver type

First make a type that is a subclass of `DMRGObserver`. We will call ours
`CustomObserver`:

```
class CustomObserver : public DMRGObserver
    {
    public:

    CustomObserver(MPS const& psi,
                   Args const& args = Args::global())
        : DMRGObserver(psi)
        {}

    }; // class CustomObserver
```

Note that the parent type, `DMRGObserver`, requires an MPS to be
passed to it, so we also require our `CustomObserver` to take an
MPS in its constructor. This MPS is a reference to the one
being optimized by the `dmrg` function.

## Overloading the `measure` method

The `DMRGObserver::measure` method is what gets called by the `dmrg` function after
each step. We can overload this method to change the information that gets 
printed by the `dmrg` function or to perform other custom measurements.

To overload `measure`, first we need to declare it as one of `CustomObserver`'s methods:

```
class CustomObserver : public DMRGObserver
    {
    public:

    CustomObserver(MPS const& psi,
                   Args const& args = Args::global())
        : DMRGObserver(psi)
        {}

    void virtual
    measure(Args const& args);

    }; // class CustomObserver
```

Next we need to actually define this method (outside of the class):

```
void CustomObserver::
measure(Args const& args = Args::global())
    {
    // your custom measurement code goes here
    }
```

Now we can put any code we want into `measure` to measure the state being optimized
by `dmrg` or report on the status of the DMRG optimization process.

## Customizing the `measure` method

What kind of properties are available to `measure`? First of all, we can access
the MPS being optimized by DMRG by calling `DMRGObserver::psi()`:

```
void CustomObserver::
measure(Args const& args = Args::global())
    {
    // Obtain a reference to psi, the MPS being optimized:
    auto& psi = DMRGObserver::psi();
    }
```

Just as usefully, we can get a lot of information from the `dmrg` function
itself that gets passed as named arguments through the `args` object.
Here are the different named arguments available:

```
void CustomObserver::
measure(Args const& args = Args::global())
    {
    // Obtain a reference to psi, the MPS being optimized:
    auto& psi = DMRGObserver::psi();

    // How many sweeps of DMRG are to be performed:
    auto nsweep = args.getInt("NSweep",0);
    // Which sweep of DMRG we are on:
    auto sw = args.getInt("Sweep",0);
    // Which half sweep of DMRG we are on:
    auto ha = args.getInt("HalfSweep",0);
    // Which bond DMRG is on:
    auto b = args.getInt("AtBond",1);

    // What is the current energy?
    auto energy = args.getReal("Energy",0);
    // What was the last truncation error computed?
    auto truncerr = args.getReal("Truncerr",0);
    // What is the current truncation error cutoff?
    auto cutoff = args.getReal("Cutoff",0);
    // What is the current maximum bond dimension?
    auto maxdim = args.getInt("MaxDim",0);

    // Should output be silenced?
    auto silent = args.getBool("Silent",false);


    }
```

You do not have to define all of these variables. The code above is just to illustrate
which named arguments are available to you to use.

## Calling the default `DMRGObserver::measure` method

When you overload `measure`, by default it "shadows" the one defined
by `DMRGObserver` which will no longer get called. However, the implementation of
`measure` inside of `DMRGObserver` performs a number of useful measurements, 
such as reporting the entanglement entropy at the center of the MPS during
each sweep.

To still run the `DMRGObserver::measure` method while also having your own
custom implementation, just put the line `DMRGObserver::measure(args);`
somewhere inside your own `measure` function.

## Overloading the `checkDone` method

Subclasses of `DMRGObserver` can also overload a method named `checkDone`
which returns a `bool`. If `checkDone` returns `true`, then the `dmrg` function will exit,
returning the current energy and MPS wavefunction.

To overload `checkDone`, first declare it in your `CustomObserver` class:

```
class CustomObserver : public DMRGObserver
    {
    public:

    CustomObserver(MPS const& psi,
                   Args const& args = Args::global())
        : DMRGObserver(psi)
        {}

    void virtual
    measure(Args const& args);

    bool virtual
    checkDone(Args const& args);

    }; // class CustomObserver
```

Next define the `checkDone` method itself (outside of the class):

```
bool virtual
checkDone(Args const& args = Args::global())
    {
    return false;
    }
```

To customize this method, you can read in the same set of named arguments as
in the `measure` function above. Let's use just one of them, the energy, to
determine when we are going to stop our `dmrg` calculation:

```
bool virtual
checkDone(Args const& args = Args::global())
    {
    auto energy = args.getReal("Energy");
    if(energy < -100.0) return true;
    return false;
    }
```

This is just a contrived example, with -100 being a made-up value, and in
your own application you might want to store the last reported energy inside
your `CustomObserver` and check whether the latest energy differs from
the last one by some relative amount. Or you could set a `checkDone` criterion
based on the truncation error, entanglement entropy, or any other quantity
you wish. 
