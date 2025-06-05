# MP2

## Overview 

There are (at least) three algorithms within NWChem that compute the
Møller-Plesset (or many-body) perturbation theory second-order
correction[@moller1934] to the Hartree-Fock energy (MP2). They vary in
capability, the size of system that can be treated and use of other
approximations

  - Semi-direct -- this is recommended for most large applications (up
    to about 2800 basis functions), especially on the IBM SP and other
    machines with significant disk I/O capability. Partially transformed
    integrals are stored on disk, multi-passing as necessary. RHF and
    UHF references may be treated including computation of analytic
    derivatives. This is selected by specifying mp2 on the task
    directive, e.g.

```
TASK MP2
```

  - Fully-direct[@wong1996] -- this is of utility if only limited I/O
    resources are available (up to about 2800 functions). Only RHF
    references and energies are available. This is selected by
    specifying `direct_mp2` on the task directive, e.g.

```
TASK DIRECT_MP2
```

  - Resolution of the identity (RI) approximation MP2 (RI-MP2)[@bernholdt1996] --
    this uses the RI approximation and is therefore only exact in the
    limit of a complete fitting basis. However, with some care, high
    accuracy may be obtained with relatively modest fitting basis sets.
    An RI-MP2 calculation can cost over 40 times less than the
    corresponding exact MP2 calculation. RHF and UHF references with
    only energies are available. This is selected by specifying rimp2 on
    the task directive, e.g.,

```
TASK RIMP2
```

All three MP2 tasks share the same input block.
```
 MP2  
   [FREEZE [[core] (atomic || <integer nfzc default 0>)] \  
           [virtual <integer nfzv default 0>]]  
   [TIGHT]  
   [PRINT]  
   [NOPRINT]  
   [VECTORS <string filename default scf-output-vectors> \  
     [swap [(alpha||beta)] <integer pair-list>] ]  
   [RIAPPROX <string riapprox default V>]  
   [FILE3C <string filename default $file_prefix$.mo3cint>  
   [SCRATCHDISK <integer>] 
 END
```
## FREEZE: Freezing orbitals

All MP2 modules support frozen core orbitals, however, only the direct
MP2 and RI-MP2 modules support frozen virtual orbitals.

By default, no orbitals are frozen. The atomic keyword causes orbitals
to be frozen according to the rules in the table below. Note that no
orbitals are frozen on atoms on which the nuclear charge has been
modified either by the user or due to the presence of an ECP. The actual
input would be

```
 freeze atomic
```

For example, in a calculation on Si(OH)<sub>2</sub>, by default the lowest
seven orbitals would be frozen (the oxygen 1*s*, and the silicon
1*s*, 2*s* and 2*p*).

### Number of orbitals considered "core" in the "freeze by atoms" algorithm

| Period | Elements | Core Orbitals                                                                              | Number of Core |
| ------ | -------- | ----------------------------------- | -------------- |
| 0      | H - He   | \-                                  | 0              |
| 1      | Li - Ne  | *1s*                                | 1              |
| 2      | Na - Ar  | *1s2s2p*                            | 5              |
| 3      | K - Kr   | *1s2s2p3s3p*                        | 9              |
| 4      | Rb - Xe  | *1s2s2p3s3p4s3d4p*                  | 18             |
| 5      | Cs - Rn  | *1s2s2p3s3p4s3d4p5s4d5p*            | 27             |
| 6      | Fr - Lr  | *1s2s2p3s3p4s3d4p5s4d5p6s4f5d6p*    | 43             |




**Caution:** The rule for freezing orbitals "by atoms" are rather
unsophisticated since the number of orbitals to be frozen is computed from
the table above by summing the number of core orbitals in each atom
present. Therefore, the corresponding number of lowest-energy orbitals are frozen.
If for some reason the actual core orbitals are not the lowest lying,
then correct results will not be obtained. It is likely that
special attention should be paid to systems including third-
and higher- period atoms.

The user may also specify the number of orbitals to be frozen by atom.
Following the  Si(OH)<sub>2</sub> example, the user could specify

```
freeze atomic O 1 Si 3
```

