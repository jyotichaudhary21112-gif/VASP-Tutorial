# Elastic Constants: Practical Workflow & Tensor Reduction

## The Big Shortcut: Is a single INCAR modification enough?

Before diving into the complex tensor mathematics, let’s answer the most common practical question: **Is it true that a single change in the `INCAR` file will give us the elastic constants?**

**Yes, that is completely true!** In modern VASP, you do not have to manually distort the crystal cell, write complex scripting loops to run dozens of separate geometric configurations, or manually fit the resulting data to equations. By setting a single major flag—`IBRION = 6`—along with an appropriate relaxation flag (`ISIF >= 3`), VASP completely automates the workflow behind the scenes. 

It automatically applies all the required strain tensors, calculates the resulting internal stress changes, and prints out the fully completed, beautifully averaged $6 \times 6$ elastic stiffness matrix directly at the bottom of your standard `OUTCAR` output file.

---

## Step 1: How VASP Computes Moduli Automatically

When you activate this automated run (`IBRION = 6`), VASP executes a finite difference scheme using your optimized ground-state geometry as a base:

1. **Atomic Forces:** The system computes ground-state forces via the **Hellmann–Feynman theorem**:
   $$F_I = -\frac{\partial E}{\partial R_I} = -\int n(\mathbf{r})\frac{\partial V_{\text{ext}}}{\partial R_I} d^3r$$
2. **Deformation Loop:** VASP applies positive and negative strain tensors ($\pm\epsilon$) to the lattice vectors based on your `POTIM` step size variable.
3. **Stress Tensor Evaluation:** For each distortion, it measures the resulting Cauchy stress tensor component responses ($\sigma_{ij}$).
4. **Hooke's Law Matrix Inversion:** It relates them using linear elasticity:
   $$\sigma_{ij} = \sum_{k,l=1}^{3} C_{ijkl} \epsilon_{kl}$$

---

## Step 2: The Core Reductions (From 81 down to 21)

A raw four-rank elasticity tensor ($C_{ijkl}$) in 3D space contains $3^4 = 81$ total elements. Symmetries collapse this complexity down rapidly before crystal geometry is even considered:

### 1. Stress & Strain Symmetries (81 $\rightarrow$ 36 Components)
Because an isolated cell must remain in rotational equilibrium, the stress tensor is intrinsically symmetric ($\sigma_{ij} = \sigma_{ji}$). Similarly, the infinitesimal strain tensor is defined symmetrically by construction:
$$\epsilon_{ij} = \frac{1}{2}\left( \frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i} \right)$$
These pair-wise internal symmetries ($C_{ijkl} = C_{jikl}$ and $C_{ijkl} = C_{ijlk}$) contract the double tensor indices down to a single index running from 1 to 6 using **Voigt Notation**:

| Tensor Pair $(ij)$ | 11 | 22 | 33 | 23 / 32 | 13 / 31 | 12 / 21 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Voigt Index ($m, n$)** | **1** | **2** | **3** | **4** | **5** | **6** |

This reduces the system parameters from 81 down to a $6 \times 6$ matrix containing 36 elements.

### 2. Thermodynamic Strain Potential (36 $\rightarrow$ 21 Components)
Because elastic deformation is conservative, a strain energy density function ($w$) exists such that:
$$\sigma_{ij} = \frac{\partial w}{\partial \epsilon_{ij}}, \quad w = \frac{1}{2} C_{ijkl} \epsilon_{ij} \epsilon_{kl}$$
By Clairaut's theorem, the order of mixed partial differentiation is irrelevant:
$$\frac{\partial^2w}{\partial \epsilon_{ij} \partial \epsilon_{kl}} = \frac{\partial^2w}{\partial \epsilon_{kl} \partial \epsilon_{ij}} \implies C_{ijkl} = C_{klij}$$
In Voigt form, this requires the stiffness matrix to be perfectly symmetric across its primary diagonal ($C_{mn} = C_{nm}$), leaving a maximum of **21 independent constants**. This state corresponds to the least-symmetric crystal system (**Triclinic**).

---

## Step 3: Crystal Symmetry Reduction Mechanisms

According to **Neumann's Principle**, any physical property of a crystal must possess at least the symmetry of its point group. If a crystal remains invariant under a spatial symmetry operation matrix $[a]$, the transformed elastic constants must identically equal the original values:

$$C'_{ijkl} = \sum_{m,n,p,q} a_{im}a_{jn}a_{kp}a_{lq}C_{mnpq} \equiv C_{ijkl}$$

Let us track how increasing crystal operations systematically eliminate and link constants down across all crystal systems.



---

