# Exchange-Correlation (XC) Functionals

In the Kohn-Sham formulation of Density Functional Theory (DFT), all the complex, many-body quantum mechanical interactions of electrons are swept into a single term: the **Exchange-Correlation energy ($E_{\text{xc}}[\rho]$)**. 

Because the exact mathematical form of $E_{\text{xc}}$ is unknown, we must rely on physical approximations to simulate real materials. Choosing the right functional determines the accuracy of your VASP calculation.

---

## 1. Jacob's Ladder of DFT Functionals

John Perdew famously conceptualized the hierarchy of XC approximations as **Jacob's Ladder**, scaling up from the simple local electron gas up to the exact quantum mechanical exchange limit. As you ascend the rungs, accuracy generally increases, but so does the computational cost.



### Rung 1: Local Density Approximation (LDA)
The simplest approximation. It assumes that the exchange-correlation energy at any specific point in space is exactly equal to that of a **homogeneous electron gas** having the same local charge density:

$$
E_{\text{xc}}^{\text{LDA}}[\rho] = \int \rho(\mathbf{r}) \varepsilon_{\text{xc}}^{\text{HEG}}(\rho(\mathbf{r})) d\mathbf{r}
$$

* **Best For:** Simple metals with slowly varying electron densities.
* **Drawback:** Tends to overbind molecules and solids, underestimating lattice constants by 1–3% and overestimating cohesive energies.

### Rung 2: Generalized Gradient Approximation (GGA)
GGA improves on LDA by looking not just at the local charge density, but also at how fast that density is changing in space by including the **gradient of the density ($\nabla \rho$)**:

$$
E_{\text{xc}}^{\text{GGA}}[\rho] = \int f(\rho(\mathbf{r}), \nabla \rho(\mathbf{r})) d\mathbf{r}
$$

* **Common Types in VASP:** **PBE** (Perdew-Burke-Ernzerhof) is the global standard for solids. **PW91** is another common variant.
* **Best For:** General structural, mechanical, and energetic properties of transition metals, semiconductors, and insulators.
* **Drawback:** Tends to slightly underbind solids, leading to slightly larger lattice constants (overestimated by 1–2%).

### Rung 3: Meta-Generalized Gradient Approximation (meta-GGA)
Meta-GGA goes a step further by including the **kinetic energy density ($\tau$)** of the occupied single-particle orbitals:

$$
E_{\text{xc}}^{\text{meta-GGA}}[\rho] = \int f(\rho(\mathbf{r}), \nabla \rho(\mathbf{r}), \tau(\mathbf{r})) d\mathbf{r}
$$

* **Common Types in VASP:** **SCAN** (Strongly Constrained and Appropriately Normed).
* **Best For:** Capturing diverse bonding environments (metallic, covalent, ionic, and hydrogen bonds) simultaneously with exceptional accuracy.

---

## 2. Advanced Rungs: Hybrid Functionals & Exact Exchange

Standard semilocal functionals (LDA, GGA) suffer from **self-interaction error**, where an electron artificially repels itself. This leads to a severe underestimation of electronic band gaps in semiconductors and insulators.

To resolve this, **Hybrid Functionals** mix a fraction of exact, non-local exchange calculated from Hartree-Fock (HF) theory with standard DFT exchange:

$$
E_{\text{xc}}^{\text{hybrid}} = \alpha E_{\text{x}}^{\text{HF}} + (1-\alpha) E_{\text{x}}^{\text{DFT}} + E_{\text{c}}^{\text{DFT}}
$$

### Popular Hybrid Choices in VASP:
* **HSE06 (Heyd-Scuseria-Ernzerhof):** Uses a screened Coulomb potential where exact exchange is only mixed at short spatial ranges. This makes it computationally feasible for periodic solids.
* **B3LYP:** Widely popular in chemistry for molecular systems, though less optimal for bulk periodic systems.

| Functional Type | Relative CPU Cost | Band Gap Accuracy | Typical Application |
| :--- | :--- | :--- | :--- |
| **LDA** | $1\times$ | Poor (Underestimates $\sim 40\%$) | Simple metallic bulks |
| **GGA-PBE** | $1\times$ | Moderate (Underestimates $\sim 30\%$) | Standard structural geometry runs |
| **SCAN** | $2\times - 3\times$ | Improved | Phase stabilities, complex bonds |
| **HSE06** | $100\times - 1000\times$ | Excellent | Optical properties & Band structures |

---

## 3. Van der Waals (vdW) Corrections

Standard DFT functionals fail completely at capturing long-range dispersion forces (dispersion interactions caused by dynamic polarization fluctuations, like those holding graphite layers together). 

To model 2D layered materials (e.g., graphene, $\text{MoS}_2$) or molecular adsorption on surfaces, you must switch on empirical dispersion corrections in VASP:
* **DFT-D3 / DFT-D4 (Grimme's methods):** Adds a semi-empirical pair-potential correction term ($-\frac{C_6}{r^6}$) to the total DFT energy.
* **vdW-DF functionals (e.g., optB86b-vdW):** A fully non-local functional approach that accounts for dispersion directly within the electronic charge density evaluation loop.
