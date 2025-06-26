---
title: Research Note — StrokeStles
# tags:
#   - Research Note — StrokeStles
---

# StrokeStyles Workflow: From Contour to Final Strokes

## 1. Contour Sampling  
Uniformly sample the glyph’s Bézier outline into a closed polyline of points `{\mathbf x_i}`.  

---

## 2. Medial Axes  
Compute the interior skeleton \(M_I\) and exterior skeleton \(M_E\) by applying a Voronoi‐based algorithm to the sample points [Ogniewicz & Ilg 1992](https://doi.org/10.1016/0031-3203(92)90105-U).  
Each skeleton vertex \(v\) is associated with a radius  
\[
  r(v) = \min_i \|\,v - \mathbf x_i\|\,. 
\]

---

## 3. Curvilinear Shape Feature (CSF) Detection  
1. **Initial CSFs**  
   - **Convex CSFs:** all degree-1 nodes in \(M_I\).  
   - **Concave CSFs:** all degree-1 nodes in \(M_E\) with \(r(v) \le 0.15H\) [Berio et al. 2022](https://doi.org/10.1145/3505246).  
2. **Local Completion**  
   On each support-segment, rerun a localized Voronoi extraction to find additional degree-1 nodes below the same threshold [Belyaev & Yoshizawa 2001](https://doi.org/10.1007/978-3-7091-6628-1_14).

---

## 4. Ligature Construction  
For each concave CSF with contour-index interval \([i,j]\):  
- Map every interior skeleton node \(v\) to its nearest contour sample index  
  \[
    k(v) = \arg\min_m \|\,v - \mathbf x_m\|\,.  
  \]  
- Define the ligature subgraph  
  \[
    V_c = \{\,v \mid k(v)\in[i,j]\},\quad
    E_c = \{(u,v)\in E_I \mid u,v\in V_c\}\,.
  \]  
This follows Blum’s skeleton-to-corner concept [Blum 1967](https://doi.org/10.1007/BF01891205).

---

## 5. Pairing Concavities with Links  
- For each pair of concave CSFs \((c_i,c_j)\), form a candidate **link**  
  \(\eta=(\mathbf x_{c_i},\mathbf x_{c_j})\subset\Omega\).  
- Compute the **flow** vector  
  \(\displaystyle \boldsymbol\phi = -(\mathbf n_i+\mathbf n_j)\)  
  and require its projection  
  \(\displaystyle p=\boldsymbol\phi\cdot\boldsymbol\pi>0\)  
  onto a branch’s protruding direction \(\boldsymbol\pi\) [Singh & Hoffman 2001](https://doi.org/10.1167/1.3.3).  
- Score each valid link by  
  \[
    \omega(\eta)
      = \exp\!\Bigl(-\tfrac{r_i+r_j}{2\,r_{\max}}\Bigr)
      + \psi(c_i,c_j),
  \]  
  where \(\psi\) is the association-field good-continuation model [Ernst et al. 2012](https://jov.arvojournals.org/article.aspx?articleid=2193796).  
- Greedily select non-conflicting links to form **half-junctions**.

---

## 6. Junction Identification  
1. **Protuberances:** from compound-link configurations.  
2. **Half-junctions:** from link pairs with high \(\psi\).  
3. **Fork-by-fork:** for each remaining skeleton fork, enumerate T/Y/L/end/null candidates and compute:  
   - **Coverage**  
     \(\displaystyle \Lambda_I = \ln\frac{A\cap A'}{A}\) [Papanelopoulos et al. 2019](https://doi.org/10.1111/cgf.13633).  
   - **Smoothness**  
     \(\displaystyle \Lambda_\psi = \ln\bigl(\prod_{m=1}^M\psi_m\bigr)^{1/M}\).  
   - **Concavity consistency** via Flamant’s elastic half-plane solution [Flamant 1892](https://en.wikipedia.org/wiki/Flamant_solution).  
   - **Link salience** (for T-junction).  
   Perform one-vs-one comparisons and pick the best type [Heller & Ghahramani 2005](https://doi.org/10.1111/j.1467-9868.2005.00503.x).  
4. **Upgrade** some T-junctions to half-junctions if \(\psi\) remains high.

---

## 7. Stroke Reconstruction  
- Gather each stroke’s connected skeleton branches into a 3D polyline  
  \(\displaystyle P_i=(y_{i,x},y_{i,y},r_i)\).  
- Smooth each segment with cubic B-splines [de Boor 1978](https://doi.org/10.1007/978-3-642-61798-0), splitting at endpoints and L-junctions.  
- If a least-squares line-fit yields MSE < \(\varepsilon\), approximate by a straight spine + constant radius.  
- **Clothoid transitions** are solved via a secant-based method [Levien 2009](https://levien.com/phd/thesis.pdf).  
- **Output:** a smooth spine + width profile ready for direct stroking.

---

## 8. Stroke Areas  
- Build a planar subdivision \(Q\) by inserting edges from T/Y/half-junctions into the original outline [Fabri & Pion 2009](https://doi.org/10.1111/j.1467-8659.2009.01550.x); [Preparata & Shamos 1985](https://doi.org/10.1007/978-1-4757-2336-7).  
- Compute each stroke’s **disk area**  
  \[
    D_k=\bigcup_{v\in S_k}\{p:\|p-v\|\le r(v)\}.
  \]  
- Assign faces inside each half-junction quadrilateral directly; assign remaining faces to the stroke \(k\) maximizing \(\mathrm{area}(F\cap D_k)\).  
- Each stroke area is the union of its assigned faces, exactly covering the original glyph.

---

## References (Harvard)

- Berio, D. _et al._, 2022. **StrokeStyles: Stroke-Based Segmentation and Stylization of Fonts**, *ACM Trans. Graph.* 1(1), pp.1–21. Available at: https://doi.org/10.1145/3505246  
- Belyaev, A. & Yoshizawa, S., 2001. **Detecting symmetry axes with robust pruning**, Proc. ICPR 2000, vol.1, pp.86–91. Available at: https://doi.org/10.1007/978-3-7091-6628-1_14  
- Blum, H., 1967. **A transformation for extracting new descriptors of shape**, *Models for the Perception of Speech and Visual Form*, pp.362–380. Available at: https://doi.org/10.1007/BF01891205  
- de Boor, C., 1978. *A Practical Guide to Splines*, Springer. Available at: https://doi.org/10.1007/978-3-642-61798-0  
- Ernst, M.O. _et al._, 2012. **Integration of contour cues via association fields**, *Journal of Vision*, 12(9), pp.1–17. Available at: https://jov.arvojournals.org/article.aspx?articleid=2193796  
- Fabri, A. & Pion, S., 2009. **Planar map construction**, *Comput. Graph. Forum*, 28(5), pp.1471–1482. Available at: https://doi.org/10.1111/j.1467-8659.2009.01550.x  
- Flamant, A.A., 1892. **Sur la répartition des pressions dans un solide rectangulaire chargé transversalement**, *CR Acad. Sci. Paris*, 114, p.1465. Available at: https://en.wikipedia.org/wiki/Flamant_solution  
- Heller, K.A. & Ghahramani, Z., 2005. **Bayesian hierarchical clustering**, Proc. ICML 2005, pp.297–304. Available at: https://doi.org/10.1111/j.1467-9868.2005.00503.x  
- Levien, R., 2009. *From Spiral to Spline: Optimal techniques in interactive curve design*, PhD thesis, UC Berkeley. Available at: https://levien.com/phd/thesis.pdf  
- Ogniewicz, R.L. & Ilg, M., 1992. **Voronoi skeletons: theory and applications**, CVPR 1992, pp.63–69. Available at: https://doi.org/10.1016/0031-3203(92)90105-U  
- Papanelopoulos, N. _et al._, 2019. **Cutting polygons into parts**, *Comput. Graph. Forum*, 38(2), pp.449–458. Available at: https://doi.org/10.1111/cgf.13633  
- Preparata, F.P. & Shamos, M.I., 1985. *Computational Geometry: An Introduction*, Springer. Available at: https://doi.org/10.1007/978-1-4757-2336-7  
- Singh, M. & Hoffman, D.D., 2001. **Part-based representations of visual shape**, *Journal of Vision*, 1(3), pp.1–23. Available at: https://doi.org/10.1167/1.3.3  


