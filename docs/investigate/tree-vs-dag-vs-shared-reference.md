# Tree vs DAG vs Shared Reference

更新日: 2026-03-28

## 結論

文の再帰構造そのものは `tree` で表せる。

ただし、情報量節約まで考えるなら、内部表現は
`shared reference を持つ DAG`
にするのが自然である。

つまり、

- 構文や節の入れ子は tree 的に表す
- 同じ部分構造の再利用は reference で共有する

というハイブリッドが最もよい。

## 3つの候補

### 1. Pure Tree

各出現を毎回その場で展開する方式。

例:

```lisp
(stmt
  (event
    (pred believe)
    (arg experiencer PERSON_SLOT_1)
    (arg content
      (stmt
        (event
          (pred come)
          (arg agent PERSON_SLOT_2))))))
```

利点:

- 直感的
- parser が簡単
- 局所性が高い

欠点:

- 同じ部分構造が複数回出ると重複する
- 比較やキャッシュで無駄が出る
- 文書全体で共有知識を持ちにくい

### 2. Pure Reference Graph

すべての節、句、関係を ID 化して、参照だけでつなぐ方式。

例:

```lisp
(ref STMT_7)
(ref REL_3)
(ref ENTITY_2)
```

利点:

- 重複は減る
- 共有や再利用がしやすい

欠点:

- 短い局所構造まで ID 化すると逆に重い
- 人間可読性が落ちる
- parser と validator が複雑になる
- 局所性が悪化する

### 3. Shared Reference DAG

基本は tree 的に書き、繰り返しや共有がある部分だけ ID 参照にする方式。

例:

```lisp
(def STMT_7
  (stmt
    (event
      (pred come)
      (arg agent PERSON_SLOT_2))))

(stmt
  (event
    (pred believe)
    (arg experiencer PERSON_SLOT_1)
    (arg content (ref STMT_7))))

(stmt
  (event
    (pred deny)
    (arg experiencer PERSON_SLOT_3)
    (arg content (ref STMT_7))))
```

利点:

- tree の分かりやすさを保てる
- 繰り返し部分だけ圧縮できる
- 同一性判定やキャッシュに向く
- 文書全体の整合性を取りやすい

欠点:

- reference 導入基準が必要
- intern table が必要
- inline と ref の両方を扱う必要がある

## なぜ DAG が自然か

文の再帰は、必ずしも「循環」を意味しない。

通常必要なのは、

- 節の入れ子
- 関係の再利用
- 同一 entity 参照
- 同一命題参照
- 同一 pattern や formula 参照

であり、これは循環グラフより
`共有つき有向非巡回グラフ`
で十分なことが多い。

したがって、

- 表現力のために tree 構造を持つ
- 圧縮のために subgraph sharing を入れる

という設計が合理的である。

## 何を shared reference にすべきか

全部を参照化しない方がよい。

参照化に向くもの:

- 繰り返し出る節
- 同じ数式
- 同じ pattern
- 同一 entity
- 同一 relation bundle
- 文書レベルで再利用される定義

inline のままでよいもの:

- 一回しか出ない短い句
- 非常に小さい局所構造
- 参照するとかえって ID の方が長いもの

## 参照化の判断基準

最小限のルールとしては、次が自然である。

1. 2回以上出る
2. ノード数が閾値以上
3. 文書全体で再参照される可能性が高い
4. 束縛や更新の単位として独立性が高い

つまり、
`出現頻度 × 構造サイズ × 再利用価値`
で intern する。

## 情報量の観点

情報量節約という観点では、shared reference DAG には次の利点がある。

- 重複部分木の保存を避けられる
- 同一意味単位を1回だけ保持できる
- token 列としても binary AST と相性がよい
- soft layer もノード単位で共有できる

特に soft layer がある場合、shared reference はさらに重要になる。

理由:

- 同じ節や relation に毎回 embedding を付け直す必要がなくなる
- 束縛更新を一か所で反映できる
- 関係の再評価結果を複数箇所で共有できる

## 再帰と参照共有の違い

ここは分けて考えるべきである。

- 再帰:
  節や句が他の節や句の内部に入ること
- 参照共有:
  同じ部分構造を複数箇所で使い回すこと

再帰は表現力の話であり、参照共有は圧縮と実装効率の話である。

したがって最適解は、

- 再帰は許す
- 共有は参照にする

である。

## 実装上の推奨

表面表現:

- S-expression 風に inline で持ってよい

内部表現:

- node id を持つ DAG
- repeated subgraph は intern
- relation と statement は参照可能にする

圧縮方針:

- default は inline
- threshold を超えたら shared reference 化

## 例

### Tree 的表現

```lisp
(stmt
  (event
    (pred say)
    (arg agent PERSON_SLOT_1)
    (arg content
      (stmt
        (event
          (pred come)
          (arg agent PERSON_SLOT_2))))))
```

### Shared Reference DAG 的表現

```lisp
(def STMT_12
  (stmt
    (event
      (pred come)
      (arg agent PERSON_SLOT_2))))

(stmt
  (event
    (pred say)
    (arg agent PERSON_SLOT_1)
    (arg content (ref STMT_12))))

(stmt
  (event
    (pred doubt)
    (arg experiencer PERSON_SLOT_3)
    (arg content (ref STMT_12))))
```

後者では、`come(PERSON_SLOT_2)` を一度だけ保存し、複数文が共有している。

## 一文での整理

文の再帰構造は tree で十分に表せるが、情報量節約と再利用まで考えるなら、実装上は `shared reference を持つ DAG` にするのが最も合理的である。
