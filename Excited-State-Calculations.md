# CIS, TDHF, TDDFT

## Overview

NWChem supports a spectrum of single excitation theories for vertical
excitation energy calculations, namely, configuration interaction
singles (CIS)[^1], time-dependent Hartree-Fock (TDHF or also known as
random-phase approximation RPA), time-dependent density functional
theory (TDDFT)[^2], and Tamm-Dancoff approximation[^3] to TDDFT. These
methods are implemented in a single framework that invokes Davidson's
trial vector algorithm (or its modification for a non-Hermitian
eigenvalue problem). The capabilities of the module are summarized as
follows:

  - Vertical excitation energies (valence and core),
  - Spin-restricted singlet and triplet excited states for closed-shell
    systems,
  - Spin-unrestricted doublet, etc., excited states for open-shell
    systems,
  - Tamm-Dancoff and full time-dependent linear response theories,
  - Davidson's trial vector algorithm,
  - Symmetry (irreducible representation) characterization and
    specification,
  - Spin multiplicity characterization and specification,
  - Transition moments and oscillator strengths,
  - Analytical first derivatives of vertical excitation energies with a
    selected set of exchange-correlation functionals (see TDDFT
    gradients documentation for further information),
  - Numerical second derivatives of vertical excitation energies,
  - Disk-based and fully incore algorithms,
  - Multiple and single trial-vector processing algorithms,
  - Frozen core and virtual approximation,
  - Asymptotically correct exchange-correlation potential by van Leeuwen
    and Baerends[^4],
  - Asymptotic correction by Casida and Salahub[^5],
  - Asymptotic correction by Hirata, Zhan, Aprà, Windus, and Dixon[^6].

These are very effective way to rectify the shortcomings of TDDFT when
applied to Rydberg excited states (see below).

## Performance of CIS, TDHF, and TDDFT methods

The accuracy of CIS and TDHF for excitation energies of closed-shell
systems are comparable to each other, and are normally considered a
zeroth-order description of the excitation process. These methods are
particularly well balanced in describing Rydberg excited states, in
contrast to TDDFT. However, for open-shell systems, the errors in the
CIS and TDHF excitation energies are often excessive, primarily due to
the multi-determinantal character of the ground and excited state wave
functions of open-shell systems in a HF reference. The scaling of
the computational cost of a CIS or TDHF calculation per state with
respect to the system size is the same as that for a HF calculation for
the ground state, since the critical step of the both methods are the
Fock build, namely, the contraction of two-electron integrals with
density matrices. It is usually necessary to include two sets of diffuse
exponents in the basis set to properly account for the diffuse Rydberg
excited states of neutral species.

The accuracy of TDDFT may vary depending on the exchange-correlation
functional. In general, the exchange-correlation functionals that are
widely used today and are implemented in NWChem work well for low-lying
valence excited states. However, for high-lying diffuse excited states
and Rydberg excited states in particular, TDDFT employing these
conventional functionals breaks down and the excitation energies are
substantially underestimated. This is because of the fact that the
exchange-correlation potentials generated from these functionals decay
too rapidly (exponentially) as opposed to the slow *-1/r*  asymptotic
decay of the true potential. A rough but useful index is the negative of
the highest occupied KS orbital energy; when the calculated excitation
energies become close to this threshold, these numbers are most likely
underestimated relative to experimental results. It appears that TDDFT
provides a better-balanced description of radical excited states. This
may be traced to the fact that, in DFT, the ground state wave function
is represented well as a single KS determinant, with less
multi-determinantal character and less spin contamination, and hence the
excitation thereof is described well as a simple one electron
transition. The computational cost per state of TDDFT calculations
scales as the same as the ground state DFT calculations, although the
prefactor of the scaling may be much greater in the former.

A very simple and effective way to rectify the TDDFT's failure for
Rydberg excited states has been proposed by Tozer and Handy[^8] and by Casida and
Salahub[^5]. They proposed to splice a *-1/r*
asymptotic tail to an exchange-correlation potential that does not have
the correct asymptotic behavior. Because the approximate
exchange-correlation potentials are too shallow everywhere, a negative
constant must be added to them before they can be spliced to the *-1/r*
 tail seamlessly in a region that is not sensitive to chemical
