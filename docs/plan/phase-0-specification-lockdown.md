# Phase 0 Specification Lockdown

更新日: 2026-03-28

## 目的

Phase 0 の目的は、実装に入る前に「何を作るか」を曖昧さなく固定することである。

この段階では、性能改善よりも次を優先する。

- 用語の固定
- IR 仕様の固定
- スロット化方針の固定
- 語彙接地方針の固定
- 失敗の表現の固定
- 評価タスクの固定
- 受入条件の固定

つまり Phase 0 は、研究の設計凍結である。

## 1. 凍結対象

### 1.1 用語

以下の用語の意味を固定する。

- 表現部
- 知識部
- 導出部
- 共通IR
- Hard Layer
- Soft Layer
- lexical grounding
- typed slot
- binding
- unresolved
- provenance
- canonical knowledge

### 1.2 中核アーキテクチャ

以下の関係を固定する。

- 表現部は入力を共通IRへ写像する
- 知識部は canonical knowledge を保持する
- 導出部は regex / formula / rule / lookup を処理する
- Soft Layer は Hard Layer の残差を補う
- lexical grounding はスロット化後の意味接地を担う

### 1.3 評価対象

初期評価タスクを狭く切る。

候補:

- 数式の自然文 <-> 形式表現変換
- regex / pattern の自然文 <-> 形式表現変換
- Markdown 混在文書の構造抽出
- 仕様文からの constraint extraction
- 小規模な entity / relation grounding

## 2. Phase 0 の成果物

Phase 0 で最低限作るべき成果物は次の通りである。

1. 共通IR最小スキーマ
2. typed slot graph スキーマ
3. lexical grounding スキーマ
4. provenance schema
5. validator ルール一覧
6. 初期ベンチマーク定義
7. baseline 定義
8. 失敗分類表
9. 受入条件

## 3. 作業パッケージ

### WP0-1: 用語と境界の固定

#### 目的

研究範囲と用語を固定し、以降の文書・実装で解釈がぶれないようにする。

#### 作業

- 用語集を作る
- 「何をしないか」を明記する
- 汎用会話代替を目標にしないことを明文化する
- `IR = Intermediate Representation` を再確認する

#### 成果物

- terminology glossary
- non-goals list
- scope statement

#### 受入条件

- 文書間で意味がぶれない
- `knowledge`, `slot`, `grounding`, `provenance` の使い方が一貫する

### WP0-2: 共通IR最小スキーマの固定

#### 目的

Phase 1 以降の実装で使う最小 IR を固定する。

#### 作業

- 必須タグを列挙する
- role の集合を固定する
- uncertainty fields を固定する
- module/import の考え方を固定する
- canonical serialization を決める

#### 成果物

- IR schema v0.1
- tag list
- role list
- uncertainty model
- serialization rule

#### 受入条件

- parser が実装可能
- validator が実装可能
- 全体が小さく閉じている

### WP0-3: Typed Slot Graph の固定

#### 目的

スロット化の最小原理を固定する。

#### 作業

- slot type を定義する
- surface retention を定義する
- relation graph の書式を定義する
- temporary slot と persistent slot を区別する
- shared reference の方針を決める

#### 成果物

- typed slot graph schema
- temporary / persistent slot policy
- relation label list

#### 受入条件

- 固有名詞、専門語、変数を表現できる
- 1 回しか出ない局所構造を inline で持てる
- 繰り返し構造を ref 化できる

### WP0-4: Lexical Grounding の固定

#### 目的

匿名化の失敗を避けるために、語彙接地の最小仕様を決める。

#### 作業

- surface を保持する
- type guess を持つ
- candidate bindings を持つ
- binding state を持つ
- copy / canonical name policy を決める

#### 成果物

- lexical grounding spec
- binding state enum
- copy/render policy

#### 受入条件

- `TERM_SLOT` だけで潰れない
- 未知語が `unresolved` で残せる
- 出力時に copy と canonical name を選べる

### WP0-5: Hard/Soft 分離方針の固定

#### 目的

Soft Layer をどこに入れるかを固定する。

#### 作業

- hard layer で明示するものを定義する
- soft layer で残すものを定義する
- slot / relation / statement / block の embedding 粒度を決める
- Soft Layer の学習目標の候補を絞る

#### 成果物

- dual representation spec
- soft layer objective list

#### 受入条件

- hard/soft の責務が衝突しない
- soft layer が「何でも入れる箱」にならない

### WP0-6: Provenance とライセンス前提の固定

#### 目的

後段の補償設計と監査に必要な記録項目を固定する。

#### 作業

- source id を定義する
- extraction step を定義する
- binding contribution を定義する
- output contribution を定義する
- canonical knowledge と source text の分離を確認する

#### 成果物

- provenance schema
- runtime log schema
- source registry schema

#### 受入条件

- どの source が効いたか追える
- source-like output を抑制できる

### WP0-7: ベースラインとベンチマーク固定

#### 目的

Phase 1 で何と比べるかを決める。

#### 作業

- baseline 候補を決める
- 初期タスクを決める
- 指標を決める
- gold / silver / bronze の層を決める

#### 成果物

- benchmark spec
- baseline spec
- metric spec

#### 受入条件

- 研究成果の比較ができる
- 後から評価指標がぶれない

## 4. 実施順序

Phase 0 は、以下の順で進めるのがよい。

1. 用語と非目標を固定する
2. 共通IR最小スキーマを固定する
3. typed slot graph を固定する
4. lexical grounding を固定する
5. hard/soft 分離を固定する
6. provenance を固定する
7. benchmark と baseline を固定する

この順序を崩すと、後で文書間の整合が崩れやすい。

## 5. Phase 0 の完了条件

Phase 0 の完了条件は、次の 5 つである。

1. 実装者が IR を迷わず書ける
2. validator の責務が明確である
3. lexical grounding の最低限の入出力が明確である
4. benchmark と baseline が決まっている
5. Phase 1 の実装チケットがそのまま切れる

## 6. Phase 1 へ進む前の確認項目

Phase 1 に進む前に、次を満たすべきである。

- 共通IR の必須タグが 20 個以内に収まっている
- binding state の enum が小さい
- provenance schema が実装可能である
- 初期タスクが 3 つ以上定義されている
- baseline が最低 2 本ある

## 7. 実装チケット例

### チケット例 1

`IR schema v0.1 を YAML / JSON / S-expression のどれで定義するか決める`

### チケット例 2

`typed slot graph の node / edge / ref の最小仕様を決める`

### チケット例 3

`surface / type / binding state を持つ lexical slot record を定義する`

### チケット例 4

`provenance ledger の項目を定義する`

### チケット例 5

`最初の 3 タスクの benchmark spec を書く`

## 8. 一文での整理

Phase 0 は、研究を始めるための準備ではなく、
`Phase 1 以降の実装と評価がぶれないように、構造・用語・評価・ provenance を凍結する工程`
である。
