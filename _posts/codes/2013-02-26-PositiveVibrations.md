---
priority: 0.17
title: "PhOnonS ITeratIVE VIBRATIONS"
excerpt: "Vibrational and electron-phonon coupling analysis via first-principles"
categories: coding main
background-image: PositiveVibrations_back.jpg
options: minihead, notitle
code-logo: PositiveVibrations.jpg
code-repo: github
code-url: https://github.com/brandimarte/vibrations
code-author: <u>Pedro Brandimarte</u> and A. R. Rocha
code-lictype: open
code-license: GPL-3.0
lang:
  - C
  - shell script
tags:
  - vibrational analysis
  - modes and frequencies
  - electron-phonon coupling
  - density functional theory
---

#### Introduction

This is a practical description of the code **PhOnonS ITeratIVE VIBRATIONS**, a tool for computing materials' vibrational modes and frequencies and electron-phonon coupling matrices, from first principles calculations.
It is based on code SIESTA [1, 2], a density functional theory (DFT) implementation.

With the purpose of calculating the vibrational modes and frequencies, the **POSITIVE VIBRATIONS** code uses the force constant (FC) matrix as calculated by SIESTA.
In order to minimize the effects arising from the use of a discrete grid on FC calculation (such as the so-called egg-box effect [1]), the **POSITIVE VIBRATIONS** code applies some corrections by enforcing momentum conservation and the FC matrix symmetrization.

To compute the electron-phonon coupling matrices some modifications were performed on SIESTA code, following the methodology described in [3].

The **POSITIVE VIBRATIONS** source code is stored as a git repository and can be easily downloaded with the line command:

```bash
git clone https://github.com/brandimarte/vibrations.git
```

___

#### Compilation

The compilation of the program is done using a `Makefile`.
Some examples of `Makefile` for different architectures are provided with the code.
Basically, the user should set the *C* compiler and its options, and give the path for LAPACK libraries (Linear Algebra PACKage [4]).

___

#### Execution Steps and Options

##### Vibrational Modes and Frequencies

