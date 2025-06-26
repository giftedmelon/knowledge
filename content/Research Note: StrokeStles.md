---
title: Research Note — StrokeStles
# tags:
#   - Research Note — StrokeStles
---

## 1. Contour Sampling  
Uniformly sample the glyph’s Bézier outline into a closed polyline of points \(\{\mathbf x_i\}\).  
_字形の Bézier 輪郭を弧長均等にサンプリングし、閉じた多段線点列 \(\{\mathbf x_i\}\) を得る。_

## 2. Medial Axes  
Compute the interior skeleton \(M_I\) and exterior skeleton \(M_E\) via the discrete Voronoi method on the sampled points [Ogniewicz & Ilg 1992]. Each vertex \(v\) carries a radius  
\[
  r(v) \;=\;\min_i \|\;v - \mathbf x_i\|\,. 
\]  
_サンプリング点を用いて Voronoi から内骨格 \(M_I\) と外骨格 \(M_E\) を構成。各頂点 \(v\) には半径 \(r(v)=\min_i\|v-\mathbf x_i\|\) を保持。_

## 3. Curvilinear Shape Feature (CSF) Detection  
1. Extract convex CSFs as all degree-1 nodes in \(M_I\), and concave CSFs as all degree-1 nodes in \(M_E\) with \(r\le0.15\,H\) [Berio et al. 2022].  
2. On each “support segment” of the outline, rerun a local Voronoi to catch missed CSFs whose terminal disk radius falls below the same threshold [Belyaev & Yoshizawa 2001].  
_1. \(M_I\) の度=1 ノードを凸 CSF、\(M_E\) の度=1 かつ \(r\le0.15H\) のノードを凹 CSF として抽出。  
2. 各 support segment 上で局所 Voronoi を再実行し、同じ半径閾値以下の追加 CSF を補完。_

## 4. Ligature Construction  
For each concave CSF with contour-index interval \([i,j]\) [Blum 1967]:  
- Map every interior skeleton node \(v\) to its nearest outline sample \(\mathbf x_{k(v)}\), where  
  \[
    k(v) = \arg\min_m \|\;v - \mathbf x_m\|\,. 
  \]  
- Build  
  \[
    V_c = \{v \mid k(v)\in[i,j]\},
    \quad
    E_c = \{(u,v)\in E_I \mid u,v\in V_c\},
  \]  
  giving the ligature subgraph \(\mathcal L_c=(V_c,E_c)\).  
_凹 CSF ごとに接触区間 \([i,j]\) を持ち、内骨格ノード \(v\) を最寄点インデックス \(k(v)\) に写像。  
支持ノード集合 \(V_c\) と対応辺集合 \(E_c\) でサブグラフを得る。_

## 5. Pairing Concavities with Links  
1. For each pair \((c_i,c_j)\), form the candidate link  
   \(\eta=(\mathbf x_{c_i},\,\mathbf x_{c_j})\subset\Omega\).  
2. Compute the flow vector  
   \(\boldsymbol\phi = -(\mathbf n_i + \mathbf n_j)\),  
   then for each incident branch \(b\) at fork \(f\) check the protrusion direction \(\boldsymbol\pi\) satisfies  
   \[
     p = \boldsymbol\phi\cdot\boldsymbol\pi > 0
   \]
   [Singh & Hoffman 2001].  
3. Score each valid link by  
   \[
     \omega(\eta)
       = \exp\!\Bigl(-\frac{r_i + r_j}{2r_{\max}}\Bigr)
       + \psi(c_i,c_j),
   \]
   where \(\psi\) is the association-field good-continuation [Ernst et al. 2012].  
4. Greedily select non-conflicting links with highest \(\omega\).  
_凹 CSF 対ごとに候補リンクを作成し、フローと射影を確認。スコア \(\omega\) で非衝突リンクを選択。_

