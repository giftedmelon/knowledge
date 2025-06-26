---
title: Research Note — StrokeStles
# tags:
#   - Research Note — StrokeStles
---

| **Step** | **Summary** |
|:---:|:---|
| **1. Contour Sampling** |  
Uniformly sample the glyph’s Bézier outline into a closed polyline of points $\{\mathbf x_i\}$.  
字形の Bézier 輪郭を弧長均等にサンプリングし、閉じた多段線点列 $\{\mathbf x_i\}$ を得る。 |
| **2. Medial Axes** |  
Compute interior $M_I$ and exterior $M_E$ axes via Voronoi on the sample points. Each vertex $v$ carries a radius  
$$r(v)=\min_i\|\;v-\mathbf x_i\|\,. $$  
サンプリング点を用いて Voronoi から内骨格 $M_I$ と外骨格 $M_E$ を構成。各頂点 $v$ には半径  
$$r(v)=\min_i\|v-\mathbf x_i\|$$  
を保持。 |
| **3. CSF Detection** |  
1. Take all degree-1 nodes in $M_I$ (convex CSFs) and in $M_E$ with $r\le0.15H$ (concave CSFs).  
2. On each support-segment, rerun local Voronoi to catch missing CSFs below the same radius threshold.  
1. $M_I$ の度=1 ノードを凸 CSF、$M_E$ の度=1 かつ $r\le0.15H$ のノードを凹 CSF として抽出。  
2. 各 support-segment 上で局所 Voronoi を再実行し、同じ半径閾値以下の追加 CSF を補完。 |
| **4. Ligature Construction** |  
For each concave CSF with contour-index interval $[i,j]$:  
- Map each interior node $v$ to its nearest sample $\mathbf x_{k(v)}$, where  
  $$k(v)=\arg\min_m\|\;v-\mathbf x_m\|\,. $$  
- Collect  
  $$V_c=\{v\mid k(v)\in[i,j]\},\quad  
    E_c=\{(u,v)\in E_I\mid u,v\in V_c\}\,. $$  
凹 CSF ごとに接触区間 $[i,j]$ を持ち、内骨格ノード $v$ を最寄点インデックス $k(v)$ に写像。  
支持ノード集合  
$$V_c=\{v\mid k(v)\in[i,j]\},$$  
対応辺集合  
$$E_c=\{(u,v)\in E_I\mid u,v\in V_c\}$$  
によるサブグラフを得る。 |
| **5. Pairing Concavities (Links)** |  
- For each pair $(c_i,c_j)$, form candidate  
  $$\eta=(\mathbf x_{c_i},\,\mathbf x_{c_j}),$$  
  requiring $\eta\subset\Omega$.  
- Compute flow  
  $$\boldsymbol\phi = -(\mathbf n_i+\mathbf n_j),$$  
  and ensure projection  
  $$p=\boldsymbol\phi\cdot\boldsymbol\pi>0$$  
  onto a branch’s protruding direction $\boldsymbol\pi$.  
- Score  
  $$\omega(\eta)
    = \exp\!\bigl(-\tfrac{r_i+r_j}{2r_{\max}}\bigr)
      + \psi(c_i,c_j),$$  
  and greedily select non-conflicting links.  
- 凹 CSF 対 $(c_i,c_j)$ ごとに候補リンク  
  $$\eta=(x_{c_i},x_{c_j})$$  
  を生成。  
- フロー  
  $$\boldsymbol\phi=-(\mathbf n_i+\mathbf n_j)$$  
  を計算し、枝の突き出し方向への射影  
  $$p=\boldsymbol\phi\cdot\boldsymbol\pi>0$$  
  を確認。  
- スコア  
  $$\omega(\eta)
    = e^{-\tfrac{r_i+r_j}{2r_{\max}}}
      + \psi(c_i,c_j)$$  
  で非衝突リンクを贅欲的に選択。 |
| **6. Junction Identification** |  
1. Proto-junctions from compound links.  
2. Half-junctions from link pairs with high $\psi$.  
3. For each remaining fork, enumerate T/Y/L/end/null candidates and compute:  
   - **Coverage**  
     $$\Lambda_I = \ln\frac{A\cap A'}{A},$$  
   - **Smoothness**  
     $$\Lambda_\psi
       = \ln\Bigl(\prod_{m=1}^M\psi_m\Bigr)^{1/M},$$  
   - **Concavity consistency**,  
   - **Link salience** (for T-junction).  
   Compare one-vs-one and pick the best.  
4. Upgrade some T’s to half-junctions if $\psi$ is high.  
1. コンパウンドリンクから Protuberance。  
2. リンク対の高い $\psi$ で Half-junction。  
3. 残りの各フォークで T/Y/L/end/null を列挙し、スコア（coverage $\Lambda_I$、smoothness $\Lambda_\psi$、凹整合性、リンク顕著度）を一対比較し最適を選択。  
4. 一部の T-junction を $\psi$ が高い場合に Half-junction に昇格。 |
| **7. Stroke Reconstruction** |  
- Collect connected branches into a 3D polyline  
  $$P_i=(y_{i,x},\,y_{i,y},\,r_i)\,. $$  
- Smooth each segment with cubic splines, split at endpoints/L-junctions.  
- If line-fit MSE<\(\varepsilon\), use straight spine + constant \(r\).  
- **Final**: smooth spine + width profile for stroking.  
- 3D ポリライン  
  $$P_i=(y_{i,x},y_{i,y},r_i)$$  
  を構成。  
- 端点と L-junction で区切った各セグメントを三次スプラインで平滑化。  
- MSE<$\varepsilon$ の場合は直線＋一定幅で近似。  
- 最終的に smooth spine + width profile として stroking 用に出力。 |
| **8. Stroke Areas** |  
- Build planar map \(Q\) by inserting T/Y/half-junction edges into the original outline.  
- Compute disk area  
  $$D_k=\bigcup_{v\in S_k}\{p:\|p-v\|\le r(v)\}\,. $$  
- Assign faces inside half-junction quads directly.  
- Remaining faces → stroke \(k\) with max \(\mathrm{area}(F\cap D_k)\).  
- Each stroke area = union of its faces.  
- T/Y/half-junction のエッジを元輪郭に挿入し、平面分割図 \(Q\) を構築。  
- 各ストロークの円盤領域  
  $$D_k=\bigcup_{v\in S_k}\{p:\|p-v\|\le r(v)\}$$  
  を計算。  
- half-junction 四辺形内の面を直接割当。  
- 残りの面を \(\max\mathrm{area}(F\cap D_k)\) となるストロークへ割当。  
- 各ストローク領域は割当面の和集合。 |
