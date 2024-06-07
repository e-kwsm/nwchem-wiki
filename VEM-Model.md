# VEM (Vertical Excitation or Emission) Model

## Overview

The VEM is a model for calculating the vertical excitation (absorption)
or vertical emission (fluorescence) energy in solution according to a
two-time-scale model of solvent polarization. The model is described in reference[@marenich2011].  

The current implementation is based on the VEM(d,RD) algorithm as
described in the above paper. The method is available only at the [TDDFT](Excited-State-Calculations.md)
level of theory, including both full-linear response TDDFT (sometimes
called LR-TDDFT or regular TDDFT) and the Tamm–Dancoff approximation,
[TDDFT-TDA](Excited-State-Calculations.md#cis-and-rpa-the-tamm-dancoff-approximation)
(sometimes just called TDA). The configuration interaction
singles (CIS) wave function method can also be used along with VEM by
considering CIS to be a special case of TDDFT-TDA.

The abbreviation VEM originally referred to the vertical excitation
model of reference[@li2000],
but the current implementation of VEM extends to both excitation and
emission calculations in solution, and the E in VEM now stands for
excitation/emission. Furthermore, the current version of VEM (based on
the Marenich et al. paper[@marenich2011]) does
not reduce to the original VEM of Li et al., but is improved as
described in reference[@marenich2011].

The VEM model is based on a nonequilibrium dielectric-continuum approach
in terms of two-time-scale solvent response. The solvent's
bulk-electrostatic polarization is described by using the reaction field
partitioned into slow and fast components, and only the fast component
is self-consistently (iteratively) equilibrated with the charge density
of the solute molecule in its final state. During the VEM calculation,
the slow component is kept in equilibrium with the initial state's
solute charge density but not with the final state's one. In the case of
vertical absorption the initial state is the ground electronic state of
the solute molecule in solution and the final state is an excited
electronic state in solution (and vice versa in the case of an emission
spectrum). Both the ground- and excited-state calculations involve an
integration of the nonhomogeneous-dielectric Poisson equation for bulk
electrostatics in terms of the COSMO model as implemented in NWChem with
the modified COSMO scaling factor (`iscren 0`) and by using the SMD
intrinsic atomic Coulomb radii (by default; see the section of the
manual describing SMD). The excited-state electron density is calculated
using the Z-Vector "relaxed density" approach.

The VEM excitation or emission energy includes only a bulk-electrostatic
contribution without any cavity–dispersion–solvent-structure (CDS)
contributions (such contributions are used in SMD ground-state
calculations as described in the SMD section of this manual, but are not
used in VEM calculations). When one considers solvatochromic shifts, the
main contributions beyond bulk electrostatics are solute–solvent
dispersion interactions, hydrogen bonding (the latter is most important
in protic solvents), and perhaps charge transfer between the solute and
the solvent. To explicitly account for solute–solvent charge transfer
and hydrogen bonding, the user can run a VEM calculation on a
supersolute that involves a solute–solvent molecular cluster with one or
a few solvent molecules added explicitly to a bare solute. The
solute–solvent dispersion contribution to the solvatochromic shift, if
desired, can be estimated by the solvation model with state-specific
polarizability (SMSSP) described in reference[@marenich2013].  

In this case, the user needs to provide values of ground- and
excited-state spherically averaged molecular polarizabilities of the
solvent.

## Syntax

The VEM-specific input options are as
follows:

### DO_COSMO_VEM:  

```
 do_cosmo_vem <integer do_cosmo_vem default 0>
```

The `do_cosmo_vem` can be set to the following values:  
- `0` (do not do any VEM calculation even if the task tddft gradient line is present; default).  
- `1` (do a nonequilibrium VEM excitation energy calculation;  
in this case the `task tddft gradient` line should be present, too)    
- `2` (do an equilibrium VEM excitation energy calculation followed by  
a nonequilibrium emission energy calculation;  
`task tddft gradient` line should be present)    

### VEM Solvent

The VEM solvent (which is water by default) can be specified by using
the solvent keyword described in the [SMD section of this manual](SMD-Model#smd-solvent-keyword) or by
specifying the VEM solvent descriptors such as

`dielec (real input)`  
`static dielectric constant`  

`dielecinf (real input)`    
optical dielectric constant which is set (by default) to the squared value of the solvent's index of refraction   
(see the keyword `soln`, but note 
that if the solvent is specified with the solvent keyword,  
 the refractive index is set by the program without needing the user to supply it.)   

Solvent descriptors set by the program are based on the Minnesota
Solvent Descriptor Database[@winget1999]:


If the option `do_cosmo_vem 1` or `do_cosmo_vem 2` is specified the
program will run VEM ground- and excited-state bulk-electrostatic
calculations using the COSMO algorithm with the SMD Coulomb radii by
default. If the user wants to use the default COSMO radii in such
calculations (this is not recommended) the option `do_cosmo_smd .false.` should be specified.

## SMSSP estimate of the solute–solvent dispersion contribution

If the SMSSP estimate of a solute–solvent dispersion contribution to the
solvatochromic shift is desired, the following options should be
used:

`polgs_cosmo_vem (real input)`  
user-provided value of the spherically-averaged molecular polarizability of the solute in the ground state (in Å<sup>3</sup>)

`poles_cosmo_vem (real input)`  
user-provided value of the spherically-averaged molecular polarizability of the solute in an exited state of interest (in Å<sup>3</sup>)

## Examples

An example of the VEM input file is provided
below.
```
echo  
title  'VEM/TDDFT-B3LYP/6-311+G(d) vertical excitation energy + SMSSP for formaldehyde in methanol'  
start  
geometry nocenter  
 O    0.0000000000    0.0000000000    0.6743110000  
 C    0.0000000000    0.0000000000   -0.5278530000  
 H    0.0000000000    0.9370330000   -1.1136860000  
 H    0.0000000000   -0.9370330000   -1.1136860000  
symmetry c1  
end  
basis  
* library 6-311+G*  
end  
dft  
 XC b3lyp  
end  
cosmo  
 do_cosmo_smd true  
 do_cosmo_vem 1  
 solvent methanol  
 polgs_cosmo_vem 2.429  
 poles_cosmo_vem 3.208  
end  
tddft  
 nroots 10  
 target 1  
 singlet  
 notriplet  
 algorithm 1  
 civecs  
end  
grad  
  root 1  
  solve_thresh 1d-05  
end  
task tddft gradient
```

## References
///Footnotes Go Here///

