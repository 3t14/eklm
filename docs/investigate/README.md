# Investigate Index

更新日: 2026-03-28

このディレクトリには、知識と表現を分離した低コスト言語モデルについての調査、構想整理、仕様草案をまとめている。

## 推奨読書順

1. [設計原則](./design-principles.md)
2. [提案アーキテクチャの差分図](./proposed-architecture-diff.md)
3. [4層数理言語モデル案](./four-layer-mathematical-language-model.md)
4. [共通IR最小タグ仕様 v0.1](./common-ir-minimal-tag-spec.md)
5. [構造割当器アーキテクチャ](./structure-assigner-architecture.md)

## 調査

- [知識と表現を分離する言語モデルの先行研究調査](./knowledge-expression-separation-survey.md)
  既存研究の系譜を、知識外部化、数式・論理・プログラム、正規表現・オートマトンの3系統を中心に整理した調査。

- [類似研究の再調査](./refined-approach-related-work-survey.md)
  現在の構想に寄せて、共通IR、semantic parsing、文法制約付き生成、モジュール合成を中心に再整理した調査。

- [類似研究との差分表と勝ち筋](./related-work-diff-and-winning-path.md)
  既存研究との差分と、この方式が勝ちやすい領域、勝ちにくい領域を一枚で整理したメモ。

## 全体構想

- [設計原則](./design-principles.md)
  現時点での中心思想を短く整理した文書。表現部、知識部、導出部の分離を定義している。

- [Core Novelty Claim](./core-novelty-claim.md)
  この構想の中核的新規性を、既存研究との差分とともに一枚で圧縮した文書。

- [Dual Representation Architecture](./dual-representation-architecture.md)
  typed slot graph と潜在空間を併用する二層表現として、この方式を整理した文書。

- [Lisp / Prolog / 提案方式の比較](./lisp-prolog-comparison.md)
  Lisp や Prolog に見える点と、本方式がそれらと本質的に異なる点を整理した比較表。

- [Meta-Attention Architecture](./meta-attention-architecture.md)
  sentence、paragraph、section、document、corpus の上位ノードに対する階層 attention / routing の設計案。

- [提案アーキテクチャの差分図](./proposed-architecture-diff.md)
  現在主流の Transformer ベース LLM と、この構想との差分を一枚図で比較したもの。

- [4層数理言語モデル案](./four-layer-mathematical-language-model.md)
  自然言語を、表層、構文、意味、メタ解釈の4層で数理化する構想を整理したもの。

## 共通表現とIR

- [TCMath/IR 設計案](./tcmath-interlingua-design.md)
  `TCMath` を実行言語ではなく、全LMが共有できる意味表現レイヤーとして再定義した文書。

- [共通IR最小タグ仕様 v0.1](./common-ir-minimal-tag-spec.md)
  共通IRの最小タグ集合、role、演算、validation、例をまとめた仕様。

- [Markdown混在文書IR仕様](./markdown-mixed-ir-spec.md)
  Markdown 文書の中に日本語、英語、数式、コードブロックを埋め込むための文書構造IR仕様。

## 抽象化方針

- [語彙抽象化ポリシー](./vocabulary-abstraction-policy.md)
  自然言語、数式、正規表現に共通して、構造を固定語彙、内容依存要素を型付きスロットへ抽象化する方針。

- [Lexical Grounding Architecture](./lexical-grounding-architecture.md)
  スロット化だけでは失われる固有名詞や専門語の意味を、surface と binding で後から接地する設計。

## 割当・解析

- [構造割当器アーキテクチャ](./structure-assigner-architecture.md)
  入力を共通IRへ割り当てるための、deterministic parser + regex/FSA + 小型Transformer + validator のハイブリッド設計。

- [Structure Assignment Pipeline](./structure-assignment-pipeline.md)
  表層関係抽出、仮スロット化、抽象関係化、知識束縛、関係再更新までの時系列パイプライン。

- [Soft Layer Learning and Post-Hoc Adaptation](./soft-layer-learning-and-post-hoc-adaptation.md)
  Soft Layer を直接注釈せず、残差として学習し、事後学習で改善する方針。

## 表現形式の比較

- [表現形式ごとのサイズ比較](./representation-size-comparison.md)
  同じ構造を Lisp 風、JSON AST、binary AST、slot/value table で保持したときの概算サイズ比較。

- [Tree vs DAG vs Shared Reference](./tree-vs-dag-vs-shared-reference.md)
  再帰構造の表現力と、shared reference による情報量節約を比較したメモ。

## 運用・権利

- [Copyright-Aware and Compensable KB Architecture](./copyright-aware-compensable-kb-architecture.md)
  source text と canonical knowledge を分離し、provenance と補償設計を可能にする KB 方針。

## 初期草案

- [TCMath/1 設計案](./tcmath-design.md)
  `TCMath` を execution substrate 寄りに解釈した初期草案。現在の中心設計は `tcmath-interlingua-design.md` 側。

## いまの中心線

現時点での中心線は次の通り。

- 共通IRは `Intermediate Representation` であり、`Information Retrieval` ではない
- 表現部は非Turing完全で、正規化可能、検証可能な構造を優先する
- 数式と正規表現は中核だが、全体は多表現型の共通IRとして設計する
- 自然言語、数式、正規表現、Markdown、コードは、独立IRを合成する形で扱う
- 曖昧な自然言語の構造割当には小型Transformerを使うが、deterministic に parse できる部分は parser を優先する
