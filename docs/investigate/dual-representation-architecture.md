# Dual Representation Architecture

更新日: 2026-03-28

## 要点

この方式では、関係性を完全に明示構造だけで持つのは難しい。

理由:

- 単語や記号の意味は関係の中で決まる
- メタ的関係は明示タグだけでは表しきれない
- 類似性、含意、談話ニュアンス、比喩は連続表現が必要になる

したがって、自然な設計は
`hard layer + soft layer`
の二層表現である。

## 基本原理

一言で言うと、

`structure-first, but not structure-only`

である。

つまり、

- 骨格は明示的な関係構造で持つ
- 取り切れない意味差は潜在空間で補う

という方針を採る。

## 1. Hard Layer

Hard layer は、共通IRの骨格である。

ここでは少なくとも次を明示する。

- typed slot
- relation
- event
- entity
- formula
- pattern
- statement
- document block
- meta relation
- choice / unknown / unresolved

例:

- `PERSON_SLOT_1`
- `TERM_SLOT_2`
- `VAR_SLOT_3`
- `rel(agent, EVENT_1, PERSON_SLOT_1)`
- `rel(theme, EVENT_1, TERM_SLOT_2)`
- `meta(certainty, REL_1, 0.72)`

Hard layer の役割は、

- 正規化
- 検証
- 比較
- 検索
- 制約処理
- 知識参照

を安く安定して行うことにある。

## 2. Soft Layer

Soft layer は、Hard layer の各要素に付随する潜在表現である。

持たせる対象:

- slot embedding
- relation embedding
- statement embedding
- block embedding
- document embedding

役割:

- 微妙な語義差の保持
- 類似用法の近接
- 談話レベルの含意
- メタ的関係のにじみ
- 未知語の文脈依存意味の保持
- 明示タグに落ちない差分の保存

つまり soft layer は、
`hard layer では失われるが、token 列だけに戻したくない情報`
を保持する層である。

## 3. スロットと関係の両方に潜在表現を持たせる

潜在空間は token にだけ付けるのでは足りない。

必要なのは、

- スロットの潜在表現
- 関係の潜在表現
- 文や節の潜在表現

である。

理由:

- 単語の意味は、その単語単体ではなく関係束の中で決まる
- メタ的意味は節や談話単位で現れる
- 同じ関係ラベルでも文脈により意味がずれる

したがって最小単位は `token embedding` ではなく、
`typed slot graph embedding`
に近い。

## 4. メタ的関係の扱い

Transformer が暗黙に学んでいたメタ的関係には、少なくとも次がある。

- 確信度
- 対比
- 引用
- 説明
- 否定のスコープ
- 話者態度
- 談話遷移
- 優先順位

これらは、完全に hard layer に押し込むと重くなりすぎる。

そこで、

- 重要で安定したメタ関係は hard layer へ
- 微妙なニュアンスは soft layer へ

と分けるのがよい。

## 5. 関係の再ification

メタ関係を扱うには、関係自体をオブジェクト化する必要がある。

つまり、

- ノードに ID を振る
- 関係にも ID を振る
- 文や節にも ID を振る
- その上にさらに関係を張る

必要がある。

例:

```lisp
(rel REL_1
  (pred buy)
  (arg agent PERSON_SLOT_1)
  (arg theme TERM_SLOT_1))

(meta META_1
  (type certainty)
  (about REL_1)
  (value f:0.72))

(meta META_2
  (type source)
  (about REL_1)
  (value SPEAKER_SLOT_1))
```

この設計により、

- 関係について語れる
- 関係同士を比較できる
- メタ解釈を階層化できる

ようになる。

## 6. 束縛は symbolic constraints と latent similarity の積で行う

未知語や未知記号を知識部へ束縛するとき、hard layer だけでは不十分である。

自然なのは、

`binding score = symbolic compatibility × latent similarity × context fit`

という考え方である。

ここで見るものは、

- 型が合うか
- 関係署名が合うか
- 単位や項構造が合うか
- 周辺の潜在ベクトルが近いか
- 文書文脈に合うか

である。

つまり知識接続は、
`記号的一致`
でも
`ベクトル類似`
でもなく、
その両方で決める。

## 7. この設計が解決すること

この二層設計にすると、次の問題を同時に緩和できる。

### 抽象化で意味が落ちる問題

内容語を抽象スロット化しても、

- 関係骨格
- 用法署名
- 潜在表現

を残せる。

### メタ的関係が消える問題

明示できるメタ関係は hard layer に置き、
それ以外は soft layer に残せる。

### Transformer 的柔軟性を失う問題

IR だけに閉じず、連続表現を sidecar として残せる。

## 8. コスト上の意味

この設計は「全部を潜在空間で持つ」より安く、
「全部を記号で持つ」より柔軟である。

安くなる理由:

- 検証、検索、制約処理は hard layer で行える
- 高価な推論は soft layer が必要な部分だけに限定できる
- 小型モデルの役割を束縛、解釈、再表現に絞れる

つまり、
`潜在空間を捨てる` のではなく、
`潜在空間を正しい場所へ押し込める`
設計である。

## 9. 最小実装の形

最小プロトタイプでは、次だけでよい。

1. 文から typed slot graph を作る
2. 各 slot と relation に小さな embedding を付ける
3. binding を hard constraints と latent similarity の積で行う
4. unresolved のまま保持できるようにする

この形でも、

- 関係ベース意味
- 抽象化と意味保持の両立
- メタ関係の部分的保持

は試せる。

## 10. 一文での定義

この方式の表現部は、

`typed slot graph を骨格とし、その各要素に潜在表現を付与した dual representation`

として設計するのが自然である。
