# Structural Relaxation & Geometry Optimization

Once your plane-wave cutoff energy (`ENCUT`) and `KPOINTS` mesh are converged, the next critical phase is **Structural Relaxation**. Real-world crystals don't exist at arbitrary mathematical dimensions; they adjust their atomic bonds and volume to minimize their overall free energy. 

This guide outlines how to configure VASP to find the true equilibrium ground-state geometry of your material.

---

## 1. Key INCAR Tags for Structural Optimizations

To tell VASP to move the ions and relax the system, you need to configure specific flags in your `INCAR` file. 

* **`IBRION`:** Controls the optimization algorithm. 
  * `IBRION = 2`: Conjugate-gradient algorithm. This is the most robust option for starting configurations that are far from equilibrium.
  * `IBRION = 1`: Quasi-Newton RMM-DIIS algorithm. Excellent for fine-tuning a structure that is already very close to its minimum.
* **`NSW`:** The maximum number of ionic steps VASP is allowed to take (e.g., `NSW = 100`). If the structure doesn't converge within this limit, the calculation stops and you will need to restart it using the updated `CONTCAR` coordinates.
* **`EDIFFG`:** The convergence criteria for the ionic relaxation loop.
  * Positive value (e.g., `EDIFFG = 1E-3`): Stops the calculation when the total energy change between two ionic steps is less than $10^{-3}$ eV.
  * Negative value (e.g., `EDIFFG = -0.02`): **Recommended.** Stops the calculation only when the absolute quantum forces on *every single atom* drop below $0.02\text{ eV}/\text{Å}$.

---

## 2. Setting the ISIF Tag (Relaxation Modes)

The `ISIF` tag is the most powerful control knob during geometry optimization. It dictates which degrees of freedom VASP is allowed to alter during the run:

| ISIF Value | Calculate Forces? | Optimize Ions? | Change Cell Shape? | Change Cell Volume? | Typical Use Case |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **2** | Yes | **Yes** | No | No | Internal relaxation (fixed lattice size/box) |
| **3** | Yes | **Yes** | **Yes** | **Yes** | Full optimization (bulk lattice parameters) |
| **4** | Yes | **Yes** | **Yes** | No | Fixed-volume shape relaxation |

> ⚠️ **Important Note on Pulay Stress (`ISIF = 3`):** When you change the volume of a cell dynamically during a calculation, the plane-wave basis set changes size artificially. This creates an unphysical pressure called Pulay Stress. To eliminate this issue, always increase your `ENCUT` value by **15–20%** above your converged value if you use `ISIF = 3`.

---

## 3. Example INCAR File for Full Bulk Relaxation (`ISIF = 3`)

Below is a standard production template for executing a full structural relaxation on a bulk material:

```ini
# Global Settings
SYSTEM = Al-Bulk-Full-Relaxation
Electronic-Convergence
EDIFF  = 1E-6        # High precision electronic tolerance

# Ionic Relaxation Settings
IBRION = 2           # Conjugate-Gradient algorithm
NSW    = 150         # Allow up to 150 atomic steps
ISIF   = 3           # Relax ions, cell shape, AND cell volume
EDIFFG = -0.02       # Converge when all atomic forces < 0.02 eV/A

# Precision Scaling
ENCUT  = 550         # Converged value (450 eV) boosted by ~20%
ISMEAR = 1           # Methfessel-Paxton smearing for metals
SIGMA  = 0.2
