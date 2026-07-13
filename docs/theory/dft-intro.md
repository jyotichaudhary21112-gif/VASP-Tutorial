## Introduction to Density Functional Theory (DFT)

Welcome to the theoretical core of this hub. Before executing calculations in code packages, it is essential to understand how quantum mechanics is simplified to predict the properties of real materials.

---

## 1. The Many-Body Quantum Problem

To find the ground-state energy of a crystal lattice, we must solve the time-independent Schrödinger equation for a system of interacting nuclei and electrons:

$$\hat{H}\Psi = E\Psi$$

For a realistic solid with $N$ electrons and $M$ nuclei, the full quantum mechanical Hamiltonian operator $\hat{H}$ consists of five distinct terms:

$$\hat{H} = -\sum_{i=1}^{N} \frac{\hbar^2}{2m_e}\nabla_i^2 - \sum_{A=1}^{M} \frac{\hbar^2}{2M_A}\nabla_A^2 - \sum_{i=1}^{N}\sum_{A=1}^{M} \frac{Z_A e^2}{4\pi\epsilon_0 r_{iA}} + \sum_{i<j}^{N} \frac{e^2}{4\pi\epsilon_0 r_{ij}} + \sum_{A<B}^{M} \frac{Z_A Z_B e^2}{4\pi\epsilon_0 R_{AB}}$$

### The Terms Broken Down:
1. **Electronic Kinetic Energy:** Kinetic movement of all electrons.
2. **Nuclear Kinetic Energy:** Kinetic movement of heavy nuclei.
3. **Electron-Nuclear Attraction:** Coulomb attraction holding electrons to the cores.
4. **Electron-Electron Repulsion:** Inter-electronic repulsion forces.
5. **Nuclear-Nuclear Repulsion:** Inter-nuclear repulsion forces.

Because every electron interacts simultaneously with every other electron, the exact wave function $\Psi(\mathbf{r}_1, \mathbf{r}_2, \dots, \mathbf{r}_N)$ depends on $3N$ spatial coordinates. For a simple macroscopic piece of metal containing $\sim 10^{23}$ particles, solving this equation directly is completely impossible.

---

## 2. Fundamental Approximations

To make materials modeling practically computable, two primary breakthroughs are implemented:

### A. The Born-Oppenheimer Approximation
Atomic nuclei are thousands of times heavier and move far slower than light electrons. We can treat the nuclei as fixed, stationary points in space relative to electronic motion. This completely eliminates the nuclear kinetic energy term and treats nuclear-nuclear repulsion as a static energy offset constant.

### B. The Hohenberg-Kohn Theorems
Hohenberg and Kohn proved that the complex many-body wave function can be replaced entirely by the physical **electron density**, $\rho(\mathbf{r})$, which depends on only **3 spatial variables ($x, y, z$)**, regardless of system size.

1. **Theorem I:** The ground-state properties of a many-electron system are uniquely determined by an electron density $\rho(\mathbf{r})$ that depends on only three spatial coordinates.
2. **Theorem II:** The density that minimizes the total energy functional is the exact ground-state density of the system.

---

## 3. The Kohn-Sham Approach

Kohn and Sham mapped the impossible problem of interacting electrons onto a fictional system of **non-interacting electrons** moving within an effective potential field ($V_{\text{eff}}$). 

The total energy functional is partitioned cleanly:

$$E[\rho] = T_s[\rho] + E_H[\rho] + E_{\text{ext}}[\rho] + E_{\text{xc}}[\rho]$$

Where:
* $T_s[\rho]$ is the kinetic energy of non-interacting electrons.
* $E_H[\rho]$ is the classical Hartree electron-electron repulsion energy.
* $E_{\text{ext}}[\rho]$ is the external potential energy from the fixed nuclei.
* **$E_{\text{xc}}[\rho]$ (Exchange-Correlation Functional):** This contains all remaining complex quantum mechanical errors (many-body exchange interactions, electron correlation, and the kinetic energy correction).

Because the exact mathematical shape of $E_{\text{xc}}$ is unknown, we must use approximations like the **Local Density Approximation (LDA)** or the **Generalized Gradient Approximation (GGA-PBE)**.

---

## 🛠️ Ground State Self-Consistent Field (SCF) Cycle

When your calculation runs, the software utilizes an interactive electronic relaxation loop to find the lowest energy:

```text
 [ Guess initial density ρ_0 ]
              │
              ▼
 [ Solve Kohn-Sham Equations ] ──► [ Compute New Density ρ_new ]
              ▲                                  │
              │                                  ▼
   [ Mix Old & New Densities ] ◄─── No ─── [ Is E_new - E_old < EDIFF? ]
                                                 │
                                                Yes
                                                 │
                                                 ▼
                                        [ Output Ground State Energy ]