effects or to the long-range behavior. The negative constant or the
shift is usually taken to be the difference of the HOMO energy from the
true ionization potential, which can be obtained either from experiment
or from a ΔSCF calculation. Recently, we proposed a new, expedient, and
self-contained asymptotic correction that does not require an ionization
potential (or shift) as an external parameter from a separate
calculation. In this scheme, the shift is computed by a semi-empirical
formula proposed by Zhan, Nichols, and Dixon[^6]. Both Casida-Salahub
scheme and this new asymptotic correction scheme give considerably
improved (Koopmans type) ionization potentials and Rydberg excitation
energies. The latter, however, supply the shift by itself unlike to
former.

## Input syntax

The module is called `TDDFT` as time-dependent density functional
theory employing a hybrid HF-DFT functional
encompasses all of the above-mentioned methods implemented. To use this
module, one needs to specify `TDDFT` on the task directive, e.g.,

```
     TASK TDDFT ENERGY
```

for a single-point excitation energy calculation, and

```
     TASK TDDFT OPTIMIZE
```

for an excited-state geometry optimization (and perhaps an adiabatic
excitation energy calculation), and

```
     TASK TDDFT FREQUENCIES
```

for an excited-state vibrational frequency calculation. The TDDFT module
first invokes DFT module for a ground-state calculation (regardless of
whether the calculations uses a HF reference as in CIS or TDHF or a DFT
functional), and hence there is no need to perform a separate
ground-state DFT calculation prior to calling a `TDDFT` task. When no
second argument of the task directive is given, a single-point
excitation energy calculation will be assumed. For geometry
optimizations, it is usually necessary to specify the target excited
state and its irreducible representation it belongs to. See the
subsections [`TARGET` and `TARGETSYM`](#target-and-targetsym-the-target-root-and-its-symmetry) for more detail.

Individual parameters and keywords may be supplied in the `TDDFT` input
block. The syntax is:
```
 TDDFT
   [(CIS||RPA) default RPA]  
   [NROOTS <integer nroots default 1>]  
   [MAXVECS <integer maxvecs default 1000>] 
   [(SINGLET||NOSINGLET) default SINGLET]  
   [(TRIPLET||NOTRIPLET) default TRIPLET]  
   [THRESH <double thresh default 1e-4>]  
   [MAXITER <integer maxiter default 100>]  
   [TARGET <integer target default 1>]  
   [TARGETSYM <character targetsym default 'none'>]  
   [SYMMETRY]  
   [ECUT] <-cutoff energy>  
   [EWIN] <-lower cutoff energy>  <-higher cutoff energy>  
   [ALPHA] <integer lower orbital>  <integer upper orbital>  
   [BETA] <integer lower orbital>  <integer upper orbital>  
   [CIVECS]  
   [GRAD, END]  
   [CDSPECTRUM]  
   [GIAO]
   [VELOCITY]  
   [ALGORITHM <integer algorithm default 0>]  
   [FREEZE [[core] (atomic || <integer nfzc default 0>)] \  
            [virtual <integer nfzv default 0>]]  
   [PRINT (none||low||medium||high||debug)  
     <string list_of_names ...>]
 END
```
The user can also specify the reference wave function in the DFT input
block (even when CIS and TDHF calculations are requested). See the
section of Sample input and output for more details.

Since each keyword has a default value, a minimal input file will be
```
 GEOMETRY
  Be 0.0 0.0 0.0  
 END  
 BASIS  
  Be library 6-31G**  
 END  
 TASK TDDFT ENERGY
```
Note that the keyword for the asymptotic correction must be given in the
DFT input block, since all the effects of the correction (and also
changes in the computer program) occur in the SCF calculation stage. See
[DFT](Density-Functional-Theory-for-Molecules#lb94-and-cs00-asymptotic-correction) (keywords `CS00` and `LB94`) for details.

## Keywords of TDDFT input block

### CIS and RPA: the Tamm-Dancoff approximation

These keywords toggle the Tamm-Dancoff approximation. `CIS` means that the
Tamm-Dancoff approximation is used and the CIS or Tamm-Dancoff TDDFT
calculation is requested. `RPA`, which is the default, requests TDHF (RPA)
or TDDFT calculation.

The performance of CIS (Tamm-Dancoff TDDFT) and RPA (TDDFT) are
comparable in accuracy. However, the computational cost is slightly
greater in the latter due to the fact that the latter involves a
non-Hermitian eigenvalue problem and requires left and right
eigenvectors while the former needs just one set of eigenvectors of a
Hermitian eigenvalue problem. The latter has much greater chance of
aborting the calculation due to triplet near instability or other
instability problems.

### NROOTS: the number of excited states

One can specify the number of excited state roots to be determined. The
default value for `NROOTS` is 1. It is advised that the users request several more
roots than actually needed, since owing to the nature of the trial
vector algorithm, some low-lying roots can be missed when they do not
have sufficient overlap with the initial guess vectors.

### MAXVECS: the subspace size

The `MAXVECS` keyword limits the subspace size of Davidson's algorithm; in other
words, it is the maximum number of trial vectors that the calculation is
allowed to hold. Typically, 10 to 20 trial vectors are needed for each
excited state root to be converged. However, it need not exceed the
product of the number of occupied orbitals and the number of virtual
orbitals. The default value is 1000.

### SINGLET and NOSINGLET: singlet excited states

`SINGLET || NOSINGLET` requests (suppresses) the calculation of singlet
excited states when the reference wave function is closed shell. The
default is `SINGLET`.

### TRIPLET and NOTRIPLET: triplet excited states

`TRIPLET || NOTRIPLET` requests (suppresses) the calculation of triplet
excited states when the reference wave function is closed shell. The
default `is TRIPLET`.

### THRESH: the convergence threshold of Davidson iteration

The `THRESH` keyword specifies the convergence threshold of Davidson's iterative
algorithm to solve a matrix eigenvalue problem. The threshold refers to
the norm of residual, namely, the difference between the left-hand side
and right-hand side of the matrix eigenvalue equation with the current
solution vector. With the default value of 1e-4, the excitation energies
are usually converged to 1e-5 hartree.

### MAXITER: the maximum number of Davidson iteration

It typically takes 10-30 iterations for the Davidson algorithm to get
converged results. The default value for `MAXITER` is 100.

### TARGET and TARGETSYM: the target root and its symmetry

At the moment, excited-state first geometry derivatives can be
calculated analytically for a set of functionals, while excited-state
second geometry derivatives are obtained by numerical differentiation.
These keywords may be used to specify which excited state root is being
used for the geometrical derivative calculation. For instance, when
`TARGET 3` and `TARGETSYM a1g` are included in the input block, the total
energy (ground state energy plus excitation energy) of the third lowest
excited state root (excluding the ground state) transforming as the
irreducible representation a1g will be passed to the module which
performs the derivative calculations. The default values for `TARGET` and `TARGETSYM`
 are `1` and `none`, respectively.

The keyword `TARGETSYM` is essential in excited state geometry
optimization, since it is very common that the order of excited states
changes due to the geometry changes in the course of optimization.
Without specifying the `TARGETSYM`, the optimizer could (and would likely)
be optimizing the geometry of an excited state that is different from
the one the user had intended to optimize at the starting geometry. On
the other hand, in the frequency calculations, `TARGETSYM` must be `none`,
since the finite displacements given in the course of frequency
calculations will lift the spatial symmetry of the equilibrium geometry.
When these finite displacements can alter the order of excited states
including the target state, the frequency calculation is not be
feasible.

### SYMMETRY: restricting the excited state symmetry

By adding the `SYMMETRY` keyword to the input block, the user can request the
module to generate the initial guess vectors transforming as the same
irreducible representation as `TARGETSYM`. This causes the final excited
state roots be (exclusively) dominated by those with the specified
irreducible representation. This may be useful, when the user is
interested in just the optically allowed transitions, or in the geometry
optimization of an excited state root with a particular irreducible
representation. By default, this option is not set. `TARGETSYM` must be
specified when `SYMMETRY` is invoked.

### ECUT: energy cutoff

The `ECUT` keyword enables restricted excitation window TDDFT (REW-TDDFT)[^9].
This is an approach best suited for core excitations. By specifying this
keyword only excitations from occupied states below the energy cutoff
will be considered.

### EWIN: energy window

The `EWIN` keyword enables a restricted energy window between a lower energy
cutoff and a higher energy cutoff. For example, `ewin -20.0 -10.0` will
only consider excitations from occupied orbitals within the specified
energy window

### Alpha, Beta: alpha, beta orbital windows

Orbital windows can be specified using the following keywords:
```
  alpha 1 4
  beta 2 5
```
Here alpha excitations will be considered from orbitals 1 through 4
depending on the number of roots requested and beta excitations will be
considered from orbitals 2 through 5 depending on the number of roots
requested.

### CIVECS: CI vectors

The `CIVECS` keyword will result in the CI vectors being written out. By default
this is off. Please note this can be a very large file, so avoid turning
on this keyword if you are calculating a very large number of roots. CI
vectors are needed for excited-state gradient and transition density
calculations.

### GRAD: TDDFT gradients

Analytical TDDFT gradients can be calculated by specifying a `grad` block
within the main `TDDFT` block

For example, the following will perform a TDDFT optimization on the
first singlet excited state (S1). Note that the `civecs` keyword must be
specified. To perform a single TDDFT gradient, replace the `optimize`
keyword with `gradient` in the task line. A complete TDDFT optimization
input example is given the Sample Inputs section. A TDDFT gradients
calculation can be used to calculate the density of a
specific excited state.
The excited stated density is written to a file with the `.dmat` suffix.

```
tddft
 nroots 2
 algorithm 1
 notriplet
 target 1
 targetsym a
 civecs
 grad
   root 1
 end
end
task tddft optimize
```

At the moment the following exchange-correlation functionals are
supported with TDDFT
gradients
```
LDA, BP86, PBE, BLYP, B3LYP, PBE0, BHLYP, CAM-B3LYP, LC-PBE, LC-PBE0, BNL, LC-wPBE, LC-wPBEh, LC-BLYP
```
### CDSpectrum: optical rotation calculations

Perform optical rotation calculations.
We recommend to use the `GIAO` keyword

### VELOCITY: velocity gauge

Perform CD spectrum calculations with the velocity gauge.

### ALGORITHM: algorithms for tensor contractions

There are four distinct algorithms to choose from, and the default value
of 0 (optimal) means that the program makes an optimal choice from the
four algorithms on the basis of available memory. In the order of
decreasing memory requirement, the four algorithms are:

  - ALGORITHM 1 : Incore, multiple tensor contraction,
  - ALGORITHM 2 : Incore, single tensor contraction,
  - ALGORITHM 3 : Disk-based, multiple tensor contraction,
  - ALGORITHM 4 : Disk-based, single tensor contraction.

The incore algorithm stores all the trial and product vectors in memory
across different nodes with the GA, and often decreases the [`MAXITER`](#maxiter-the-maximum-number-of-davidson-iteration)
value to accommodate them. The disk-based algorithm stores the vectors
on disks across different nodes with the DRA, and retrieves each vector
one at a time when it is needed. The multiple and single tensor
contraction refers to whether just one or more than one trial vectors
are contracted with integrals. The multiple tensor contraction algorithm
is particularly effective (in terms of speed) for CIS and TDHF, since
the number of the direct evaluations of two-electron integrals is
diminished substantially.

### FREEZE: the frozen core/virtual approximation

Some of the lowest-lying core orbitals and/or some of the highest-lying
virtual orbitals may be excluded in the CIS, TDHF, and TDDFT
calculations by the `FREEZE` keyword (this does not affect the ground state HF
or DFT calculation). No orbitals are frozen by default. To exclude the
atom-like core regions altogether, one may request

```
 FREEZE atomic
```

To specify the number of lowest-lying occupied orbitals be excluded, one
may use

```
 FREEZE 10
```

which causes 10 lowest-lying occupied orbitals excluded. This is
equivalent to writing

```
 FREEZE core 10
```

To freeze the highest virtual orbitals, use the `virtual` keyword. For
instance, to freeze the top 5 virtuals

```
 FREEZE virtual 5
```
### TRIALS: restart

Setting the keyword `trials` restart the calculation from the trials vector of a previous run.

```
trials 
```
### PRINT: output verbosity

The `PRINT` keyword changes the level of output verbosity. One may also request
some particular items in the table
below.


| Item                  | Print Level | Description                                        |
|-----------------------|-------------|----------------------------------------------------|
| "timings"             | high        | CPU and wall times spent in each step              |
| "trial vectors"       | high        | Trial CI vectors                                   |
| "initial guess"       | debug       | Initial guess CI vectors                           |
| "general information" | default     | General information                                |
| "xc information"      | default     | HF/DFT information                                 |
| "memory information"  | default     | Memory information                                 |
| "convergence"         | debug       | Convergence                                        |
| "subspace"            | debug       | Subspace representation of CI matrices             |
| "transform"           | debug       | MO to AO and AO to MO transformation of CI vectors |
| "diagonalization"     | debug       | Diagonalization of CI matrices                     |
| "iteration"           | default     | Davidson iteration update                          |
| "contract"            | debug       | Integral transition density contraction            |
| "ground state"        | default     | Final result for ground state                      |
| "excited state"       | low         | Final result for target excited state              |

Printable items in the TDDFT modules and their default print levels.


## Sample input

The following is a sample input for a spin-restricted TDDFT calculation
of singlet excitation energies for the water molecule at the
B3LYP/6-31G\*.
```
START h2o  
TITLE "B3LYP/6-31G* H2O"  
GEOMETRY 
 O     0.00000000     0.00000000     0.12982363  
 H     0.75933475     0.00000000    -0.46621158 
 H    -0.75933475     0.00000000    -0.46621158  
END  
BASIS  
 * library 6-31G* 
END  
DFT  
 XC B3LYP  
END  
TDDFT  
 RPA 
 NROOTS 20  
END  
TASK TDDFT ENERGY
```
To perform a spin-unrestricted TDHF/aug-cc-pVDZ calculation for the CO+
radical,
```
START co  
title "TDHF/aug-cc-pVDZ CO+"  
charge 1  
geometry  
 c  0.0  0.0  0.0  
 o  0.0  0.0  1.5
 symmetry c2v # enforcing abelian symmetry
end  
basis  
 * library aug-cc-pvdz  
end  
dft  
 xc hfexch  
 mult 2  
end
task dft optimize
tddft  
 rpa  
 nroots 5  
end  
task tddft energy
```
A geometry optimization followed by a frequency calculation for an
excited state is carried out for BF at the CIS/6-31G\* level in the
following sample input.
```
start bf  
title "CIS/6-31G* BF optimization frequencies"  
geometry  
 b 0.0 0.0 0.0  
 f 0.0 0.0 1.2
 symmetry c2v # enforcing abelian symmetry
end  
basis  
 * library 6-31g*  
end  
dft  
 xc hfexch  
end  
tddft  
 cis  
 nroots 3  
 notriplet  
 target 1  
  civecs  
  grad  
    root 1  
  end  
end  
task tddft optimize  
task tddft frequencies
```
TDDFT with an asymptotically corrected SVWN exchange-correlation
potential. Casida-Salahub scheme has been used with the shift value of
0.1837 a.u. supplied as an input parameter.
```
START tddft_ac_co  
GEOMETRY  
 O 0.0 0.0  0.0000  
 C 0.0 0.0  1.1283  
 symmetry c2v # enforcing abelian symmetry
END  
BASIS SPHERICAL  
 C library aug-cc-pVDZ  
 O library aug-cc-pVDZ  
END  
DFT  
 XC Slater VWN_5  
 CS00 0.1837  
END  
TDDFT  
 NROOTS 12  
END  
TASK TDDFT ENERGY
```
TDDFT with an asymptotically corrected B3LYP exchange-correlation
potential. Hirata-Zhan-Apra-Windus-Dixon scheme has been used (this is
only meaningful with B3LYP functional).
```
START tddft_ac_co  
GEOMETRY  
 O 0.0 0.0  0.0000  
 C 0.0 0.0  1.1283  
 symmetry c2v # enforcing abelian symmetry
END  
BASIS SPHERICAL  
 C library aug-cc-pVDZ  
 O library aug-cc-pVDZ  
END  
DFT  
 XC B3LYP  
 CS00  
END  
TDDFT  
 NROOTS 12  
END  
TASK TDDFT ENERGY
```
TDDFT for core states. The following example illustrates the usage of an
energy cutoff and energy and orbital windows.[^9]

```
echo  
start h2o_core  
memory 1000 mb  
geometry units au noautosym noautoz  
  O 0.00000000     0.00000000     0.22170860  
  H 0.00000000     1.43758081    -0.88575430  
  H 0.00000000    -1.43758081    -0.88575430  
end  
basis  
 O library 6-31g*  
 H library 6-31g*  
end  
dft  
 xc beckehandh  
 print "final vector analysis"  
end  
task dft  
tddft  
 ecut -10  
 nroots 5  
 notriplet  
 thresh 1d-03  
end  
task tddft  
tddft  
 ewin -20.0 -10.0  
 cis  
 nroots 5  
 notriplet  
 thresh 1d-03  
end  
task tddft  
dft  
 odft  
 mult 1  
 xc beckehandh  
 print "final vector analysis"  
end  
task dft  
tddft  
 alpha 1 1  
 beta 1 1  
 cis  
 nroots 10  
 notriplet  
 thresh 1d-03  
end  
task tddft
```
TDDFT optimization with LDA of Pyridine with the 6-31G basis[^10]

```
echo  
start tddftgrad_pyridine_opt  
title "TDDFT/LDA geometry optimization of Pyridine with 6-31G"  
geometry nocenter  
 N     0.00000000    0.00000000    1.41599295  
 C     0.00000000   -1.15372936    0.72067272  
 C     0.00000000    1.15372936    0.72067272  
 C     0.00000000   -1.20168790   -0.67391011  
 C     0.00000000    1.20168790   -0.67391011  
 C     0.00000000    0.00000000   -1.38406147  
 H     0.00000000   -2.07614628    1.31521089  
 H     0.00000000    2.07614628    1.31521089  
 H     0.00000000    2.16719803   -1.19243296  
 H     0.00000000   -2.16719803   -1.19243296  
 H     0.00000000    0.00000000   -2.48042299  
 symmetry c1  
end  
basis spherical  
* library "6-31G"  
end  
driver  
  clear  
  maxiter 100  
end  
dft  
  iterations 500  
  grid xfine  
end  
tddft  
  nroots 2  
  algorithm 1  
  notriplet  
  target 1  
  targetsym a  
  civecs  
  grad  
    root 1  
  end  
end  
task tddft optimize
```
TDDFT calculation followed by a calculation of the transition density
for a specific excited state using the DPLOT block
```
echo  
start h2o-td  
title h2o-td  

charge 0  
geometry units au
symmetry group c1  
 O    0.00000000000000      0.00000000000000      0.00000000000000  
 H    0.47043554760291      1.35028113274600      1.06035416576826  
 H   -1.74335410533480     -0.23369304784300      0.27360785442967  
end  
basis "ao basis"
* library "Ahlrichs pVDZ"
end  
dft  
 xc bhlyp  
 grid fine  
 direct  
 convergence energy 1d-5  
end  
tddft  
 rpa  
 nroots 5  
 thresh 1d-5  
 singlet  
 notriplet  
 civecs  
end  
task tddft energy  
dplot  
 civecs h2o-td.civecs_singlet  
 root 2  
 LimitXYZ  
  -3.74335 2.47044 50  
  -2.23369 3.35028 50  
  -2 3.06035 50  
   gaussian  
   output root-2.cube  
end  
task dplot
```
TDDFT protocol for calculating the valence-to-core (1s) X-ray emission spectrum [^11]


1.  Calculate the neutral ground state.   
2.  Calculate a full core hole (FCH) ionized state self-consistently, where the  
1s core orbital of the absorbing center is swapped with a virtual orbital. Use the   
maximum overlap constraint to prevent core hole collapse during the FCH calculation.
3.  Perform a LR-TDDFT calculation within the TDA is performed with the FCH ionized    
state as reference. 
4.  Final spectra is produced by taking the absolute value of the negative eigenvalues.

## Spectrum parser
A Python script is available for parsing NWChem output for TDDFT/vspec excitation energies, and optionally Lorentzian broadenening the spectra . The  nw_spectrum.py file can be found at [https://raw.githubusercontent.com/nwchemgit/nwchem/master/contrib/parsers/nw_spectrum.py](https://raw.githubusercontent.com/nwchemgit/nwchem/master/contrib/parsers/nw_spectrum.py)
```
Usage: nw_spectrum.py [options]

Reads NWChem output from stdin, parses for the linear response TDDFT or DFT
vspec excitations, and prints the absorption spectrum to stdout.  It will
optionally broaden peaks using a Lorentzian with FWHM of at least two
energy/wavelength spacings.  By default, it will automatically determine data
format (tddft or vspec) and generate a broadened spectrum in eV.

Example:

	nw_spectrum -b0.3 -p5000 -wnm < water.nwo > spectrum.dat

Create absorption spectrum in nm named "spectrum.dat" from the NWChem output
file "water.nwo" named spectrum.dat with peaks broadened by 0.3 eV and 5000
points in the spectrum.


Options:
  -h, --help            show this help message and exit
  -f FMT, --format=FMT  data file format: auto (default), tddft, vspec, dos
  -b WID, --broad=WID   broaden peaks (FWHM) by WID eV (default 0.1 eV)
  -n NUM, --nbin=NUM    number of eigenvalue bins for DOS calc (default 20)
  -p NUM, --points=NUM  create a spectrum with NUM points (default 2000)
  -w UNT, --units=UNT   units for frequency:  eV (default), au, nm
  -d STR, --delim=STR   use STR as output separator (four spaces default)
  -x, --extract         extract unbroadened roots; do not make spectrum
  -C, --clean           clean output; data only, no header or comments
  -c CHA, --comment=CHA
                        comment character for output ('#' default)
  -v, --verbose         echo warnings and progress to stderr
```
## References
///Footnotes Go Here///

[^1]: J. B. Foreman, M. Head-Gordon, J. A. Pople, and M. J. Frisch, *J. Phys. Chem.* **96**, 135 (1992), DOI:[10.1021/j100180a030](https://dx.doi.org/10.1021/j100180a030)    
[^2]: C. Jamorski, M. E. Casida, and D. R. Salahub, *J. Chem. Phys.* **104**, 5134 (1996), DOI:[10.1063/1.471140](https://dx.doi.org/10.1063/1.471140); R. Bauernschmitt and R. Ahlrichs, *Chem. Phys. Lett.* **256**, 454 (1996), DOI:[10.1016/0009-2614(96)00440-X](https://dx.doi.org/10.1016/0009-2614(96)00440-X); R. Bauernschmitt, M. Häser, O. Treutler, and R. Ahlrichs, *Chem. Phys. Lett.* **264**, 573 (1997), DOI:[10.1016/S0009-2614(96)01343-7](https://dx.doi.org/10.1016/S0009-2614(96)01343-7).  
[^3]: S. Hirata and M. Head-Gordon, *Chem. Phys. Lett.* **314**, 291 (1999).  DOI:[10.1016/S0009-2614(99)01149-5](https://dx.doi.org/10.1016/S0009-2614(99)01149-5)     
[^4]: R. van Leeuwen and E. J. Baerends, *Phys. Rev. A* **49**,  2421 (1994), DOI:[10.1103/PhysRevA.49.2421](https://dx.doi.org/10.1103/PhysRevA.49.2421)  
[^5]: M. E. Casida, C. Jamorski, K. C. Casida, and D. R. Salahub, *J. Chem. Phys.* **108**, 4439 (1998), DOI:[10.1063/1.475855](https://dx.doi.org/10.1063/1.475855)
[^6]: S. Hirata, C.-G. Zhan, E. Aprà, T. L. Windus, and D. A. Dixon, *J. Phys. Chem. A* **107**, 10154 (2003).  DOI:[10.1021/jp035667x](https://dx.doi.org/10.1021/jp035667x)  
[^8]: D. J. Tozer and N. C. Handy, *J. Chem. Phys.* **109**, 10180 (1998), DOI:[10.1063/1.477711](https://dx.doi.org/10.1063/1.477711)  
[^9]: K. Lopata, B. E. Van Kuiken, M. Khalil, N. Govind, "Linear-Response and Real-Time Time-Dependent Density Functional Theory Studies of Core-Level Near-Edge X-Ray Absorption", *J. Chem. Theory Comput.*, 2012, **8** (9), pp 3284–3292, DOI:[10.1021/ct3005613](https://dx.doi.org/10.1021/ct3005613)  
[^10]: D. W. Silverstein, N. Govind,  H. J. J. van Dam, L. Jensen, "Simulating One-Photon Absorption and Resonance Raman Scattering Spectra Using Analytical Excited State Energy Gradients within Time-Dependent Density Functional Theory" *J. Chem. Theory Comput.*, 2013, **9** (12), pp 5490–5503, DOI:[10.1021/ct4007772](https://dx.doi.org/10.1021/ct4007772)  
[^11]: Y. Zhang, S. Mukamel, M. Khalil, N. Govind, "Simulating Valence-to-Core X-ray Emission Spectroscopy of Transition Metal", *J. Chem. Theory Comput*., 2015, **11** (12), pp 5804–5809, DOI:[10.1021/acs.jctc.5b00763](https://dx.doi.org/10.1021/acs.jctc.5b00763)  
