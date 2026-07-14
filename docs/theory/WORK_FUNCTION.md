# Electrostatic Work Function Calculations in VASP

This documentation covers the theoretical background, computational workflow, and data extraction methods for calculating the work function ($\Phi$) of metallic slab surfaces using VASP.

---

## 1. Theoretical Framework

The work function ($\Phi$) is the minimum energy required to remove an electron from the interior of a solid surface to the vacuum level directly outside. It is calculated using the relation:

$$\Phi = V_{\text{vac}} - E_F$$

Where:
* **$V_{\text{vac}}$** is the electrostatic potential energy plateau value located in the center of the vacuum region.
* **$E_F$** is the Fermi energy of the relaxed slab system.

To obtain an accurate, noise-free value for $V_{\text{vac}}$, the 3D local potential grid $V(x,y,z)$ from VASP is planarly averaged along the $z$-axis (perpendicular to the surface facet):

$$V_{\text{avg}}(z) = \frac{1}{A} \iint V(x,y,z) \, dx \, dy$$

---

## 2. Two-Step Computational Workflow

A rigorous work function calculation requires separating the geometric relaxation from the electrostatic potential evaluation.

### Step 1: Structural Relaxation
Relax the initial slab structure to optimize the interplanar spacing of the surface layers.
* **Key Setup:** `IBRION = 2`, `NSW = 60`, `ISIF = 2` (locks the cell volume/shape, allowing only ions to relax).
* **Result:** Generates the optimized ground-state geometry saved in `CONTCAR`.

### Step 2: Static Evaluation Run
Copy the relaxed coordinates (`CONTCAR` $\rightarrow$ `POSCAR`) to a new folder and modify the `INCAR` parameters to freeze ionic motion and export the Hartree potential grid.

---

## 3. Recommended `INCAR` Template (e.g., $\text{Ni}(111)$ Slab)

```ini
SYSTEM = Ni(111) Work Function Static Run
PREC   = Accurate
ENCUT  = 450.0
EDIFF  = 1e-08
ALGO   = Normal

# 1. LOCK GEOMETRY FOR STATIC RUN
IBRION = -1      # Static calculation (no ionic adjustments)
NSW    = 0       # 0 ionic steps
ISIF   = 2       

# 2. ELECTRONIC SMEARING
ISMEAR = 1       # Methfessel-Paxton is standard for metals
SIGMA  = 0.2     # Use ISMEAR = 0 / SIGMA = 0.05 if a sharper Fermi edge is needed

# 3. SPIN POLARIZATION (Crucial for magnetic transition metals)
ISPIN  = 2
MAGMOM = 14*2.0  # Initializing with a high guess ensures clean convergence

# 4. POTENTIAL GRID OUTPUT (Critical for Work Function)
LVTOT  = .TRUE.  # Writes the total local potential to the LOCPOT file
LVHAR  = .TRUE.  # Writes the electrostatic (Hartree) potential to LOCPOT
LREAL  = .FALSE.
LWAVE  = .FALSE.
LCHARG = .TRUE.
