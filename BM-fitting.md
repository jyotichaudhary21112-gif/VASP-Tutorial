# Derivation of the Third-Order Birch-Murnaghan Equation of State ($E(V)$)

To obtain the energy-volume ($E(V)$) relationship from the third-order Birch-Murnaghan pressure equation, we integrate the pressure with respect to volume, utilizing the thermodynamic definition:

$$P(V) = -\frac{dE}{dV} \implies E(V) = E_0 - \int_{V_0}^{V} P(V) \, dV$$

---

### Step 1: Expressing Pressure using Eulerian Finite Strain ($f$)
Instead of integrating directly with respect to volume $V$, we change variables to the Eulerian finite strain $f$, defined as:

$$f = \frac{1}{2}\left[\left(\frac{V_0}{V}\right)^{2/3} - 1\right] \implies \left(\frac{V_0}{V}\right)^{2/3} = 2f + 1$$

From the pressure derivation, the pressure in terms of strain $f$ is given by:

$$P(f) = 3K_0 (1 + 2f)^{5/2} f \left[1 + \frac{3}{4}(K_0' - 4)f\right]$$

---

### Step 2: Converting the Volume Differential to Strain
To change the integral's differential from $dV$ to $df$, we differentiate the strain definition with respect to volume:

$$df = -\frac{1}{3}V_0^{2/3} V^{-5/3} dV \implies dV = -3 V_0 \left(1 + 2f\right)^{-5/2} df$$

---

### Step 3: Setting Up and Simplifying the Integral
Substituting $P(f)$ and $dV$ into the energy integral:

$$E(f) = E_0 - \int_{0}^{f} P(f) \left( -3 V_0 (1 + 2f)^{-5/2} \right) df$$

Notice how the term $(1 + 2f)^{5/2}$ from the pressure expression and the term $(1 + 2f)^{-5/2}$ from the differential **completely cancel each other out**:

$$E(f) = E_0 + 3 V_0 K_0 \int_{0}^{f} f \left[ 1 + \frac{3}{4}(K_0' - 4)f \right] df$$

---

### Step 4: Solving the Polynomial Integral
Integrating the simple polynomial with respect to $f$ yields:

$$\int f \left[ 1 + \frac{3}{4}(K_0' - 4)f \right] df = \frac{1}{2}f^2 + \frac{3}{16}(K_0' - 4)f^3$$

Multiplying through by $3 V_0 K_0$ gives the final **BM3 Energy-Volume equation**:

$$E(V) = E_0 + \frac{9}{2} V_0 K_0 \left[ \frac{f^2}{2} + \frac{3}{4}(K_0' - 4)\frac{f^3}{3} \right]$$

---

### Summary for DFT / SQS Implementation
1. **Data Generation:** Compute total energies ($E$) across a range of strained volumes ($V$) for your alloy supercell using DFT.
2. **Curve Fitting:** Use a non-linear least-squares solver to fit the $(E_i, V_i)$ pairs to the $E(V)$ equation above.
3. **Extracted Parameters:** The optimizer determines $E_0$ (minimum energy), $V_0$ (equilibrium volume), and $K_0$ (bulk modulus).
