# TBSCI-202603

This repository provides executable binaries of the TBSCI eigensolver used in the associated manuscript. 
Bitstring preparation is performed using an external SCI tool as described in the manuscript.

Representative input and output data for selected benchmark systems are distributed via GitHub Releases.

---

## Contents

- bin/
  Prebuilt executables and corresponding makefiles:
  - `Fugaku_exe.out`, `Intel_exe.out`
  - `makefile_Fugaku`, `makefile_Intel`

- README.md
  This documentation.

Large input/output data files are NOT stored in the repository and must be downloaded separately from 
the Releases page.

---

## Data availability

Representative input and output data accompanying the manuscript are provided as compressed files 
through GitHub Releases:

- FCI_results.tar.gz
- TBSCI_results.tar.gz

Release page:
https://github.com/enhua-xu/TBSCI-202603/releases


---

## File conventions

Each test-case directory contains the core set of files required to reproduce the reported calculations.

- *.FCIDUMP

  One- and two-electron integrals in FCIDUMP format.
  Core orbitals are included in the FCIDUMP files; freezing is applied at runtime via the `NFO` parameter
  specified in the corresponding `*.input` file.

  All FCIDUMP files are generated using PySCF. Accordingly, the character tables, irreducible 
  representations, and their ordering follow the symmetry conventions adopted by PySCF.

- *.charT

  Character table for the point group used in the calculation.
  - N2 and Cr2: D2h (8 irreducible representations)
  - CN: C2v (4 irreducible representations)

- *.input

  Key parameters used in the calculations (parameter names are case-sensitive):

  - `NFO`: number of frozen core orbitals. `Default: 0`.
  - `NFV`: number of frozen virtual orbitals. `Default: 0`.
  - `Nirp`: number of irreducible representations (irreps) of the point group (e.g., 8 for D2h, 4 for C2v).  
    `Default: 1`.
  - `Target_irp`: target irreducible representation (symmetry sector) for the ground-state calculation.  
    `Default: 0`.
  - `Conv`: energy convergence threshold. `Default: 1.0D-7`.
  - `BALANCE`: load-balancing strategy for distributing segments across MPI processes. `Default: 0`.
    - `0`: memory-average distribution
    - `1`/`2`: overall average distribution (`2` recommended)
 
- *.strA and *.strB

  Alpha- and beta-spin bitstrings.
  
  Frozen core orbitals are excluded from all bitstrings.
  
  The first alpha and beta bitstrings are assumed to correspond to the Hartree–Fock reference determinant.
  In `FCI_results/`, the `*.strA` and `*.strB` files contain the complete sets of alpha- and beta-spin 
  bitstrings in the FCI space.
  
  In `TBSCI_results/`, the corresponding files contain bitstrings extracted from important determinants 
  provided by DICE and ordered by their weights; each line contains an integer-encoded bitstring and its
  associated weight.

- *.out

  In `FCI_results/`, each test case contains a single output file corresponding to the FCI calculation.

  In `TBSCI_results/`, multiple output files are provided for each test case. For the closed-shell systems 
  reported here, nAlpha = nBeta, and therefore only a single number appears in the filename. Using the same 
  nAlpha and nBeta values should reproduce results that are nearly identical to the corresponding outputs.
  
---

## How to run (typical workflow)

Typical workflow:

1. Download the data archives from the Releases page.
2. Extract the archives to obtain `FCI_results/` and `TBSCI_results/`.
3. Choose a test-case directory, for example: `TBSCI_results/CR2_STO3G/`
4. Run the executable with MPI.

Example commands (Linux/HPC):

  Download data archives (example for release tag v1.0)
  ```
  wget https://github.com/enhua-xu/TBSCI-202603/releases/download/v1.0/FCI_results.tar.gz
  wget https://github.com/enhua-xu/TBSCI-202603/releases/download/v1.0/TBSCI_results.tar.gz
  ```

  Extract
  ```
  tar -xzf FCI_results.tar.gz
  tar -xzf TBSCI_results.tar.gz
  ```

  Enter a test-case directory
  ```
  cd TBSCI_results/CR2_STO3G/
  ```

  Load runtime environment (example for Intel oneAPI; may differ by system)
  ```
  source /opt/intel/oneapi/setvars.sh
  ```

  Run (Intel_exe example; mpirun/mpiexec depending on system)
  ```
  mpirun -np <nproc> /path/to/Intel_exe.out <prefix> <nAlpha> <nBeta> <nThreads>
  ```

where:
- `<prefix>` is the name of the selected test case (e.g., `CR2_STO3G`), which is used to locate the
  corresponding input files (`filename.FCIDUMP`, `filename.strA`, `filename.strB`, `filename.input`, 
  and `filename.charT`).
- `<nAlpha>` is the number of alpha bitstrings to include in the TBSCI calculation.
- `<nBeta>` is the number of beta  bitstrings to include in the TBSCI calculation.
- `<nThreads>` is the number of OpenMP threads per MPI process.

Notes:
- `<nAlpha>` MUST be ≥ `<nproc>`; otherwise CI-vector segments cannot be distributed across MPI processes.
- `<nThreads>` MUST be > `1`. One OpenMP thread is dedicated to data fetching,
  while the remaining threads are used for computation.
  
---

## Scope, limitations, and citation

All results reported in the associated manuscript and the output files provided in the GitHub Releases
were obtained on Fugaku using `Fugaku_exe.out`.

Intel_exe.out was compiled on an internal Intel-based Linux cluster and is provided for reference; 
its parallel performance on other HPC systems has not been systematically evaluated.

- This repository contains executable binaries and representative data only; source code is not included.
- This repository is frozen to match the results reported in the associated manuscript.
- Future developments will be released in separate repositories.
- If you use this repository in scientific work, please cite the associated paper:

  https://arxiv.org/abs/2503.10335

Formal citation information will be added upon publication.
