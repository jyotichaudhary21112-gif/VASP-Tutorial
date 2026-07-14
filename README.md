# Advanced Materials Modeling Hub 🚀

Welcome to the **Advanced Materials Modeling Hub**! This repository hosts a comprehensive, hands-on pedagogical manual and workflow cookbook for running ab initio Density Functional Theory (DFT) simulations using VASP (Vienna Ab initio Simulation Package).

The goal of this project is to bridge the gap between abstract quantum mechanics equations and practical, production-grade command-line workflows for materials engineering.

---

## 📂 Documentation Structure

The tutorial site is organized logically into five core pillars:

1. **Theoretical Foundations** – Introduction to the Hohenberg-Kohn theorems, Kohn-Sham equations, and Exchange-Correlation functionals (LDA, GGA-PBE).
2. **VASP Basics & Convergence** – Mastering the "Big 4" input files (`INCAR`, `POSCAR`, `KPOINTS`, `POTCAR`) and automation scripts for `ENCUT` and K-point convergence testing.
3. **Bulk & Mechanical Properties** – Computing Equation of State (EOS) curves, Bulk Modulus ($B_0$), and full Elastic Stiffness Tensors ($C_{ij}$) for all crystal systems.
4. **Surface & Defect Engineering** – Slicing crystal lattices to calculate Surface Energies ($\gamma$), extracting **Electrostatic Work Functions ($\Phi$)** via planar-averaged electrostatic potentials (`LOCPOT`), predicting equilibrium Wulff Shapes, and mapping Generalized Stacking Fault Energies (GSFE).
5. **Kinetics & Reaction Pathways** – Locating transition states and activation energy barriers using the Nudged Elastic Band (NEB) method.
---
## 🧪 DFT Calculation Setup: VASP Best Practices

When setting up your VASP calculations, the accuracy of your results depends heavily on the chosen pseudopotentials and the energy cutoff (`ENCUT`).

### Pseudopotential Selection
For transition metals like Molybdenum (Mo) and Tungsten (W), choose your `POTCAR` files based on the required level of accuracy:
* **`_sv` (semi-core s- and p-valence):** Recommended for high-precision structural and energetic studies.
* **`_pv` (semi-core p-valence):** A balanced choice for general bulk and surface calculations.

### Setting the Energy Cutoff (ENCUT)
Consistency is critical. To ensure your calculations are converged:

1. **Check ENMAX:** Locate the `ENMAX` value inside your `POTCAR` files. You can find this by running the following command in your terminal:
   ```bash
   grep "ENMAX" POTCAR


# VASP Computational Tutorials & Research

Welcome to my research repository. This space tracks my journey in **Density Functional Theory (DFT)** and **Solidification Theory**, specifically focusing on refractory metals like Cr, Mo, and W.

---

## 1. ⚙️ VASP Configuration Reference
The `INCAR` file is the control center for your simulations. Use this reference to understand the core tags required for stable metallic calculations.

| Tag | Category | Purpose | When to use |
| :--- | :--- | :--- | :--- |
| `PREC` | Accuracy | Precision level | Always set to `Accurate` for reliable energies. |
| `ENCUT` | Energy | Plane-wave cutoff | Must be set to $1.3 \times \max(ENMAX)$ from `POTCAR`. |
| `ALGO` | Solver | Electronic algorithm | `Fast` is more stable than `Normal` for cell relaxation. |
| `ISMEAR` | Integration | Brillouin zone sampling | `1` is standard for metallic systems. |
| `SIGMA` | Broadening | Fermi surface broadening | `0.1-0.2` helps stabilize metallic occupation. |
| `ISIF` | Geometry | Structural constraints | `2` for ion relaxation, `3` for full cell relaxation. |
| `EDIFFG` | Stopping | Force convergence | `-0.01` ensures atoms are well-relaxed. |
| `ISYM` | Symmetry | Symmetrization | `0` disables symmetry to prevent numerical crashes. |



---

## 2. 🚀 The Systematic Relaxation Workflow
To avoid the `ZPOTRF` error, never start with a full cell relaxation. Follow these stages:

### Phase A: Ion Relaxation (Fixed Cell)
Use this to allow atoms to settle into their positions without changing the cell dimensions.
* **INCAR:** `ISIF = 2`, `IBRION = 2`, `ALGO = Fast`.

### Phase B: Full Cell Relaxation
Once Phase A converges, use the resulting `CONTCAR` as your new `POSCAR` to relax the cell shape and volume.
* **INCAR:** `ISIF = 3`, `ALGO = Fast`.

---

## 3. 🛠 Troubleshooting: "ZPOTRF" Failure
If you encounter `LAPACK: Routine ZPOTRF failed!`, follow this recovery sequence:
1. **Relax Precision:** Temporarily set `EDIFF = 1E-6` (you can tighten it later).
2. **Disable Symmetry:** Set `ISYM = 0` in your `INCAR`.
3. **Slow Down Mixing:** Add the following to your `INCAR`:
   ```text
   AMIX = 0.2
   BMIX = 0.0001
## 🛠️ How to Run This Documentation Locally

This website is built using **MkDocs** with the premium **Material theme**. If you want to clone this repository and preview the site locally on your computer, follow these simple steps:

### 1. Clone the repository
```bash
git clone [https://github.com/YOUR_USERNAME/vasp-tutorials.git](https://github.com/YOUR_USERNAME/vasp-tutorials.git)
cd vasp-tutorials
