# 共通IR最小タグ仕様 v0.1

更新日: 2026-03-28

## 用語

この文書での `IR` は `Intermediate Representation` の意味である。`Information Retrieval` ではない。

Retrieval はこの IR を単位として知識を引く処理であり、IR 自体は「共有意味表現」である。

## 目的

この仕様の目的は、全LMで共有できる最小の共通表現を定めることにある。

必要条件:

- 非Turing完全
- token 化しやすい
- 機械検証しやすい
- 数式と正規表現を中核として持てる
- 曖昧性を保持できる
- 外部知識参照を持てる
- 小さいモデルでも扱える

この仕様は「表現部」のためのものであり、「導出部」の実行仕様ではない。

## 設計原則

1. syntax は極力小さくする
2. semantic role は明示する
3. infix は保存形式に使わない
4. 曖昧性は消さずに表現する
5. 数式とパターンを一級市民にする
6. 実行は参照先へ外出しする

## 最小構文

保存形式は S-expression 風の prefix 表現とする。

```lisp
<node> ::= <atom>
         | (<tag> <field>*)

<field> ::= <atom>
          | (<role> <node>)
```

例:

```lisp
(assert
  (eq
    (quantity (id sym:force) (unit unit:newton))
    (mul
      (quantity (id sym:mass) (unit unit:kg))
      (quantity (id sym:acceleration) (unit unit:m_per_s2))))))
```

$$
F = ma
$$

## 構成

共通IRは次の5群で構成する。

1. 文レベルタグ
2. 意味タグ
3. role タグ
4. 演算タグ
5. メタタグ

## 1. 文レベルタグ

最小集合:

```text
doc
stmt
assert
ask
define
refer
```

意味:

- `doc`: 文書単位
- `stmt`: 文や発話の単位
- `assert`: 事実、主張、法則の表明
- `ask`: 問い合わせ
- `define`: 用語、式、規則、参照先の定義
- `refer`: 曖昧参照、共参照、指示対象の表現

## 2. 意味タグ

最小集合:

```text
entity
event
relation
quantity
formula
constraint
pattern
choice
ref
```

意味:

- `entity`: 物、人、概念、対象
- `event`: 時間を伴う出来事
- `relation`: 述語関係
- `quantity`: 数量や変数
- `formula`: 数式や論理式のまとまり
- `constraint`: 制約
- `pattern`: 正規表現や有限パターン
- `choice`: 曖昧候補集合
- `ref`: 外部知識や外部手続き参照

## 3. role タグ

最小集合:

```text
id
pred
arg
role
value
unit
time
place
cause
condition
surface
target
source
via
conf
status
text
```

意味:

- `id`: ノード識別子
- `pred`: 述語名
- `arg`: 引数
- `role`: 引数役割名
- `value`: 値
- `unit`: 単位
- `time`: 時間
- `place`: 場所
- `cause`: 原因
- `condition`: 条件
- `surface`: 表層文字列
- `target`: 参照先対象
- `source`: 出典元
- `via`: 適用規則や経路
- `conf`: 信頼度
- `status`: 解決状態
- `text`: 補助テキスト

## 4. 演算タグ

### 4.1 数式・比較

```text
eq
neq
lt
le
gt
ge
add
sub
mul
div
pow
neg
```

### 4.2 論理

```text
and
or
not
implies
iff
```

### 4.3 パターン

```text
match
concat
union
repeat
optional
class
anchor_start
anchor_end
```

意図:

- 数式は `eq/add/mul/...` で表す
- 正規表現は raw string ではなく `pattern` ノード配下に構造化できる

## 5. メタタグ

最小集合:

```text
alt
unknown
unresolved
kb
proc
rule
theorem
```

意味:

- `alt`: 候補
- `unknown`: 値や対象が未定
- `unresolved`: 曖昧性が未解決
- `kb`: 知識ベース参照
- `proc`: 外部導出や実行参照
- `rule`: 規則参照
- `theorem`: 定理参照

## 原子 token

### namespace 付き atom を使う

