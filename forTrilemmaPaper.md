## トリレンマの式について

意味あるか分からないけど、論文の内容に加えて以下くらいは言えるはず。

1. 相加相乗平均の不等式から、一般にちょっとだけ条件を付けられる：
    
    $B_{h}, B_{tx}, n_{tx} \geq 0$ なので $B = B_{h} + B_{tx} \cdot n_{tx} \geq 2 \sqrt{B_{h} (B_{tx} \cdot n_{tx})}$ より、
    
    $1 = \frac{B_{h} + B_{tx}}{T} \cdot \frac{1}{F} \cdot \mathbf{H}^T \mathbf{P} \mathbf{H}$
   $\geq \frac{2 \sqrt{B_{h} (B_{tx} \cdot n_{tx})}}{T} \cdot \frac{1}{F} \cdot \mathbf{H}^T \mathbf{P} \mathbf{H}$
   $= \frac{2 \sqrt{B_{h} B_{tx}}}{\sqrt{n_{tx}}} \frac{n_{tx}}{T} \cdot \frac{1}{F} \cdot \mathbf{H}^T \mathbf{P} \mathbf{H}$ 
    
3. コーシー・シュワルツの不等式から、一般にちょっとだけ条件を付けられる：
    
    ベクトルの内積を $\langle {\mathbf{a}, \mathbf{b}} \rangle := \mathbf{a}^T \mathbf{b}$ 、ノルムを $|| \mathbf{a} || = \sqrt{\langle {\mathbf{a}, \mathbf{a}} \rangle}$ とすると $| \langle {\mathbf{a}, \mathbf{b}} \rangle | \leq || \mathbf{a} || || \mathbf{b} ||$ なので、定義から $0 \leq \mathbf{H}^T \mathbf{P} \mathbf{H}$ なのはそのまま使いつつ、たとえば
    
    $0 \leq \mathbf{H}^T \mathbf{P} \mathbf{H} = \mathbf{H}^T (\mathbf{P} \mathbf{H}) \leq ||\mathbf{H}|| || \mathbf{P} \mathbf{H} || = \sqrt{{\sum_{i=0}}^n {H_i}^2} \sqrt{{\sum_{j=0}}^n {\lparen {{\sum_{k=0}}^n t_{jk} H_k} \rparen}^2}$ 
    

## その他思ったこと

- セキュリティをフォーク率と切り離せたらトリレンマを改善できるというのは良い発見だと思った。逆に、スケーラビリティと分散性はこれでほぼ尽きそうな印象
- 3つの積が、単に定数というだけじゃなくて、この定数が1と言っているのがちょっと強めなこと言えてて良い感じ
- $t_{ij}=0$ なのを「Because no block propagates between identical
nodes, these diagonal elements are assigned a value of 0.」と書いてたけど、「自分自身への伝播は時間0で終わる」っていう解釈とする方が自然に思える
- 分散性の定義についてVitalikの定義と比較してたりするけど、同じ地理的範囲とか政治的に分散されていない場合とかは1つのノードとして数える、みたいな「有効ノード数」とか「有効ハッシュレート」を考えることにすれば、今回の議論で言うところの**H**に帰着できる気がする
- $\mathbf{P}$が対称行列なら固有値とかもーちょい言えることがあるけど、ネットワークのトポロジーでこいつを対称行列にするとかできないもんだろうか？グラフ理論あたりで、行列の要素を良い感じに整列できる定理とか無いのかな？
