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

## Step 2: The Math of Tensor Reduction (Voigt Notation)

A raw four-rank elasticity tensor ($C_{ijkl}$) in 3D space contains $3^4 = 81$ total elements. Symmetries collapse this complexity down rapidly:

### 1. Stress & Strain Symmetry (81 down to 36)
Because a cell must remain in rotational equilibrium, the stress tensor is symmetric ($\sigma_{ij} = \sigma_{ji}$). Similarly, the infinitesimal strain tensor is defined symmetrically by construction:
$$\epsilon_{ij} = \frac{1}{2}\left( \frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i} \right)$$

### 2. Voigt Contracted Notation
We map double indices $(ij)$ down to single indices running from 1 to 6:

| Tensor Pair $(ij)$ | 11 | 22 | 33 | 23 / 32 | 13 / 31 | 12 / 21 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Voigt Index ($m, n$)** | **1** | **2** | **3** | **4** | **5** | **6** |

This turns our 81 parameters into a $6 \times 6$ matrix ($c_{mn}$).

### 3. Energy Potential Symmetry (36 down to 21)
Because elastic deformation is a conservative thermodynamic process, the internal strain energy density ($w$) yields:
$$\frac{\partial^2w}{\partial \epsilon_{ij} \partial \epsilon_{kl}} = \frac{\partial^2w}{\partial \epsilon_{kl} \partial \epsilon_{ij}} \implies C_{ijkl} = C_{klij}$$
In Voigt form, this makes the matrix perfectly symmetric along its main diagonal ($c_{mn} = c_{nm}$), leaving a maximum of **21 independent constants** for a low-symmetry triclinic crystal.

---

## Step 3: Crystal Symmetries & Matrix Visualizations

As crystal symmetry increases, Neumann's principle requires that the elastic tensor remains invariant under symmetry transformation matrices ($C'_{ijkl} = C_{ijkl}$). This eliminates most cross-coupling terms:

### Cubic System (3 Independent Constants: $C_{11}, C_{12}, C_{44}$)
$$\begin{bmatrix}
C_{11} & C_{12} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{12} & C_{11} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{44}
\end{bmatrix}$$

### Hexagonal System (5 Independent Constants)
Rotational invariance in the basal plane fixes the final shear parameter to a structural relationship: $C_{66} = \frac{1}{2}(C_{11} - C_{12})$.
$$\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{13} & 0 & 0 & 0 \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & \frac{1}{2}(C_{11} - C_{12})
\end{bmatrix}$$

### Tetragonal System (6 Independent Constants)
$$\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & C_{16} \\
C_{12} & C_{11} & C_{13} & 0 & 0 & -C_{16} \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
C_{16} & -C_{16} & 0 & 0 & 0 & C_{66}
\end{bmatrix}$$

---

## Step 4: Extracting the Output from OUTCAR

When your calculation finishes, open your `OUTCAR` file and scroll down to the bottom. VASP evaluates the calculations step-by-step and prints several block segments:

1. **Rigid Ion Component:** Look for `SYMMETRIZED ELASTIC MODULI (kBar)`.
2. **Ionic Relaxation Contribution:** Look for `ELASTIC MODULI CONTR FROM IONIC RELAXATION (kBar)`.
3. **Total Composite Matrix:** The final complete properties grid appears underneath the **`TOTAL ELASTIC MODULI (kBar)`** marker. Always extract your $C_{ij}$ values from this definitive combined block!

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
