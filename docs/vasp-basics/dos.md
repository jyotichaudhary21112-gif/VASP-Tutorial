# Density of States (DOS) Calculations

The **Density of States (DOS)** is a fundamental property that describes the number of electronic states available for electrons to occupy at specific energy levels. It is one of the most powerful tools for determining whether a material behaves as a metal, semiconductor, or insulator, and for tracking orbital hybridization.

Computing a clean DOS is a **two-step workflow**:
1. Run a high-precision static Self-Consistent Field (SCF) calculation to get a converged charge density.
2. Run a non-self-consistent (ICHARG=11) run with a much denser k-point grid to sample the energy levels smoothly.

---

## Step 1: The Static SCF Calculation

Before calculating the DOS, VASP needs an accurate ground-state electronic charge density. This run uses the optimized coordinates from your previous relaxation step.

### Input File Preparations
* **`POSCAR`:** Copy the optimized structure from your relaxation run (`cp ../relaxation/CONTCAR ./POSCAR`).
* **`INCAR`:** Set `NBRION = -1` and `NSW = 0` since the atoms should not move. Increase the electronic convergence tolerance (`EDIFF = 1E-6`).

### Example INCAR for Static Step (`INCAR.scf`)
```ini
SYSTEM = Al-Bulk-Static-SCF
ENCUT  = 500         # Use your converged ENCUT value
EDIFF  = 1E-6        # Strict electronic convergence

# No structural movements
IBRION = -1          # Ions are locked
NSW    = 0           # 0 ionic steps

# Smearing for accurate total energy
ISMEAR = 1           # Methfessel-Paxton for metals
SIGMA  = 0.2
LCHARG = .TRUE.      # CRITICAL: Saves CHGCAR for the next step
LWAV   = .FALSE.     # Saves disk space by turning off WAVECAR
