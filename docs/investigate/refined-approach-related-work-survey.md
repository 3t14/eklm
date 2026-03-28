# 類似研究の再調査

更新日: 2026-03-28

## 対象にしている構想

今回の再調査では、類似研究の対象を次のように絞った。

- 表現部、知識部、導出部を分離する
- 中心に置くのは `Intermediate Representation` としての共通IRである
- 数式と正規表現を中核表現として重視する
- 自然言語、数式、Markdown、コードをモジュール合成で扱いたい
- 自然言語の曖昧部分だけを小型Transformerで構造割当に使いたい
- deterministic に parse できるものは parser を優先したい

したがって、今回の調査は単なる RAG や tool use 一般ではなく、

1. interlingua / meaning representation
2. semantic parsing / structure assignment
3. grammar-constrained structured generation
4. modular symbolic-neural design

に近い研究を中心にしている。

## 結論

2026年3月28日時点で、この構想に完全一致する単一アーキテクチャは見当たらない。

ただし、近い研究はかなりある。特に近いのは次の3系統である。

1. 共通意味表現を作る研究
   AMR, UCCA, UMR, BMR, UDS, ULF, GF
2. 自然言語を意味表現へ割り当てる研究
   semantic parsing, MRP, DRT/ULF parser, controlled sublanguage, compositional parser
3. LLM出力を文法やオートマトンで拘束する研究
   constrained decoding, automata-based decoding, grammar-constrained logical parsing

この構想に最も近い既存の「核」を一つずつ選ぶなら、

