# Forked to be used in AiiDA

### Default settings:
 * ionizationdata.dat \
EA: experimental, [T.Andersen et al., 1999](http://aip.scitation.org/doi/10.1063/1.556047)\
IP: experimental, [C.E.Moore, 1970](https://nvlpubs.nist.gov/nistpubs/Legacy/NSRDS/nbsnsrds34.pdf)

 * chargecenters.dat: common oxidation state (lowered if missing ionizationdata)
 * Number of expansion cells: 2 (5x5x5)
 * relative dielectric constant (eps_eff): 1.67

### Look in the example folder for a run.sh script to run the program and specify the input settings. 

EQeq
====

Charge equilibration method for crystal structures.
### Summary

The source code in this program demonstrates the charge equilibration method described
in the accompanying paper. The purpose of the source code provided is to be
minimalistic and do "just the job" described. In practice, you may wish to add various
features to the source code to fit the particular needs of your project.

#### Major highlights of program:

 * Obtains charges for atoms in periodic systems without iteration
 * Can use non-neutral charge centers for more accurate point charges
 * Designed for speed (but without significant code optimizations)

#### Features not implemented but that you may want to consider adding:

 * Spherical cut-offs (for both real-space and reciprocal-space sums)
 * An iterative loop that guesses the appropriate charge center (so the user does not have to guess)
 * Ewald parameter auto-optimization
 * Various code optimizations

#### Running the program:

Program expects two input files `ionization.dat` and `chargecenters.dat`. Please
look at source code to see what the other optional inputs are for (should be
mostly self-explanatory). Compile with something like:

```
g++ main.cpp -O3 -o eqeq
```

and run with

```
./eqeq my_file.cif
```

#### Python bindings

To facilitate automation and scaling, this version of EQeq can be operated via
Python. To enable, you must build EQeq as a shared library:

```
g++ -c -fPIC main.cpp -O3 -o eqeq.o
g++ -shared -Wl,-soname,libeqeq.so -O3 -o libeqeq.so eqeq.o
sudo cp libeqeq.so /usr/lib
```

(for Macs, replace `-soname` with `-install_name`)
(if you don't have sudo access, `mkdir ~/lib; cp libeqeq.so ~/lib`, then change
the path at the top of `eqeq.py`)

From the command line, you can run `eqeq.py`:

```
python eqeq.py --help
python eqeq.py IRMOF-1.cif --output-type mol --method ewald
```

This includes extensive help documentation on running EQeq, and an easy
interface to change individual parameters.

To use globally with Python scripts, you must put the `EQeq` directory on your
PYTHONPATH:

```
export PYTHONPATH:/path/above/EQeq:$PYTHONPATH
```

Then, you can call EQeq from Python with

```python
import EQeq
EQeq.run("IRMOF-1.cif")
```

The input takes both filenames and actual data, and outputs either files or
strings. This change allows for streaming data, which enables EQeq to be used
as part of a broader code pipeline.

```python
import EQeq
# Load a file. In practice, this can come from any source, such as a database
with open("IRMOF-1.cif") as in_file:
    data = in_file.read()
charges = EQeq.run(data, output_type="list", method="ewald")
```
