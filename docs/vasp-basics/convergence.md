# ENCUT & KPOINTS Convergence Testing

Before running structural relaxations or mechanical properties calculations, you must perform convergence testing. Running calculations with an un-converged plane-wave cutoff or a sparse Brillouin zone grid will result in incorrect total energies and forces.

Our goal is to find the minimum values where the total energy stabilizes within **1 meV per atom**, optimizing computational speed without sacrificing precision.

---

## 1. Plane-Wave Cutoff (ENCUT) Convergence

The plane-wave cutoff determines the maximum kinetic energy of the plane waves used as basis functions. It is controlled by the `ENCUT` tag in the `INCAR` file.

### Step-by-Step Hands-On Guide
To test `ENCUT`, keep your lattice coordinates and `KPOINTS` fixed, and systematically vary the cutoff energy. A good rule of thumb is to start from the maximum `ENMAX` value found in your `POTCAR` file and scale upward.

### Example Bash Script for Automation (`run_encut.sh`)
Instead of manually modifying the `INCAR` file every time, you can use this simple automation script to run the calculations sequentially:

```bash
#!/bin/bash
# Loop over ENCUT values from 350 eV to 600 eV in steps of 50
for i in 350 400 450 500 550 600
do
cat > INCAR << EOF
SYSTEM = Al-Bulk-ENCUT-Test
IBRION = -1      # No ionic relaxation
NSW    = 0       # 0 ionic steps
ISMEAR = 1       # Methfessel-Paxton smearing
SIGMA  = 0.2
ENCUT  = $i      # Varying Cutoff Energy
EDIFF  = 1E-5    # Electronic convergence tolerance
EOF

echo "Running VASP for ENCUT = $i eV..."
mpirun -np 4 vasp_std

# Extract total free energy (TOTEN) from the OUTCAR file
E=$(grep "FREE ENERGIE" OUTCAR | tail -1 | awk '{print $5}')
echo "$i$E" >> encut_convergence.dat
done

```
---

## 2. K-Point Grid Density (KPOINTS) Convergence

The `KPOINTS` file controls how the continuous Brillouin zone of your crystal lattice is sampled across a discrete grid. For metals, a denser grid is necessary to capture the sharp Fermi surface accurately, while insulators can get away with a much sparser grid.

### Step-by-Step Hands-On Guide
1. **Fix your ENCUT:** Keep your `ENCUT` value locked at the converged parameter you found in Step 1 (e.g., 500 eV).
2. **Keep Core Files Constant:** Keep your `INCAR` settings and `POSCAR` structure completely identical.
3. **Vary the Grid:** Systematically increase the mesh dimensions in the `KPOINTS` file (e.g., $4\times4\times4$, $6\times6\times6$, $8\times8\times8$, etc.) and record the final total free energy.

### Example Automation Script (`run_kpoints.sh`)
You can automate this grid sweep using a bash script that overwrites the `KPOINTS` file in each loop iteration:

```bash
#!/bin/bash
# Loop over Monkhorst-Pack grid sizes
for k in 4 6 8 10 12 14
do
cat > KPOINTS << EOF
K-Points Convergence Loop
0
Monkhorst-Pack
$k $k$k
0  0  0
EOF

echo "Running VASP for KPOINTS grid = ${k}x${k}x${k}..."
mpirun -np 4 vasp_std

# Extract total free energy from OUTCAR
E=$(grep "FREE ENERGIE" OUTCAR | tail -1 | awk '{print $5}')
echo "$k$E" >> kpoints_convergence.dat
done

```
