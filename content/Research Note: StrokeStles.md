---
title: Research Note — StrokeStles
# tags:
#   - Research Note — StrokeStles
---

## 1. 輪郭サンプリング  
グリフのベジェ輪郭を弧長均等にサンプリングし、閉じた多段線点列  
\(\{\mathbf x_i\}\) を得る。

## 2. 内部・外部メディアル軸の抽出  
サンプリング点に対して離散的なVoronoi法を用い、内部骨格 \(M_I\) と外部骨格 \(M_E\) を計算する。  
各頂点 \(v\) は次の半径を持つ：  
\[
  r(v) = \min_i \|\;v - \mathbf x_i\|\,. 
\]

## 3. 曲線形状特徴（CSF）の検出  
1. 内部骨格 \(M_I\) 上の次数1ノードを凸CSF、  
   外部骨格 \(M_E\) 上の次数1ノードかつ \(r\le0.15H\) を凹CSFとして抽出する。  
2. 各サポートセグメント上で局所Voronoiを再実行し、同じ半径閾値以下のCSFを補完する。

## 4. リガチュアの構築  
凹CSFごとに輪郭上の接触区間 \([i,j]\) を定め、  
- 内部骨格ノード \(v\) を最寄りの輪郭点 \(\mathbf x_{k(v)}\) に写像する：  
  \[
    k(v) = \arg\min_m \|\;v - \mathbf x_m\|\,. 
  \]  
- 次の部分グラフを作成する：  
  \[
    V_c = \{v \mid k(v)\in[i,j]\},\quad
    E_c = \{(u,v)\in E_I \mid u,v\in V_c\}.
  \]  
これがCSF \(c\) のリガチュア \(\mathcal L_c=(V_c,E_c)\) となる。

## 5. 凹部のペアリング（リンク）  
1. 各凹CSFペア \((c_i,c_j)\) について、候補リンク  
   \(\eta=(\mathbf x_{c_i},\mathbf x_{c_j})\subset\Omega\) を生成する。  
2. フローを計算する：  
   \(\boldsymbol\phi = -(\mathbf n_i + \mathbf n_j)\)。  
   各フォーク \(f\) の分岐 \(b\) に対し、突出方向 \(\boldsymbol\pi\) が  
   \[
     p = \boldsymbol\phi\cdot\boldsymbol\pi > 0
   \]  
   を満たすか確認する。  
3. 有効なリンクを次のように評価する：  
   \[
     \omega(\eta)
       = \exp\!\Bigl(-\frac{r_i + r_j}{2\,r_{\max}}\Bigr)
       + \psi(c_i,c_j),
   \]  
   ここで \(\psi\) はアソシエーションフィールドによるグッドコンティニュエーション。  
4. \(\omega\) が高い順に贅欲的に非衝突リンクを選択する。

## 6. ジャンクションの識別  
1. コンパウンドリンクから Protuberance を生成。  
2. グッドコンティニュエーションが高いリンク対から Half-junction を生成。  
3. 残る各フォーク \(f\) について：  
   - T/Y/L/ストローク末端/null の候補を列挙。  
   - 以下のスコアを計算：  
     - **Coverage**：  
       \(\displaystyle \Lambda_I = \ln\frac{A \cap A'}{A}\).  
     - **Smoothness**：  
       \(\displaystyle \Lambda_\psi = \ln\bigl(\prod_{m=1}^M \psi_m \bigr)^{1/M}.\)  
     - **Concavity consistency**：Flamant の弾性半平面解に基づく。  
     - **Link salience**：T-junction の場合のみ。  
   - 各対を一対一で比較し、最適なジャンクション種類を選択。  
4. 必要に応じて、T-junction を Half-junction に昇格する。

## 7. ストローク再構築  
- 各ストロークの連結分岐を取り、3次元ポリライン  
  \[
    P_i = (\,y_{i,x},\,y_{i,y},\,r_i\,)
  \]  
  としてまとめる。  
- 各セグメントを三次スプラインで平滑化し、端点やL-junctionで分割する。  
- 直線近似誤差（MSE）が閾値 \(\varepsilon\) 未満なら、直線脊線＋一定幅で近似。  
- 必要に応じて Clothoid 過渡を挿入。  
- 最終的に平滑脊線と幅プロファイルを用いてストロークを“stroking”で描画する。

## 8. ストローク領域の構築  
1. 元の輪郭にジャンクションで定義されたエッジを挿入し、平面地図 \(Q\) を作成する。  
2. 各ストローク \(k\) について円盤領域を統合：  
   \[
     D_k = \bigcup_{v\in S_k}\{p : \|p-v\|\le r(v)\}.
   \]  
3. Half-junction 四辺形内の面を直接対応するストロークへ割り当てる。  
4. 残る面 \(F\) について、\(\mathrm{area}(F\cap D_k)\) が最大となるストローク \(k\) に割り当てる。  
5. 各ストローク領域は割り当てられた面の和集合となる。


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