In this case only the lowest four orbitals would be frozen. If the user
does not specify the orbitals by atom, the rules default  to values reported in the [Table above](#number-of-orbitals-considered-core-in-the-freeze-by-atoms-algorithm).

Caution: The system does not check for a valid number of orbitals per
atom. If the user specifies to freeze more orbitals then are available
for the atom, the system will not catch the error. The user must specify
a logical number of orbitals to be frozen for the atom.

The FREEZE directive may also be used to specify the number of core
orbitals to freeze. For instance, to freeze the first 10 orbitals

```
 freeze 10
```

or equivalently, using the optional keyword core

```
 freeze core 10
```

Again, note that if the 10 orbitals to be frozen do not correspond to
the first 10 orbitals, then the swap keyword of the VECTORS directive
must be used to order the input orbitals correctly ([MO
vectors](#vectors-mo-vectors)).

To freeze the highest virtual orbitals, use the virtual keyword. For
instance, to freeze the top 5 virtuals

```
 freeze virtual 5
```

Again, note that this only works for the direct-MP2 and RI-MP2 energy
codes.

## TIGHT: Increased precision

The TIGHT directive can be used to increase the precision in the MP2
energy and gradients.

By default the MP2 gradient package should compute energies accurate to
better than a micro-Hartree, and gradients accurate to about five
decimal places (atomic units). However, if there is significant linear
dependence in the basis set the precision might not be this good. Also,
for computing very accurate geometries or numerical frequencies, greater
precision may be desirable.

This option increases the precision to which both the SCF (from
10<sup>-6</sup> to 10<sup>-8</sup> and CPHF 10<sup>-4</sup> to $10<sup>-6</sup>
are solved, and also tightens thresholds for computation of the AO and
MO integrals (from 10<sup>-9</sup> to 10<sup>-11</sup> within the MP2 code.

## SCRATCHDISK: Limiting I/O usage

This directive - used only in the semi-direct algorithm - allows to
limit the per process disk usage. Mandatory argument for this keyword is
the maximum number of MBytes. For example, the following input line

```
 scratchdisk 512
```

puts an upper limit of 512 MBytes to the semi-direct MP2 usage of disk
(again, on a per process base).

## PRINT and NOPRINT

The standard print control options are recognized. The list of
recognized names are given in the table below.  

| Item                    | Print Level | Description                                |
| ----------------------- | ----------- | ------------------------------------------ |
| **RI-MP2**              |             |                                            |
| "2/3 ints"              | debug       | Partial 3-center integrals                 |
| "3c ints"               | debug       | MO 3-center integrals                      |
| "4c ints b"             | debug       | "B" matrix with approx. 4c integrals       |
| "4c ints"               | debug       | Approximate 4-center integrals             |
| "amplitudes"            | debug       | "B" matrix with denominators               |
| "basis"                 | high        |                                            |
| "fit xf"                | debug       | Transformation for fitting basis           |
| "geombas"               | debug       | Detailed basis map info                    |
| "geometry"              | high        |                                            |
| "information"           | low         | General information about calc.            |
| "integral i/o"          | high        | File size information                      |
| "mo ints"               | debug       |                                            |
| "pair energies"         | debug       | (working only in `direct_mp2`)             |
| "partial pair energies" | debug       | Pair energy matrix each time it is updated |
| "progress reports"      | default     | Report completion of time-consuming steps  |
| "reference"             | high        | Details about reference wavefunction       |
| "warnings"              | low         | Non-fatal warnings                         |

## VECTORS: MO vectors

All of the (supported) MP2 modules require use of converged canonical
SCF (RHF or UHF) orbitals for correct results. The vectors are by
default obtained from the preceding SCF calculation, but it is possible
to specify a different source using the VECTORS directive. For instance,
to obtain vectors from the file /tmp/h2o.movecs, use the directive

```
 vectors /tmp/h2o.movecs
```

As noted above ([FREEZE](#freeze-freezing-orbitals)) if
the SCF orbitals are not in the correct order, it is necessary to
permute the input orbitals using the swap keyword of the VECTORS
directive. For instance, if it is desired to freeze a total six orbitals
corresponding to the SCF orbitals 1-5, and 7, it is necessary to swap
orbital 7 into the 6th position. This is accomplished by

```
 vectors swap 6 7
```

The swap capability is examined in more detail in [Input/output of MO
vectors](Hartree-Fock-Theory-for-Molecules#vectors-inputoutput-of-mo-vectors).

## RI-MP2 fitting basis

The RI-MP2 method requires a fitting basis, which must be specified with
the name "ri-mp2 basis" (see [Basis](Basis)). For instance,

```
 basis "ri-mp2 basis"
   O s; 10000.0 1
   O s;  1000.0 1
   O s;   100.0 1
   ...
 end
```

Alternatively, using a standard capability of basis sets
([Basis](Basis)) another named basis may be associated with
the fitting basis. For instance, the following input specifies a basis
with the name "small fitting basis" and then defines this to be the
"ri-mp2 basis".

```
 basis "small fitting basis"
   H s; 10    1
   H s;  3    1
   H s;  1    1
   H s;  0.1  1
   H s;  0.01 1
 end
```

```
 set "ri-mp2 basis" "small fitting basis"
```

## FILE3C: RI-MP2 3-center integral filename

The default name for the file used to store the transformed 3-center
integrals is "file_prefix.mo3cint" in the scratch directory. This may
be overridden using the FILE3C directive. For instance, to specify the
file /scratch/h2o.3c, use this directive

```
 file3c /scratch/h2o.3c
```

## RIAPPROX: RI-MP2 Approximation

The type of RI approximation used in the RI-MP2 calculation is
controlled by means of the RIAPPROX directive. The two possible values
are V and SVS (case sensitive), which correspond to the approximations
with the same names described by Vahtras et al.[@vahtras1993]. The default is V.

## Advanced options for RI-MP2

These options, which functioned at the time of writing, are not
currently supported.

### Control of linear dependence

Construction of the RI fit requires the inversion of a matrix of fitting
basis integrals which is carried out via diagonalization. If the fitting
basis includes near linear dependencies, there will be small eigenvalues
which can ultimately lead to non-physical RI-MP2 correlation energies.
Eigenvectors of the fitting matrix are discarded if the corresponding
eigenvalue is less than `min eval` which defaults to 10<sup>-8</sup>. This
parameter may be changed by setting the a parameter in the database. For
instance, to set it to 10<sup>-10</sup>

```
 set "mp2:fit min eval" 1e-10
```

### Reference Spin Mapping for RI-MP2 Calculations

The user has the option of specifying that the RI-MP2 calculations are
to be done with variations of the SCF reference wavefunction. This is
accomplished with a SET directive of the form,

```
 set "mp2:reference spin mapping" <integer array default 0>
```

Each element specified for array is the SCF spin case to be used for the
corresponding spin case of the correlated calculation. The number of
elements set determines the overall type of correlated calculation to be
performed. The default is to use the unadulterated SCF reference
wavefunction.

For example, to perform a spin-unrestricted calculation (two elements)
using the alpha spin orbitals (spin case 1) from the reference for both
of the correlated reference spin cases, the SET directive would be as
follows,

```
 set "mp2:reference spin mapping" 1 1
```

The SCF calculation to produce the reference wavefunction could be
either RHF or UHF in this case.

The SET directive for a similar case, but this time using the beta-spin
SCF orbitals for both correlated spin cases, is as follows,

```
 set "mp2:reference spin mapping" 2 2
```

The SCF reference calculation must be UHF in this case.

The SET directive for a spin-restricted calculation (one element) from
the beta-spin SCF orbitals using this option is as follows,

```
 set "mp2:reference spin mapping" 2
```

The SET directive for a spin-unrestricted calculation with the spins
flipped from the original SCF reference wavefunction is as follows,

```
 set "mp2:reference spin mapping" 2 1
```

### Batch Sizes for the RI-MP2 Calculation

The user can control the size of each batch in the transformation and
energy evaluation in the MP2 calculation, and consequently the memory
requirements and number of passes required. This is done using two SET
directives of the following form,

```
 set "mp2:transformation batch size" <integer size default -1>
 set "mp2:energy batch size" <integer isize jsize default -1 -1>
```

The default is for the code to determine the batch size based on the
available memory. Should there be problems with the program-determined
batch sizes, these variables allow the user to override them. The
program will always use the smaller of the user's value of these entries
and the internally computed batch size.

The transformation batch size computed in the code is the number of
occupied orbitals in the *(occ vir|fit)* three-center integrals to be
produced at a time. If this entry is less than the number of occupied
orbitals in the system, the transformation will require multiple passes
through the two-electron integrals. The memory requirements of this
stage are two global arrays of dimension *`batch size` x vir x fit* with
the "fit" dimension distributed across all processors (on shell-block
boundaries). The compromise here is memory space versus multiple
integral evaluations.

The energy evaluation batch sizes are computed in the code from the
number of occupied orbitals in the two sets of three-center integrals to
be multiplied together to produce a matrix of approximate four-center
integrals. Two blocks of integrals of dimension *(`batch isize` x vir)*
and *(`batch jsize` x vir)* by fit are read in from disk and multiplied
together to produce *`batch isize` `batch jsize` vir^2* approximate
integrals. The compromise here is performance of the distributed matrix
multiplication (which requires large matrices) versus memory space.

### Energy Memory Allocation Mode: RI-MP2 Calculation

The user must choose a strategy for the memory allocation in the energy
evaluation phase of the RI-MP2 calculation, either by minimizing the
amount of I/O, or minimizing the amount of computation. This can be
accomplished using a SET directive of the form,

```
 set "mp2:energy mem minimize" <string mem_opt default I>
```

A value of I entered for the string `mem_opt` means that a strategy to
minimize I/O will be employed. A value of C tells the code to use a
strategy that minimizes computation.

When the option to minimize I/O is selected, the block sizes are made as
large as possible so that the total number of passes through the
integral files is as small as possible. When the option to minimize
computation is selected, the blocks are chosen as close to square as
possible so that permutational symmetry in the energy evaluation can be
used most effectively.

### Local Memory Usage in Three-Center Transformation

For most applications, the code will be able to size the blocks without
help from the user. Therefore, it is unlikely that users will have any
reason to specify values for these entries except when doing very
particular performance measurements.

The size of `xf3ci:AO 1 batch size` is the most important of the three, in
terms of the effect on performance.

Local memory usage in the first two steps of the transformation is
controlled in the RI-MP2 calculation using the following SET directives,

```
 set "xf3ci:AO 1 batch size" <integer max>
 set "xf3ci:AO 2 batch size" <integer max>
 set "xf3ci:fit batch size" <integer max>
```

The size of the local arrays determines the sizes of the two matrix
multiplications. These entries set limits on the size of blocks to be
used in each index. The listing above is in order of importance of the
parameters to performance, with xf3ci:AO 1 batch size being most
important.

Note that these entries are only upper bounds and that the program will
size the blocks according to what it determines as the best usage of the
available local memory. The absolute maximum for a block size is the
number of functions in the AO basis, or the number of fitting basis
functions on a node. The absolute minimum value for block size is the
size of the largest shell in the appropriate basis. Batch size entries
specified for max that are larger than these limits are automatically
reset to an appropriate value.

## One-electron properties and natural orbitals

If an MP2 energy gradient is computed, all contributions are available
to form the MP2 linear-response density. This is the density that when
contracted with any spin-free, one-electron operator yields the
associated property defined as the derivative of the energy. Thus, the
reported MP2 dipole moment is the derivative of the energy w.r.t. an
external electric field and is not the expectation value of the operator
over the wavefunction. It has been shown that evaluating the MP2 density
through a derivative provides more accurate results, presumably because
this matches the way experiments probe the electron density more
closely[raghavachari1981][@diercksen1981][@rice1985][@wiberg1992].

Only dipole moments are printed by the MP2 gradient code, but natural
orbitals are produced and stored in the permanent directory with a file
extension of ".mp2nos". These may be fed into the [property
package](Properties) to compute more general properties as in the following example.
```
start h2o
geometry
   O        2.15950        0.88132        0.00000
   H        3.12950        0.88132        0.00000
   H        1.83617        0.89369       -0.91444
end

basis spherical
  * library aug-cc-pVDZ
end

mp2
  freeze atomic
end

task mp2 gradient

property
  vectors  h2o.mp2nos
  mulliken
end

task mp2 property
```
Note that the MP2 linear response density matrix is not necessarily positive
definite so it is not unusual to see a few small negative natural
orbital occupation numbers. Significant negative occupation numbers have
been argued to be a sign that the system might be near degenerate[@gordon1999].

## SCS-MP2: Spin-Component Scaled MP2

Each MP2 output contains the calculation of the SCS-MP2 correlation
energies as suggested by S.Grimme[@grimme2003]

The SCS keyword is only required for gradients calculations:
```
 MP2  
   [SCS]  
 END
```
Scaling factors for the two components (parallel and opposite spin) can
be defined by using the keywords FSS (same spin factor) and FOS
(opposite spin factor):
```
mp2  
   scs  
   fss   1.13  
   fos   0.56  
end
```
Default values are FSS=0.333333333, FOS=1.2 for MP2, and FSS=1.13, FOS=1.27 for
CCSD.

## References
///Footnotes Go Here///

