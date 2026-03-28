# 類似研究との差分表と勝ち筋

更新日: 2026-03-28

## 目的

この文書は、ここまで整理してきた方式を既存研究と比較し、

- どこが既存と重なるか
- どこが差分になるか
- どこに勝ち筋があるか

を一枚で把握するためのメモである。

ここでの提案方式は、次を前提にしている。

- 表現部、知識部、導出部を分離する
- 共通IRを中心に置く
- 数式と正規表現を中核表現として重視する
- 自然言語、Markdown、コード、数式をモジュール合成で扱う
- deterministic parser と小型Transformerを併用する

## 比較表

| 比較対象 | 重なる点 | 主な差分 | この方式への示唆 |
| --- | --- | --- | --- |
| [GF](https://aclanthology.org/2020.cl-2.6/) | 抽象構文を interlingua として置く。言語別モジュールを合成する。 | 文法工学が中心で、知識部と導出部の分離は主題ではない。数式、正規表現、コード混在も中心ではない。 | 共通IRのモジュール設計では最も近い。 |
| [UMR](https://aclanthology.org/2024.lrec-main.229/) | language-neutral な意味表現を目指す。文書レベルまで扱う。 | グラフ意味表現が中心で、数式、regex、文書構造IR、知識分離は中心ではない。 | 共通意味表現の粒度設計と文書レベル拡張の参考になる。 |
| [BMR](https://aclanthology.org/2022.acl-long.121/) | 言語横断の fully semantic representation を志向する。 | 意味資源への依存が強く、軽量IRというより重い semantic formalism に近い。 | 知識部との強い接続を考えるときの参考になる。 |
| [ULF](https://aclanthology.org/W19-0402/) / UDS / DRT / MRS | 型、意味役割、未解決情報の保持を重視する。 | 多表現型のモジュールIRではなく、数式や regex を第一級には置かない。 | `unknown`, `choice`, `unresolved` の設計に効く。 |
| [MRP / Broad-Coverage Semantic Parsing as Transduction](https://aclanthology.org/D19-1392/) | 一つの parser で rich symbolic target を扱う。 | 既存 formalism への parser が中心で、新しい language model 全体設計ではない。 | parser 学習と評価設計の参考になる。 |
| [Constrained Language Models Yield Few-Shot Semantic Parsers](https://aclanthology.org/2021.emnlp-main.608/) | 中間表現を挟み、少量データで semantic parsing を成立させる。 | canonical utterance は自然言語寄りで、多表現型IRではない。 | 小型Transformerを構造割当器として使う根拠になる。 |
| [Transparent Semantic Parsing with UD Graph Transformations](https://aclanthology.org/2022.coling-1.367/) | deterministic parser と symbolic transformation を重視する。 | 文書IR、数式IR、regex IR、知識分離は前面にない。 | hybrid parsing 方針に最も近い。 |
| [Neural Semantic Parsing with Extremely Rich Symbolic Meaning Representations](https://aclanthology.org/2025.cl-1.7/) | rich symbolic target の有効性を示す。 | symbolic target が重くなると parser 側も難しくなる。 | IR を rich にしすぎると学習が難しくなるという警告になる。 |
| [Automata-based constraints for language model decoding](https://arxiv.org/abs/2407.08103) | grammar や automata で出力構造を拘束する。 | 入力理解の共通IR設計や知識分離は扱わない。 | IR生成時の well-formedness 保証に有効。 |
| [KBLaM](https://arxiv.org/abs/2410.10450) / RAG 系 | 知識を重みの外へ出す。更新を軽くする。 | 共通IRや言語構造の明示表現は中心ではない。 | 知識部の外部化と runtime 参照の実装で参考になる。 |

## 何が新規性になり得るか

単一要素として完全に新しいものは少ない。新規性は、主に次の組み合わせにある。

1. 共通IRを、単なる意味グラフではなく多表現型のモジュール集合として設計すること
2. 数式と正規表現を第一級の表現族として前面に置くこと
3. Markdown、自然言語、数式、コードを同一文書内で独立IRとして合成すること
4. deterministic parser と小型Transformerを明示的に分業させること
5. 表現部、知識部、導出部を language model 全体設計として分離すること

要するに、勝負は「新しい parser」でも「新しい KB」でもなく、
`低コストな modular language system として全体を再編成すること`
にある。

## 勝ち筋

### 1. 形式性が高い領域で勝つ

この方式が最も強いのは、意味のかなりの部分を

- 数式
- 制約
- パターン
- 型
- ルール

に落とせる領域である。

具体例:

- 数学
- 物理
- プログラム解析
- 仕様書
- 法務文書の定型部分
- 医療プロトコル
- ログ解析

理由:

- 自明な推論を安い演算に落とせる
- 曖昧な部分だけ小型Transformerに回せる
- 幻覚より `未解決` として止まりやすい

### 2. 更新頻度が高い知識領域で勝つ

Transformer 型LMは、知識更新に再学習や微調整が要る。ここが重い。

一方この方式では、

- 知識部の差し替え
- 参照ルーティングの更新
- ルールや式の追加

で済ませやすい。

したがって、

- 頻繁に変わる仕様
- 企業内ナレッジ
- 法令や規格
- バージョン依存の技術文書

のような領域ではコスト優位が出やすい。

### 3. 小型モデルで成立させやすい

この方式では、大きいモデルを万能化する代わりに、小さいモデルを

- 構造割当
- 参照選択
- 再表現

へ限定できる。

したがって勝ち筋は、

- 低レイテンシ
- 低学習コスト
- オンデバイス寄り
- ドメイン特化の安価な運用

にある。

これは、巨大モデルの汎用性能で正面勝負するのではなく、
`十分に強い小型系を安く成立させる`
方向の勝ち筋である。

### 4. 混在文書処理で勝つ

Markdown 文書の中に、

- 日本語説明
- 英語引用
- 数式
- コードブロック
- パターン定義

が混在するケースでは、トークン列一本化だけでは無駄が多い。

この方式では、それぞれを別IRで処理し、必要なところだけ接続できる。

これは特に、

- 技術文書
- 研究ノート
- 教材
- 実験記録
- 開発ドキュメント

で効く。

### 5. 監査性と失敗の質で勝つ

この方式は、全問正解の万能系ではなく、
`間違え方を改善する`
ことにも価値がある。

具体的には、

- どの規則を使ったか追える
- どの知識片を引いたか追える
- どこが未解決かを明示できる
- どの型制約で失敗したか分かる

ので、監査やデバッグがしやすい。

これは、

- 研究用途
- 企業内運用
- 高信頼性が必要な支援システム

で強い。

## 勝ちにくい領域

逆に、この方式が不利になりやすいのは次である。

- 雑談や感性的な会話が中心の用途
- 世界知識が広く散らばり、形式化しにくい用途
- 高度な常識補完や語用論が支配的な用途
- 出力の厳密性より流暢性や創造性が重要な用途

こうした領域では、大規模 Transformer の end-to-end 柔軟性がまだ強い。

したがって、この方式の勝ち筋は
`汎用会話の完全代替`
ではなく、
`構造性が高く、更新が多く、監査が重要な領域で、低コストに強い系を作ること`
にある。

## 研究提案としての主張

研究提案として一番強い主張は、次のようになる。

本提案は、既存の interlingua、semantic parsing、constrained decoding、external knowledge の成果を統合し、数式と正規表現を第一級に含む modular IR を中心に、表現部、知識部、導出部を分離した低コスト言語モデルを提案する。

この主張の焦点は、

- より大きいモデルを作ること
- より大量の pretraining を行うこと

ではなく、

- 何を学習し
- 何を外部化し
- 何を決定的に処理し
- 何だけを確率的に残すか

を再設計する点にある。

## 次に詰めるべきこと

次の論点を詰めると、研究提案としてかなり強くなる。

1. 最小プロトタイプでどのタスクに勝つか
2. 何を主要評価指標にするか
3. 共通IRをどこまで rich にするか
4. 小型Transformerをどの位置で使うか
5. 知識部の最小構成を何にするか

特に重要なのは、「勝ち筋があるタスクを最初から狭く切ること」である。
