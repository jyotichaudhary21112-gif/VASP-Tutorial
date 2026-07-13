# Advanced Materials Modeling Hub 🚀

Welcome to the **Advanced Materials Modeling Hub**! This repository hosts a comprehensive, hands-on pedagogical manual and workflow cookbook for running ab initio Density Functional Theory (DFT) simulations using VASP (Vienna Ab initio Simulation Package).

The goal of this project is to bridge the gap between abstract quantum mechanics equations and practical, production-grade command-line workflows for materials engineering.

---

## 📂 Documentation Structure

The tutorial site is organized logically into five core pillars:

1. **Theoretical Foundations** – Introduction to the Hohenberg-Kohn theorems, Kohn-Sham equations, and Exchange-Correlation functionals (LDA, GGA-PBE).
2. **VASP Basics & Convergence** – Mastering the "Big 4" input files (`INCAR`, `POSCAR`, `KPOINTS`, `POTCAR`) and automation scripts for `ENCUT` and K-point convergence testing.
3. **Bulk & Mechanical Properties** – Computing Equation of State (EOS) curves, Bulk Modulus ($B_0$), and full Elastic Stiffness Tensors ($C_{ij}$) for all crystal systems.
4. **Surface & Defect Engineering** – Slicing crystal lattices to calculate Surface Energies ($\gamma$), predicting equilibrium Wulff Shapes, and mapping Generalized Stacking Fault Energies (GSFE).
5. **Kinetics & Reaction Pathways** – Locating transition states and activation energy barriers using the Nudged Elastic Band (NEB) method.

---

## 🛠️ How to Run This Documentation Locally

This website is built using **MkDocs** with the premium **Material theme**. If you want to clone this repository and preview the site locally on your computer, follow these simple steps:

### 1. Clone the repository
```bash
git clone [https://github.com/YOUR_USERNAME/vasp-tutorials.git](https://github.com/YOUR_USERNAME/vasp-tutorials.git)
cd vasp-tutorials
