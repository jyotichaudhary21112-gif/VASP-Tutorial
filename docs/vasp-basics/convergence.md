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