最小 namespace:

```text
ent:
sym:
unit:
kb:
proc:
rule:
theorem:
s:
i:
q:
f:
```

例:

```text
ent:john
sym:force
unit:newton
kb:physics.newton2
proc:algebra.solve_linear
s:"bank"
i:3
q:22/7
f:0.62
```

推奨:

- entity は `ent:*`
- 数式記号は `sym:*`
- 単位は `unit:*`
- 文字列は `s:"..."`
- 整数は `i:*`
- 有理数は `q:*`
- 信頼度は `f:*`

## 正規化規則

共通IRは canonical に比較できなければならない。最小規則は次である。

1. 保存形式は常に prefix
2. infix は authoring 時のみ許可
3. role の並び順は固定する
4. `choice` 内の `alt` は `conf` 降順
5. 数式の自明な単位正規化は保存前に行う
6. 同一ノード内の重複 role は禁止
7. `unknown` と `unresolved` は明示的に保持する

推奨 role 順序:

```text
id pred role arg value unit time place cause condition target source via surface text conf status
```

## 最小バリデーション規則

### 文レベル

- `doc` は 0 個以上の `stmt` を持つ
- `stmt` は主タグを 1 個持つ
- 主タグは `assert`, `ask`, `define`, `refer` のいずれか

### event / relation

- `event` と `relation` は `pred` を 1 個持つ
- `arg` は 0 個以上
- `role` は `arg` と併用可能

### quantity

- `quantity` は `id` または `value` のどちらかを持つ
- `unit` は任意

### choice

- `choice` は 1 個以上の `alt` を持つ
- 各 `alt` は `conf` を持ってよい
- 合計を 1 に正規化するかどうかは runtime policy に任せる

### ref

- `ref` は `target` を 1 個持つ
- `target` は `kb:*`, `proc:*`, `rule:*`, `theorem:*` のいずれか

### pattern

- `pattern` は `match`, `concat`, `union`, `repeat`, `optional`, `class` のいずれかを内部に持つ

## 最小例

### 1. 数式法則

```lisp
(stmt
  (assert
    (eq
      (quantity (id sym:force) (unit unit:newton))
      (mul
        (quantity (id sym:mass) (unit unit:kg))
        (quantity (id sym:acceleration) (unit unit:m_per_s2))))))
```

$$
F = ma
$$

### 2. 正規表現パターン

```lisp
(stmt
  (define
    (pattern
      (id sym:postal_code_jp)
      (concat
        (repeat (class s:"[0-9]") (value i:3))
        (match s:"-")
        (repeat (class s:"[0-9]") (value i:4))))))
```

対応する表層パターン:

$$
\texttt{[0-9]\{3\}-[0-9]\{4\}}
$$

### 3. 曖昧参照

```lisp
(stmt
  (refer
    (surface s:"bank")
    (choice
      (alt (entity (id ent:finance_bank)) (conf f:0.62))
      (alt (entity (id ent:river_bank)) (conf f:0.38))
      (status unresolved))))
```

### 4. 知識参照

```lisp
(stmt
  (ask
    (ref
      (target kb:physics.newton2)
      (via rule:force_from_mass_and_acceleration))))
```

### 5. 外部導出参照

```lisp
(stmt
  (ask
    (ref
      (target proc:algebra.solve_linear))))
```

## 推奨される最小実装

最初の版では、次だけ実装すればよい。

1. parser
2. validator
3. canonicalizer
4. `assert/ask/define/refer` の4系統
5. 数式演算タグ
6. `pattern` とその内部演算
7. `choice/alt/conf/status`
8. `ref/target/via`

## この仕様が狙うもの

この最小仕様は、万能な完全意味論ではなく、次を両立させるための核である。

- 全LMで共有できること
- 低コストで処理できること
- 数式と正規表現を中心に据えられること
- 曖昧性を保持できること
- 知識部と導出部を分離できること

一言で言えば、これは「Information Retrieval のためのIR」ではなく、「Knowledge-aware language processing のための Intermediate Representation」である。