### 1. Monoclinic System (1 Two-Fold Rotation Axis $\rightarrow$ 13 Independent Constants)
Characterized by a single 2-fold rotation axis ($C_2$) or a mirror plane perpendicular to an axis (traditionally chosen along the $y$ or $z$ axis). Assuming a 2-fold rotation about the $z$-axis ($x_3$), the transformation matrix is:

$$
a = \begin{bmatrix} -1 & 0 & 0 \\ 0 & -1 & 0 \\ 0 & 0 & 1 \end{bmatrix}
$$

Applying $C'_{ijkl} = a_{im}a_{jn}a_{kp}a_{lq}C_{mnpq}$, any component containing an odd number of indices with value `1` or `2` flips sign ($C'_{ijkl} = -C_{ijkl}$). To satisfy Neumann's Principle, these terms must vanish: 

$$C_{14} = C_{15} = C_{24} = C_{25} = C_{34} = C_{35} = C_{46} = C_{56} = 0$$

This cleanly eliminates 8 parameters, leaving **13 independent constants**:

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & C_{16} \\
C_{12} & C_{22} & C_{23} & 0 & 0 & C_{26} \\
C_{13} & C_{23} & C_{33} & 0 & 0 & C_{36} \\
0 & 0 & 0 & C_{44} & C_{45} & 0 \\
0 & 0 & 0 & C_{45} & C_{55} & 0 \\
C_{16} & C_{26} & C_{36} & 0 & 0 & C_{66}
\end{bmatrix}
$$

---

### 2. Orthorhombic System (3 Mutually Perpendicular Two-Fold Axes $\rightarrow$ 9 Independent Constants)
Possesses three mutually perpendicular 2-fold rotation axes. Adding a second independent 2-fold rotation (e.g., about the $y$-axis) introduces an additional transformation step:

$$
a = \begin{bmatrix} -1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & -1 \end{bmatrix}
$$

This second operation eliminates all remaining shear-axial cross-coupling terms containing single-direction dependencies ($C_{16} = C_{26} = C_{36} = C_{45} = 0$), completely separating normal stresses from pure shear planes to yield **9 independent constants**:

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & 0 \\
C_{12} & C_{22} & C_{23} & 0 & 0 & 0 \\
C_{13} & C_{23} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{55} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{66}
\end{bmatrix}
$$

---

### 3. Trigonal System (1 Three-Fold Rotation Axis $\rightarrow$ 6 or 7 Independent Constants)
Characterized by a single 3-fold rotation axis ($C_3$, $120^\circ$ rotation) parallel to the principal axis ($z$). The rotation intermixes coordinates in the $xy$-basal plane, generating dependencies that enforce structural couplings ($C_{11} = C_{22}$, $C_{13} = C_{23}$, $C_{44} = C_{55}$). Depending on whether additional 2-fold axes are present in the point group, it yields either **6 or 7 independent constants**:

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{13} & C_{14} & 0 & 0 \\
C_{12} & C_{11} & C_{13} & -C_{14} & 0 & 0 \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
C_{14} & -C_{14} & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & C_{14} \\
0 & 0 & 0 & 0 & C_{14} & \frac{1}{2}(C_{11} - C_{12})
\end{bmatrix}
$$

---

### 4. Tetragonal System (1 Four-Fold Rotation Axis $\rightarrow$ 6 or 7 Independent Constants)
Introduces a single 4-fold rotation axis ($C_4$) parallel to the $z$-axis ($x_3$). A clockwise $90^\circ$ ($\pi/2$ rad) rotation maps coordinates as $x_1 \rightarrow x_2$ and $x_2 \rightarrow -x_1$:

$$
a = \begin{bmatrix} 0 & 1 & 0 \\ -1 & 0 & 0 \\ 0 & 0 & 1 \end{bmatrix}
$$

Let us compute a transformed tensor component using this mapping to see how variables become linked:

$$
C'_{1111} = a_{12}a_{12}a_{12}a_{12}C_{2222} = (1)(1)(1)(1)C_{2222} = C_{2222}
$$

Because $C'_{1111} = C_{1111}$, it proves $C_{11} = C_{22}$. Tracking this mapping links the variables into symmetry pairs ($C_{11} = C_{22}$, $C_{13} = C_{23}$, $C_{44} = C_{55}$, and $C_{16} = -C_{26}$).

For lower symmetry tetragonal point groups ($4, \bar{4}, 4/m$), $C_{16}$ remains active (**7 independent constants**). For higher tetragonal classes ($4mm, \bar{4}2m, 422, 4/mmm$), perpendicular secondary 2-fold axes force $C_{16} = 0$, dropping the tensor to **6 independent constants**:

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{13} & 0 & 0 & 0 \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{66}
\end{bmatrix}
$$

---

