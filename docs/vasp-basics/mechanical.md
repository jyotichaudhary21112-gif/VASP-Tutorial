# Mechanical Properties: Bulk Modulus & Elastic Constants

Once a material's ground-state structure is optimized, you can calculate its response to mechanical deformation. This guide covers how to determine the **Bulk Modulus ($B_0$)** using manual strain loops and how to calculate the full **Elastic Constants Tensor ($C_{ij}$)** automatically using VASP.

---

## 1. Bulk Modulus via Energy vs. Strain Scaling

The Bulk Modulus measures a material's resistance to uniform compression. The standard approach is to apply small scaling strains ($\pm 1\%, \pm 2\%, \pm 3\%$) to the optimized equilibrium lattice volume ($V_0$), calculate the total energy ($E$) for each step, and fit the resulting curve to the **Birch-Murnaghan Equation of State (EOS)**.



### Step-by-Step Hands-On Guide
1. Start with your fully relaxed `CONTCAR` structure as your baseline parameter.
2. Write a script to scale the lattice vectors in the `POSCAR` by a scaling factor $x$ (where $x = 0.97$ to $1.03$).
3. Run a static calculation for each volume. **Crucial:** Ensure `ENCUT` is set 15–20% higher than your standard converged value to eliminate Pulay stress errors as the cell volume changes!

### Automation Script Example (`run_volume_sweep.sh`)
```bash
#!/bin/bash
# Loop through scaling factors from -3% to +3% deformation
for scale in 0.97 0.98 0.99 1.00 1.01 1.02 1.03
do
  # Create a temporary directory for each strain state
  mkdir -p "vol_${scale}"
  
  # Use an awk script or tool to modify the lattice parameter line (line 2) of POSCAR
  # In this example, we assume your base POSCAR is ready and we just scale line 2:
  awk -v s="$scale" 'NR==2 {print s} NR!=2 {print}' base_POSCAR > "vol_${scale}/POSCAR"
  
  cp INCAR POTCAR KPOINTS "vol_${scale}/"
  cd "vol_${scale}"
  
  echo "Running static calculation for volume scale = $scale"
  mpirun -np 4 vasp_std
  
  # Extract volume and energy
  VOL=$(grep "volume of cell" OUTCAR | tail -1 | awk '{print $5}')
  ENG=$(grep "FREE ENERGIE" OUTCAR | tail -1 | awk '{print $5}')
  echo "$VOL    $ENG" >> ../eos_data.dat
  cd ..
done