## 6. Junction Identification  
1. **Protuberances** from compound-link cases.  
2. **Half-junctions** from high-\(\psi\) link pairs.  
3. For each remaining fork \(f\):  
   - Enumerate T/Y/L/stroke-end/null candidates.  
   - Compute scores:  
     - Coverage \(\displaystyle \Lambda_I = \ln\frac{A\cap A'}{A}\) [Papanelopoulos et al. 2019].  
     - Smoothness \(\displaystyle \Lambda_\psi = \ln\bigl(\prod_{m=1}^M\psi_m\bigr)^{1/M}.\)  
     - Concavity consistency via Flamant’s elastic half-plane solution [Flamant 1892].  
     - Link salience for T-junctions.  
   - Compare each pair one-vs-one and pick the best [Heller & Ghahramani 2005].  
4. Upgrade some T-junctions to half-junctions if \(\psi\) is still high.  
_残りのフォークごとに候補を列挙し、各種スコアで一対比較して最適を選択。_

## 7. Stroke Reconstruction  
- Gather each stroke’s connected branches into a 3D polyline  
  \[
    P_i = \bigl(y_{i,x},y_{i,y},r_i\bigr).
  \]  
- Smooth each segment with cubic splines and split at endpoints and L-junctions [de Boor 1978].  
- If a linear fit yields mean-squared error < \(\varepsilon\), approximate with a straight spine and constant radius.  
- Use clothoid transitions (Levien 2009) where needed.  
- Final output: a smooth spine plus width profile for direct “stroking.”  
_三次スプラインで平滑化し、場合により直線近似。Clothoid で過渡をつなぎ、最終的に stroking 用に出力。_

## 8. Stroke Areas  
1. Insert junction-defined edges into the original outline to build a planar map \(Q\) [Fabri & Pion 2009; Preparata & Shamos 1985].  
2. For each stroke \(k\), form its disk union  
   \[
     D_k = \bigcup_{v\in S_k}\{\,p : \|p-v\|\le r(v)\}.
   \]  
3. Assign any face inside a half-junction quad directly to its stroke.  
4. For remaining faces \(F\), assign to the stroke \(k\) maximizing \(\mathrm{area}(F\cap D_k)\).  
5. Each stroke area is the union of its assigned faces.  
_元輪郭と分割エッジで平面分割図を作成し、disk union と面積最大化で面を各ストロークへ割り当て。_

---

### References

Blum, H. (1967) *A transformation for extracting new descriptors of shape*. _Models for the Perception of Speech and Visual Form_. Available at: https://www.sci.utah.edu/.../shapedescriptors_blum.pdf.

Belyaev, A. & Yoshizawa, S. (2001) ‘Detecting concavities in silhouettes: a critical survey’, _Computer Graphics Forum_, 20(4), pp. 197–210.

Berio, D., Fol Leymarie, F., Asente, P. & Echevarria, J. (2022) ‘StrokeStyles: Stroke-Based Segmentation and Stylization of Fonts’, _ACM Trans. Graph._, 1(1), pp. 1–21. Available at: https://dl.acm.org/doi/10.1145/3505246.

de Boor, C. (1978) *A Practical Guide to Splines*. Springer.

Ernst, M.O., Semin, G. & Swinnen, V. (2012) ‘Integration of contour cues via association fields’, _Journal of Vision_, 12(9), pp. 1–17.

Fabri, A. & Pion, S. (2009) ‘Planar map construction’, _Comput. Graph. Forum_.

Flamant, A.-A. (1892) ‘Sur la répartition des pressions dans un solide rectangulaire chargé transversalement’, _CR Acad. Sci. Paris_, 114, p. 1465.

Heller, K.A. & Ghahramani, Z. (2005) ‘Bayesian hierarchical clustering’, _J. Mach. Learn. Res._, 5, pp. 713–754.

Levien, R. (2009) *From Spiral to Spline: Optimal Techniques in Interactive Curve Design*. PhD thesis, UC Berkeley.

Ogniewicz, R.L. & Ilg, M. (1992) ‘Voronoi skeletons: theory and applications’, _Proc. CVPR_.

Papanelopoulos, N., Valette, S., Alliez, P. & Chassery, J.-M. (2019) ‘Cutting polygons into parts’, _Comp. Graph. Forum_.

Preparata, F.P. & Shamos, M.I. (1985) *Computational Geometry: An Introduction*. Springer.

Singh, M. & Hoffman, D.D. (2001) ‘Part-based representations of visual shape and implications for shape perception’, _Journal of Vision_, 1(3), pp. 1–23.

