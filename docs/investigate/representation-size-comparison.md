# 表現形式ごとのサイズ比較

更新日: 2026-03-28

## 目的

この文書は、同じ共通表現を

1. Lisp 風 S-expression
2. JSON AST
3. binary AST
4. slot/value table

で保持したとき、どの程度の情報量になるかを概算するためのものである。

ここでの比較は、`共通IR最小タグ仕様 v0.1` のサンプルを対象にしたラフな見積もりであり、厳密な圧縮率ではない。

## 前提

### 共通前提

- 対象サンプルは [common-ir-minimal-tag-spec.md](/Users/3t14/dev/codex/eklm/docs/investigate/common-ir-minimal-tag-spec.md) の最小例を使う
- ASCII のみを使うので、文字数と UTF-8 byte 数は同じとみなす
- ここでの `IR` は `Intermediate Representation`

### Lisp token 列の前提

- `(` と `)` は各 1 token
- `assert`, `quantity`, `eq`, `ref` などのタグは各 1 token
- `sym:force`, `kb:physics.newton2`, `i:3` などの atom も、あらかじめ ID 化されていて各 1 token
- 16-bit token ID なら 1 token = 2 byte
- 32-bit token ID なら 1 token = 4 byte

この前提はかなり有利である。atom が分割 token になるなら、Lisp 方式はさらに重くなる。

### JSON AST の前提

比較のため、かなりコンパクトな JSON を仮定する。

```json
{"t":"stmt","c":[...]}
```

つまり、

- `t` = tag
- `c` = children

だけを使う最小 JSON AST とする。それでも括弧より重い。

### binary AST の前提

汎用的な prefix binary AST を仮定する。

- list node: `kind(1) + tag_id(2) + arity(1)` = 4 byte
- atom node: `kind(1) + atom_id(2)` = 3 byte

共有 vocabulary は外部にあり、サンプルごとに tag 名や atom 名の文字列は持たないとする。

### slot/value table の前提

汎用的なフラット表を仮定する。

- list node row: `node_id(2) + tag_id(2) + flags(1) + arity(1)` = 6 byte
- edge row: `parent_id(2) + slot_id(1) + kind(1) + target_id(2)` = 6 byte

つまり、

- ノード本体は 1 row
- 親子関係や slot は別 row

で表す。

これは汎用性は高いが、軽量さだけを見ると binary AST より不利である。

## 対象サンプル

- 数式法則 `F = ma`
- 正規表現パターン
- 曖昧参照 `bank`
- 知識参照
- 外部導出参照

## 結果

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

| 形式 | 概算サイズ |
| --- | ---: |
| Lisp text | 199 B |
| Lisp token IDs, 16-bit | 90 B |
| Lisp token IDs, 32-bit | 180 B |
| JSON AST | 333 B |
| binary AST | 70 B |
| slot/value table | 186 B |

補足:

- token 数は 45
- そのうち括弧 token は 26
- 構造 token 比率は約 58%

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

$$
\texttt{[0-9]\{3\}-[0-9]\{4\}}
$$

| 形式 | 概算サイズ |
| --- | ---: |
| Lisp text | 192 B |
| Lisp token IDs, 16-bit | 84 B |
| Lisp token IDs, 32-bit | 168 B |
| JSON AST | 312 B |
| binary AST | 66 B |
| slot/value table | 174 B |

補足:

- token 数は 42
- そのうち括弧 token は 24
- 構造 token 比率は約 57%

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

| 形式 | 概算サイズ |
| --- | ---: |
| Lisp text | 190 B |
| Lisp token IDs, 16-bit | 90 B |
| Lisp token IDs, 32-bit | 180 B |
| JSON AST | 332 B |
| binary AST | 70 B |
| slot/value table | 186 B |

補足:

- token 数は 45
- そのうち括弧 token は 26
- 構造 token 比率は約 58%

### 4. 知識参照

```lisp
(stmt
  (ask
    (ref
      (target kb:physics.newton2)
      (via rule:force_from_mass_and_acceleration))))
```

| 形式 | 概算サイズ |
| --- | ---: |
| Lisp text | 108 B |
| Lisp token IDs, 16-bit | 34 B |
| Lisp token IDs, 32-bit | 68 B |
| JSON AST | 154 B |
| binary AST | 26 B |
| slot/value table | 66 B |

### 5. 外部導出参照

```lisp
(stmt
  (ask
    (ref
      (target proc:algebra.solve_linear))))
```

| 形式 | 概算サイズ |
| --- | ---: |
| Lisp text | 65 B |
| Lisp token IDs, 16-bit | 26 B |
| Lisp token IDs, 32-bit | 52 B |
| JSON AST | 103 B |
| binary AST | 19 B |
| slot/value table | 48 B |

## 傾向

### JSON AST は最も重い

可読性とツール互換性は高いが、quotes, commas, keys が入るので重い。コンパクト JSON でも Lisp text より大きくなる。

### Lisp token 列は中間

予約語と atom を 1 token 化できるなら、かなり小さい。ただし括弧 token が多く、構造オーバーヘッドが大きい。今回のサンプルでは 57% から 62% が構造 token だった。

### binary AST が最も軽い

タグ ID、atom ID、arity だけで木構造を復元できるので、括弧や quotes が不要になる。そのため、今回のサンプルでは常に最小だった。

### 汎用 slot/value table は意外と軽くない

人間には扱いやすく、DB 向きではあるが、一般木構造を素直に flatten すると edge row の分だけ増える。そのため、汎用形では binary AST より重い。

## 何を採用すべきか

この比較から言えるのは次の通りである。

- 人間向け canonical 表現:
  Lisp 風 S-expression は有力
- 外部 API やツール互換:
  JSON AST は分かりやすい
- 言語モデル内部や runtime memory:
  binary AST か ID 列が本命
- 分析や検索インデックス:
  slot/value table は補助表現として有力

つまり、1つに統一するより、

1. authoring / canonical form = Lisp 風
2. interchange form = JSON AST
3. runtime storage = binary AST
4. index / analytics = slot/value table

の多層構成にする方が合理的である。

## この比較から分かる重要点

もし本当に「学習コストと推論コストを極めて安くしたい」なら、Lisp 風表現をそのまま内部表現にしてはいけない。

Lisp 風は canonical text としては良いが、内部では構造記号の比率が高すぎる。したがって、実運用では次の流れが自然である。

1. 入力または知識を canonical Lisp 風表現に落とす
2. それを binary AST にコンパイルする
3. slot/value table を補助索引として持つ
4. runtime memory には binary AST 断片か、そのさらに圧縮された ID 列だけを載せる

## 一文での結論

Lisp 風表現は人間向け・正規化向けには有効だが、低コストな知識保持の本命は binary AST であり、slot/value table は補助索引として使うのが最も自然である。