### 5. Hexagonal System (1 Six-Fold Rotation Axis $\rightarrow$ 5 Independent Constants)
Characterized by a single 6-fold rotation axis ($C_6$, $60^\circ$ rotation) along the $z$-axis. This higher order axial rotation induces complete elastodynamic isotropy within the basal $xy$-plane. Along with maintaining the tetragonal mappings ($C_{11} = C_{22}$, $C_{13} = C_{23}$, $C_{44} = C_{55}$), it sets a strict mechanical constraint that forces the pure in-plane shear modulus $C_{66}$ to depend directly on the primary directional expansion terms:

$$
C_{66} = \frac{1}{2}(C_{11} - C_{12})
$$

All remaining off-diagonal elements vanish, leaving exactly **5 independent constants**:

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{13} & 0 & 0 & 0 \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & \frac{1}{2}(C_{11} - C_{12})
\end{bmatrix}
$$

---

### 6. Cubic System (4 Three-Fold Rotation Axes $\rightarrow$ 3 Independent Constants)
The cubic system represents the highest possible crystal symmetry family. It retains the single 4-fold axial operations along the cell edges but introduces **4 independent 3-fold rotation axes ($C_3$) pointing down the internal $[111]$ body diagonals**. 

A counter-clockwise rotation of $120^\circ$ ($2\pi/3$ rad) around a $[111]$ direction cyclically permutes all three spatial axes simultaneously ($x_1 \rightarrow x_2 \rightarrow x_3 \rightarrow x_1$), yielding the transformation matrix:

$$
a = \begin{bmatrix} 0 & 1 & 0 \\ 0 & 0 & 1 \\ 1 & 0 & 0 \end{bmatrix}
$$

Applying this cyclic symmetry across our remaining variables collapses the matrix entirely:

* **Principal Moduli Equivalency:** $$C'_{2222} = a_{23}a_{23}a_{23}a_{23}C_{3333} \implies C_{22} = C_{33}$$
  Combined with our prior tetragonal steps, this locks all three principal axes to a single value: $C_{11} = C_{22} = C_{33}$.
* **Cross-Coupling Moduli Equivalency:**
  $$C'_{1122} = a_{12}a_{12}a_{23}a_{23}C_{2233} \implies C_{12} = C_{23} = C_{13}$$
* **Shear Moduli Equivalency:**
  $$C'_{2323} = a_{23}a_{31}a_{23}a_{31}C_{3131} \implies C_{44} = C_{55} = C_{66}$$

Every cross-coupling term vanishes, leaving exactly **3 independent elastic variables** ($C_{11}, C_{12}, C_{44}$):

$$
\begin{bmatrix}
C_{11} & C_{12} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{12} & C_{11} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{44}
\end{bmatrix}
$$
## Step 4: Extracting the Output from OUTCAR

When your calculation finishes, open your `OUTCAR` file and scroll down to the bottom. VASP evaluates the calculations step-by-step and prints several block segments:

1. **Rigid Ion Component:** Look for `SYMMETRIZED ELASTIC MODULI (kBar)`.
2. **Ionic Relaxation Contribution:** Look for `ELASTIC MODULI CONTR FROM IONIC RELAXATION (kBar)`.
3. **Total Composite Matrix:** The final complete properties grid appears underneath the **`TOTAL ELASTIC MODULI (kBar)`** marker. Always extract your $C_{ij}$ values from this definitive combined block!

> ⚠️ **Unit Conversion Reminder:** VASP records these tensor elements in **kilobars (kBar)**. To convert them to standard publication units of **Gigapascals (GPa)**, divide the printed values by exactly **10** ($10\text{ kBar} = 1\text{ GPa}$).

---

## Step 5: Born Mechanical Stability Checklist

A crystal structure is stable if and only if its total elastic energy variation is strictly positive ($\Delta E > 0$) for any deformation. This means your output matrix must be positive definite. Check your raw values against these inequalities:

### Cubic
* $C_{11} - C_{12} > 0$
* $C_{11} + 2C_{12} > 0$
* $C_{44} > 0$

### Hexagonal
* $C_{11} > |C_{12}|$
* $2C_{13}^2 < C_{33}(C_{11} + C_{12})$
* $C_{44} > 0$

### Tetragonal
* $C_{11} > |C_{12}|$
* $2C_{13}^2 < C_{33}(C_{11} + C_{12})$
* $C_{44} > 0$
* $2C_{16}^2 < C_{66}(C_{11} - C_{12})$

### Orthorhombic
* $C_{11} > 0, \quad C_{11}C_{22} > C_{12}^2$
* $C_{11}C_{22}C_{33} + 2C_{12}C_{13}C_{23} - C_{11}C_{23}^2 - C_{22}C_{13}^2 - C_{33}C_{12}^2 > 0$
* $C_{44} > 0, \quad C_{55} > 0, \quad C_{66} > 0$
