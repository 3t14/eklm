# Copyright-Aware and Compensable KB Architecture

更新日: 2026-03-28

## 注意

この文書は法的助言ではない。

目的は、著作権リスクを下げつつ、将来的にライセンスや補償の設計根拠を持ちやすい知識アーキテクチャを整理することである。

## 問題

知識を分離したモデルは、重み内部の依拠を減らせる一方で、
`どのソースに依拠したか`
が可視化されやすい。

これは両刃である。

- 良い点:
  provenance が取りやすい
- 危ない点:
  依拠性が高く見えやすい

したがって必要なのは、
`source text`
と
`canonical knowledge`
を分ける設計である。

## 結論

安全で実務向きなのは、

- 原文は原文として別管理
- KB には canonicalized knowledge を入れる
- 実行時には provenance ledger を持つ
- 出力では表現再現リスクを抑える
- 将来的な補償は ledger ベースで設計する

という方式である。

## 1. 3層分離

知識資産は少なくとも次の3層に分けるべきである。

### A. Source Layer

原文そのもの。

例:

- 書籍本文
- 記事本文
- 論文本文
- ドキュメント本文

### B. Canonical Knowledge Layer

原文から抽出、正規化した知識。

例:

- 事実
- 関係
- 定義
- 数式
- 制約
- pattern
- usage signature

### C. Runtime Derivation Layer

実行時に参照、束縛、導出で使う層。

例:

- slot binding candidate
- top-k reference
- rule firing trace
- output provenance

## 2. 何を KB に入れるべきか

比較的強いのは次である。

- 事実
- 関係
- 型
- 数量
- 数式
- 制約
- pattern
- usage signature
- source id

比較的危ないのは次である。

- 長い原文断片
- 一意な比喩や表現
- 文章の選択配列そのもの
- source text をほぼ復元できる中間表現

したがって KB は、
`意味を保持しつつ表現を落とす`
方向で正規化すべきである。

## 3. Provenance Ledger

この方式では、通常の end-to-end LM より provenance を持ちやすい。

最小限、各知識片や runtime decision に対して次を記録できる。

- source id
- extraction method
- confidence
- canonicalization step
- binding contribution
- output contribution

例:

```lisp
(prov P_1
  (source src:book_023)
  (kb_item kb:physics.newton2)
  (step extracted_formula)
  (conf f:0.94))
```

## 4. 補償設計に向く理由

この方式では、

- どの source から canonical item を作ったか
- どの source 群が binding を支えたか
- どの source が runtime で参照されたか

を記録しやすい。

これにより、少なくとも技術的には、

- source ごとの寄与集計
- document family ごとの寄与集計
- runtime usage ベースの按分

が可能になる。

ここでいう按分は、法的義務そのものではなく、
`ライセンス実務や収益分配の根拠候補`
である。

## 5. 寄与率の考え方

寄与率は一つの値ではなく、分けて持つ方がよい。

- extraction contribution
- canonical KB contribution
- runtime retrieval contribution
- binding contribution
- output exposure contribution

つまり、
`何にどれだけ効いたか`
を一つに潰さない。

これにより、

- facts への寄与
- wording への寄与
- final output への寄与

を分けて評価しやすくなる。

## 6. latent を使う場合の考え方

latent 空間を多用すると、寄与は不透明になりやすい。

ただし、この方式では

- hard layer
- soft layer
- provenance ledger

を分けるので、通常の LLM より説明可能性は高い。

自然なのは、

- hard contribution
- soft contribution

を分けて持つことである。

例:

- hard: 明示KBに入った fact / rule / formula
- soft: reranking や disambiguation に効いた latent similarity

この区別は、補償設計でも有用である。

## 7. 出力時の安全設計

出力では、次を避けるべきである。

- 長い source-like passage の再現
- 単一ソースに近すぎる要約
- 原文の表現順序の模倣

代わりに推奨されるのは、

- canonical knowledge からの生成
- copy は entity name や technical label に限定
- quotation は明示的に扱う
- output provenance を持つ

## 8. ライセンス実務として何ができるか

実務的には、次のような設計があり得る。

1. source registry を持つ
2. canonicalization 後の KB item に source linkage を持たせる
3. runtime access log を保存する
4. output exposure score を集計する
5. その集計をライセンス料や revenue share の補助指標にする

この方式なら、
`重みの中で何が起きたか分からない`
よりはるかに管理しやすい。

## 9. 限界

ただし、次は残る。

- 寄与率は法的責任そのものではない
- latent contribution は一意に算出しにくい
- 事実と表現の境界は簡単ではない
- 市場代替性や output similarity は別途見る必要がある

したがって、この方式は
`著作権問題を消す`
のではなく、
`著作権リスクと補償可能性を管理しやすくする`
ものと考えるべきである。

## 10. 最小実装

最小プロトタイプなら、次だけでよい。

1. source id を各 KB item に付ける
2. canonical knowledge と source text を別保存する
3. runtime retrieval log を取る
4. output provenance を保存する
5. 長い source-like output を抑制する

## 一文での整理

この方式の著作権上の強みは、依拠を消すことではなく、`source text と canonical knowledge を分離し、runtime provenance を記録することで、リスク管理と補償設計を可能にすること` にある。