- interlingua とモジュール性では [GF](https://aclanthology.org/2020.cl-2.6/)
- 多言語意味表現では [UMR](https://aclanthology.org/2024.lrec-main.229/)
- fully semantic な言語横断表現では [BMR](https://aclanthology.org/2022.acl-long.121/)
- 構造割当器のハイブリッド性では [Transparent Semantic Parsing with UD Graph Transformations](https://aclanthology.org/2022.coling-1.367/)
- richer symbolic target への neural parsing では [Neural Semantic Parsing with Extremely Rich Symbolic Meaning Representations](https://aclanthology.org/2025.cl-1.7/)
- constrained output では [Automata-based constraints for language model decoding](https://arxiv.org/abs/2407.08103)

が代表になる。

## 1. 共通意味表現・interlingua 系

### 1.1 GF: 抽象構文を interlingua として使う

- [Abstract Syntax as Interlingua: Scaling Up the Grammatical Framework from Controlled Languages to Robust Pipelines](https://aclanthology.org/2020.cl-2.6/)

この研究は、今回の構想にかなり近い。GF は自然言語の背後にある `abstract syntax` を interlingua として扱い、40以上の言語へ展開できる。

近い点:

- 言語横断の共通表現を置く
- 言語別実現はモジュールとして分ける
- 合成的に拡張できる
- parser / generator の両方向がある

違う点:

- GF は文法工学寄りで、現代LLMの曖昧性処理や外部知識分離までは中心に置かない
- 数式や正規表現を第一級表現として前面には出していない

評価:

今回の「言語別IR + 文書IR + 共通コアIRを合成する」発想に最も近い古典系の成功例である。

### 1.2 UMR: 多言語・文書レベルの意味表現

- [Building a Broad Infrastructure for Uniform Meaning Representations](https://aclanthology.org/2024.lrec-main.229/)
- [Uniform Meaning Representation Parsing as a Pipelined Approach](https://aclanthology.org/2024.textgraphs-1.3/)
- [Generating Text from Uniform Meaning Representation](https://aclanthology.org/2025.ijcnlp-long.16/)

UMR は AMR を拡張し、文書レベル関係まで含めた多言語意味表現を目指している。

近い点:

- language-neutral を志向している
- sentence graph と document graph を分けている
- parsing を pipeline で考えている

違う点:

- グラフ表現が中心で、数式・正規表現・コードブロックのモジュール合成は前提にしていない
- 知識部と導出部の分離は設計中心ではない

評価:

「全LMで共有できる意味表現」の最近の代表候補。今回の構想に最も近い現代的 interlingua の一つ。

### 1.3 BMR: fully semantic な interlingua

- [Fully-Semantic Parsing and Generation: the BabelNet Meaning Representation](https://aclanthology.org/2022.acl-long.121/)

BMR は BabelNet と VerbAtlas を使い、言語固有制約から抽象化した `fully-semantic` な interlingual formalism を提案している。

近い点:

- language-independent meaning representation を強く志向している
- parsing と generation の両方向がある
- lexical resource を意味表現の中心に置く

違う点:

- BabelNet 依存が強く、今回のような軽量共通IRというより「意味資源に深く結びついた interlingua」
- regex / 数式 / 文書構造モジュールは中心にない

評価:

今回の構想が「知識と表現の分離」を semantic KB と強く結びつける方向に進むなら、かなり参考になる。

### 1.4 UCCA, UDS, ULF, DRT/MRS 系

- [SemEval-2019 Task 1: Cross-lingual Semantic Parsing with UCCA](https://aclanthology.org/S19-2001/)
- [The Universal Decompositional Semantics Dataset and Decomp Toolkit](https://aclanthology.org/2020.lrec-1.699/)
- [A Type-coherent, Expressive Representation as an Initial Step to Language Understanding](https://aclanthology.org/W19-0402/)
- [Translation using Minimal Recursion Semantics](https://aclanthology.org/1995.tmi-1.2/)

これらは「どの情報を意味表現側で解決し、どの情報を未解決のまま保持するか」の設計例として重要である。

特に ULF は、

- strictly typed
- close to surface form
- ただし quantifier scope, word sense, anaphora などは未解決に残す

という立場を取る。これは今回の `choice / unknown / unresolved` にかなり近い。

評価:

今回の構想における「非Turing完全な共通IR」や「段階的解像」の設計に直接効く。

## 2. 構造割当・semantic parsing 系

### 2.1 Cross-framework Meaning Representation Parsing

- [Proceedings of the Shared Task on Cross-Framework Meaning Representation Parsing at CoNLL 2019](https://aclanthology.org/volumes/K19-2/)
- [Proceedings of the CoNLL 2020 Shared Task: Cross-Framework Meaning Representation Parsing](https://aclanthology.org/volumes/2020.conll-shared/)
- [Broad-Coverage Semantic Parsing as Transduction](https://aclanthology.org/D19-1392/)

この系統は、AMR, UCCA, MRS 系など異なる意味表現を、ある程度統一的に parse しようとした研究群である。

近い点:

- 一つの parser で複数 formalism を扱おうとする
- graph-based parser や transition-based parser を用いる

違う点:

- 多くは「既存意味表現への parser」を作る方向で、今回のような新しい modular IR 設計そのものは対象ではない

評価:

共通IRの実装可能性や parser 設計を考える上で、最も実務的に近い先行研究。

### 2.2 Controlled sublanguage / canonical utterance

- [Constrained Language Models Yield Few-Shot Semantic Parsers](https://aclanthology.org/2021.emnlp-main.608/)
- [Few-Shot Semantic Parsing with Language Models Trained on Code](https://aclanthology.org/2022.naacl-main.396/)

この系統は、LLMが自然文を直接 logical form に出すのではなく、まず controlled sublanguage や canonical utterance に paraphrase し、それを target MR へ変換する。

近い点:

- LLM に最終 formalism を丸投げしない
- 中間表現をはさむ
- 少量データで bootstrapping しやすい

違う点:

- controlled sublanguage は English-like な canonical text であって、今回のような多表現型IRではない

評価:

「小型 Transformer を構造割当器として使う」設計の、かなり近い神経部品の先例。

### 2.3 Rule-based / hybrid semantic parsing

- [Transparent Semantic Parsing with Universal Dependencies Using Graph Transformations](https://aclanthology.org/2022.coling-1.367/)
- [Scope-enhanced Compositional Semantic Parsing for DRT](https://aclanthology.org/2024.emnlp-main.1093/)

前者は UD parser を使い、その上に graph transformations を適用して DRT ベースの formal meaning representation へ写す。後者は DRT parsing に対して compositional, neurosymbolic parser を導入している。

近い点:

- deterministic parser と symbolic transformation を重視する
- neural end-to-end より transparency, well-formedness を重視する
- 複雑な formalism では pure seq2seq が崩れやすいという問題意識を共有する

評価:

今回の「deterministic に parse できるものは parser を優先し、曖昧な自然言語だけ Transformer で割り当てる」という方針に最も近い。

### 2.4 Rich symbolic targets と parser

- [Neural Semantic Parsing with Extremely Rich Symbolic Meaning Representations](https://aclanthology.org/2025.cl-1.7/)
- [Boosting a Semantic Parser Using Treebank Trees Automatically Annotated with Unscoped Logical Forms](https://aclanthology.org/2025.dmr-1.3/)

これらは、symbolic target を richer にした場合、parser 側に何が起こるかを示している。

示唆:

- richer symbolic target は OOV や compositionality に有利
- ただし、構造が重くなると通常指標では不利になることもある
- 自動データ拡張や treebank-derived supervision が重要になる

評価:

今回の構想で「まず IR を作れば自然に学習できる」とは限らず、rich IR には parser 学習の工夫が必要だと示す。

## 3. 文法制約・structured generation 系

### 3.1 オートマトンと constrained decoding

- [Automata-based constraints for language model decoding](https://arxiv.org/abs/2407.08103)
- [Grammar-Constrained Decoding Makes Large Language Models Better Logical Parsers](https://aclanthology.org/2025.acl-industry.34/)
- [Earley-Driven Dynamic Pruning for Efficient Structured Decoding](https://arxiv.org/abs/2506.01151)
- [Draft-Conditioned Constrained Decoding for Structured Generation in LLMs](https://arxiv.org/abs/2603.03305)

これらは「LLM に structured output を出させるとき、grammar や automata でどこまで拘束できるか」を扱う。

近い点:

- regex / CFG / logical grammar を第一級に扱う
- smaller model でも構造の妥当性を改善できる
- constrained decoding を training-free または lightweight に使える

違う点:

- 多くは出力形式の保証が主で、入力理解の共通IR設計までは踏み込まない

評価:

今回の構想で Transformer が IR を出力する場合、生成フェーズの信頼性を上げるための実装候補になる。

## 4. この構想との対応表

| 構想の要素 | 近い研究 | コメント |
| --- | --- | --- |
| 多言語 interlingua | GF, UMR, BMR | GF はモジュール合成、UMR/BMR は意味グラフ側が強い |
| rich symbolic meaning | UDS, ULF, DRT, MRS, taxonomical parser | underspecification と typed structure の参考になる |
| 構造割当器 | semantic parsing, MRP, controlled sublanguage, UD graph transformations | 小型Transformerを parser/scorer として使う設計に近い |
| regex / formal constraints | automata-based decoding, grammar-constrained logical parsing | 出力IRの well-formedness 保証に近い |
| modular document + language + code | GF の abstract syntax は近いが、Markdown + code + formula の混在まで正面から扱う代表研究は少ない | ここは新規性が出やすい |
| 知識と表現の分離 | 以前の survey にある RAG / KBLaM / symbolic solver 系 | 今回の再調査では中心ではないが重要な補助線 |

## 5. 何が既存で、何がまだ無いか

### 既存でかなり進んでいるもの

- 言語横断の意味表現
- graph-based / logical-form-based semantic parsing
- underspecified meaning representation
- constrained decoding
- hybrid symbolic-neural parsing

### まだ明確な代表が無いもの

- 数式と正規表現を中核に据えた共通IR
- Markdown, 自然言語, 数式, コードを同一文書内で modular に扱う IR
- 表現部、知識部、導出部を明確に分けた language model 全体設計
- deterministic parser + regex/FSA + 小型Transformer + knowledge routing を一体化したアーキテクチャ

## 6. 再調査後の判断

この構想は、研究系譜としてはかなり多くの先行研究に接続している。ただし、それらは別々のコミュニティに散っている。

- computational semantics
- semantic parsing
- grammar engineering
- constrained decoding
- neuro-symbolic NLP

そのため、今回の新規性は「単一要素の発明」より、

- modular interlingua
- regex / math as first-class sublanguages
- deterministic + neural hybrid assignment
- externalized knowledge / derivation

を一本の設計にまとめる点にある。

## 7. いま一番近い比較対象

現時点で比較対象として特に重要なのは次の6本である。

1. [Abstract Syntax as Interlingua: Scaling Up the Grammatical Framework from Controlled Languages to Robust Pipelines](https://aclanthology.org/2020.cl-2.6/)
2. [Building a Broad Infrastructure for Uniform Meaning Representations](https://aclanthology.org/2024.lrec-main.229/)
3. [Fully-Semantic Parsing and Generation: the BabelNet Meaning Representation](https://aclanthology.org/2022.acl-long.121/)
4. [Transparent Semantic Parsing with Universal Dependencies Using Graph Transformations](https://aclanthology.org/2022.coling-1.367/)
5. [Neural Semantic Parsing with Extremely Rich Symbolic Meaning Representations](https://aclanthology.org/2025.cl-1.7/)
6. [Automata-based constraints for language model decoding](https://arxiv.org/abs/2407.08103)

この6本で、

- interlingua
- multilinguality
- hybrid parsing
- rich symbolic target
- formal constraints

がほぼ一通り見える。

## 8. 短い結論

2026年3月28日時点で、今回の構想に最も近い研究の中心は `interlingual meaning representation + semantic parsing + constrained decoding` の交点にある。

ただし、`数式・正規表現を第一級に持つ modular IR`、`Markdown/自然言語/数式/コード混在の文書IR`、`表現部・知識部・導出部の分離` を一体として押し出した代表研究は、まだ見当たらない。

したがって、研究的には「全く無からの発明」ではないが、組み合わせとしては十分に新規性がある。
