# Theoretical Framework: Elastic Constants & Tensor Reduction

Before setting up automated strain calculations in VASP, it is vital to understand the underlying continuum mechanics and crystal symmetries that govern the elastic stiffness tensor. This page covers the mathematical reduction of the fourth-rank elasticity tensor and the Born criteria for mechanical stability.

---

## 1. The Elasticity Tensor ($C_{ijkl}$) and Voigt Notation

Forces acting on atoms within a deformed cell are fundamentally calculated via the **Hellmann–Feynman theorem**:

$$F_I = -\frac{\partial E}{\partial R_I} = -\int n(\mathbf{r})\frac{\partial V_{\text{ext}}}{\partial R_I} d^3r$$

In the regime of linear elasticity, generalized Hooke's Law relates the stress tensor ($\sigma_{ij}$) to the infinitesimal strain tensor ($\epsilon_{kl}$) through the fourth-rank elasticity tensor ($C_{ijkl}$):

$$\sigma_{ij} = \sum_{k,l=1}^{3} C_{ijkl} \epsilon_{kl}$$

Naively, a fourth-rank tensor in 3D space contains $3^4 = 81$ independent components. However, physical conservation laws drastically reduce this number:

1. **Symmetry of the Stress Tensor ($\sigma_{ij} = \sigma_{ji}$):** Enforced by the balance of angular momentum (rotational equilibrium). Reduces components from 81 to 36.
2. **Symmetry of the Strain Tensor ($\epsilon_{ij} = \epsilon_{ji}$):** Defined explicitly as the symmetric part of the displacement gradient:
   $$\epsilon_{ij} = \frac{1}{2}\left( \frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i} \right)$$

### Voigt Notation Mapping
Because both tensors are symmetric, we map the double indices $(ij)$ and $(kl)$ to a single Voigt index running from 1 to 6:

| Tensor Pair $(ij)$ | 11 | 22 | 33 | 23 / 32 | 13 / 31 | 12 / 21 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Voigt Index ($m, n$)** | **1** | **2** | **3** | **4** | **5** | **6** |

This contracts the $9 \times 9$ system into a manageable $6 \times 6$ stiffness matrix ($c_{mn}$).

### The Strain Energy Potential (Reduction to 21 Constants)
Since elastic deformation is conservative, there exists a strain energy density function $w$:
$$\sigma_{ij} = \frac{\partial w}{\partial \epsilon_{ij}}, \quad w = \frac{1}{2} C_{ijkl} \epsilon_{ij} \epsilon_{kl}$$

Because the order of mixed partial derivatives does not matter, the Voigt matrix must be fully symmetric ($c_{mn} = c_{nm}$), leaving a maximum of **21 independent elastic constants** for an completely anisotropic (triclinic) system.

---

## 2. Crystal Symmetries & Matrix Reductions

As crystal symmetry increases, Neumann's principle dictates that the transformed elastic constants must equal the original constants ($C'_{ijkl} = C_{ijkl}$) under a symmetry transformation matrix $a_{ij}$:

$$C'_{ijkl} = \sum_{m,n,p,q} a_{im}a_{jn}a_{kp}a_{lq}C_{mnpq}$$

Below are the definitive $6 \times 6$ elastic stiffness matrices across the major crystal systems:

### Cubic System (3 Independent Constants: $C_{11}, C_{12}, C_{44}$)
Subject to additional 3-fold rotations about the internal $[111]$ body diagonals, all cross-coupling variables vanish:

$$\begin{bmatrix}
C_{11} & C_{12} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{12} & 0 & 0 & 0 \\
C_{12} & C_{12} & C_{11} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{44}
\end{bmatrix}$$

### Hexagonal System (5 Independent Constants)
Characterized by a 6-fold rotation axis. Rotational invariance in the basal plane forces a strict constraint on the final shear term: $C_{66} = \frac{1}{2}(C_{11} - C_{12})$.

$$\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & 0 \\
C_{12} & C_{11} & C_{13} & 0 & 0 & 0 \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
0 & 0 & 0 & 0 & 0 & \frac{1}{2}(C_{11} - C_{12})
\end{bmatrix}$$

### Tetragonal System (6 or 7 Independent Constants)
For ordinary tetragonal systems (point groups $4mm$, $\bar{4}2m$, $422$, $4/mmm$), $C_{16} = 0$, leaving 6 independent constants:

$$\begin{bmatrix}
C_{11} & C_{12} & C_{13} & 0 & 0 & C_{16} \\
C_{12} & C_{11} & C_{13} & 0 & 0 & -C_{16} \\
C_{13} & C_{13} & C_{33} & 0 & 0 & 0 \\
0 & 0 & 0 & C_{44} & 0 & 0 \\
0 & 0 & 0 & 0 & C_{44} & 0 \\
C_{16} & -C_{16} & 0 & 0 & 0 & C_{66}
\end{bmatrix}$$

---

## 3. Born Mechanical Stability Criteria

A crystal structure is stable under the harmonic approximation if the total elastic energy variation is strictly positive ($\Delta E > 0$) for any arbitrary non-zero strain tensor. Mathematically, this requires the $6 \times 6$ elastic stiffness matrix to be **positive definite** (all eigenvalues and principal minors must be greater than zero).



When analyzing your VASP output data, you must explicitly check the following algebraic inequalities based on your crystal type:

### Cubic Crystals
* $C_{11} - C_{12} > 0$
* $C_{11} + 2C_{12} > 0$
* $C_{44} > 0$

### Hexagonal Crystals
* $C_{11} > |C_{12}|$
* $2C_{13}^2 < C_{33}(C_{11} + C_{12})$
* $C_{44} > 0$

### Tetragonal Crystals
* $C_{11} > |C_{12}|$
* $2C_{13}^2 < C_{33}(C_{11} + C_{12})$
* $C_{44} > 0$
* $2C_{16}^2 < C_{66}(C_{11} - C_{12})$

### Orthorhombic Crystals
* $C_{11} > 0, \quad C_{11}C_{22} > C_{12}^2$
* $C_{11}C_{22}C_{33} + 2C_{12}C_{13}C_{23} - C_{11}C_{23}^2 - C_{22}C_{13}^2 - C_{33}C_{12}^2 > 0$
* $C_{44} > 0, \quad C_{55} > 0, \quad C_{66} > 0$