The first step is to execute a FC calculation with SIESTA code (for a detailed description on how to perform a force constant calculation in SIESTA and the required flags in input file, refer to the SIESTA user's guide [2]).
The usual FC calculation is enough if one is interested only in vibrational frequencies and modes, which then can be calculated with the following command:

```bash
vibrations ~/PathToFCdir input.fdf onlyPh
```

The first string (`vibrations`) is the **POSITIVE VIBRATIONS** executable.
The second (`~/PathToFCdir`) is the path to the directory where the FC calculation was performed.
The third argument (`input.fdf`) is the SIESTA fdf input file used in the FC calculation (refer to SIESTA user's guide for details) and the last one (`onlyPh`) tells to the code that one is only looking for the vibrational frequencies and modes (i.e. the electron-phonon coupling computation will be neglected).

During the **POSITIVE VIBRATIONS** code execution the calculation progress is printed on screen.
The computed vibrational frequencies are enumerated in ascending order and the corresponding modes follow the same enumeration (i.e. the last mode corresponds to the highest frequency).

One possibility to accelerate the FC calculation on SIESTA is to split over the atoms which are being moved, by performing a FC computation on each atom independently.
An example of script for that (`runFC.sh`) is provided with the code.
For each dynamic atom *i* (those who will be displaced), the script creates the folder `FCi` and submit for execution.
In this case the total force constant matrix will be split over the individual directories and, in order to inform the **POSITIVE VIBRATIONS** code about that, one should include an additional `splitFC` option:

```bash
vibrations ~/PathToFCdir input.fdf onlyPh splitFC
```

The **POSITIVE VIBRATIONS** code can generate files for visualizing the vibrational modes according to *Jmol* viewer format [5].
For that, the file `SystemLabel.xyz` with non-displaced coordinates must be present on the directory where the FC calculation was performed.
For each mode *j* a file `SystemLabelJMOLj.xyz` is generated, containing the system's coordinates and the mode eigenvector.
When opening this file with *Jmol* viewer one can visualize the corresponding vectors and an animation of the vibrational mode.

##### Electron-Phonon Coupling Matrices

In order to compute the electron-phonon coupling matrices one should include the following flag in the `input.fdf` file for FC calculation:

> **FCwriteHS** (*Boolean*): When `FCwriteHS` is set to `T`, the modified SIESTA will write: a file `SystemLabel_d.gHS` for each displacement *d* containing the Hamiltonian and the overlap matrices, a file `SystemLabel.ef` containing the FC step and the corresponding computed Fermi energy, and a file `SystemLabel.orb` containing the last orbital index of each atom of the entire system (starting from 0).
> 
> *Default value:* `F`

The generated files will be used by the **POSITIVE VIBRATIONS** code for computing the derivative of the electronic Hamiltonian with respect to the atoms displacements, which is needed when calculating the electron-phonon coupling [3].

In addition to the FC calculation, one needs to execute the modified SIESTA one more time for computing matrix elements like &#139;*i*'&#124;*k*&#155; and &#139;*l*&#124;*j*'&#155;, where &#139;*i*'&#124; represents the variation of the basis orbital with respect to the displacement of an atom in a given direction.
These elements will be used by the **POSITIVE VIBRATIONS** code for correcting the electronic Hamiltonian derivatives.
They can be obtained from the overlap matrix resulting from a SIESTA execution for a duplicated system, where the duplicated atoms are displaced in one of the directions (totaling six different runs: *-x*, *+x*, *-y*, *+y*, *-z* and *+z*).

To automate this procedure, a script `runOS.sh` was develop.
It receives the SIESTA fdf input file (`input.fdf`), sets the flag `FCwriteHS` to `F` and include the following flag as `T`:

> **FConlyS** (*Boolean*): When `FConlyS` is set to `T`, the modified SIESTA will write the overlap matrix (which happens at the beginning of the program) at the file `SystemLabel.oS` and finishes the execution.
> 
> *Default value:* `F`

The script `runOS.sh` then duplicates the system and displaces the duplicated atoms in each direction.
At each displacement *d* the system label is modified to `SystemLabel_d` (where *d* goes from 1 to 6 meaning *-x*, *+x*, *-y*, *+y*, *-z* and *+z*, respectively) and the modified SIESTA is executed.
This procedure is very fast, since the overlap matrix is built at the beginning of SIESTA and the execution is finished just after that.

Once this is done the electron-phonon coupling matrices can be computed (together with the vibrational frequencies and modes) with **POSITIVE VIBRATIONS** code with the following command:

```bash
vibrations ~/PathToFCdir input.fdf full
```

Here `vibrations` is the **POSITIVE VIBRATIONS** executable, `~/PathToFCdir` is the path to the directory where the FC calculation was performed, `input.fdf` is the SIESTA fdf input file used in the FC calculation (refer to SIESTA user's guide for details) and `full` tells to the code that the electron-phonon coupling matrices will be computed besides the vibrational frequencies and modes.
Note that the overlap matrix files described above (`SystemLabel_d.oS`) should be in the FC folder.

If the FC calculation was split over the displaced atoms (see **Vibrational Modes and Frequencies** above) one should include an additional `splitFC` option:

```bash
vibrations ~/PathToFCdir input.fdf full splitFC
```

In addition to the output files described in **Vibrational Modes and Frequencies** above, the **POSITIVE VIBRATIONS** code generates the following file:

> **SystemLabel.Meph**: The first line contains integer numbers separated by white spaces corresponding to the following information: the number of spin components, the number of dynamic atoms (those that were displaced on FC calculation), the number of dynamic orbitals, the first dynamic orbital index and the last dynamic orbital index.
> The next line contains the vibrational frequencies (in eV) in descending order separated by white spaces (if it was considered *n* atoms in the vibrational analysis, then there should have *3 n* frequencies).
> The following lines contains the electron-phonon coupling matrices (in eV) in the same order as the frequencies (for each frequency there must have one electron-phonon coupling matrix).

___

#### Problem Handling

The users are encouraged to report problems and bugs to the **POSITIVE VIBRATIONS**'s developers at <a target="_blank" href="https://github.com/brandimarte/vibrations">github</a>.
Patches and fixes will be uploaded to the web-site <a target="_blank" href="https://github.com/brandimarte/vibrations">https://github.com/brandimarte/vibrations</a>.

___

#### Acknowledgements

PBM thanks CNPq (National Council of Technological and Scientific Development) for financial support (grant 142792/2010-1).

___

#### References

[1] <a target="_blank" href="https://doi.org/10.1088/0953-8984/14/11/302">J. M. Soler, E. Artacho, J. D. Gale, A. García, J. Junquera, P. Ordejón and D. Sánchez-Portal, *J. Phys. Cond. Mat.* **14**, 2745-2779 (2002).</a>

[2] <a target="_blank" href="http://departments.icmab.es/leem/siesta/">http://departments.icmab.es/leem/siesta/</a>

[3] <a target="_blank" href="https://doi.org/10.1103/physrevb.75.205413">T. Frederiksen, M. Paulsson, M. Brandbyge and A.-P. Jauho, *Phys. Rev. B* **75**, 205413 (2007).</a>

[4] LAPACK - Linear Algebra PACKage. <a target="_blank" href="http://www.netlib.org/lapack/">http://www.netlib.org/lapack/</a>

[5] Jmol: an open-source Java viewer for chemical structures in 3D. <a target="_blank" href="http://jmol.sourceforge.net">http://jmol.sourceforge.net</a>
