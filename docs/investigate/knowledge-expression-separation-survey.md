# 知識と表現を分離する言語モデルの先行研究調査

更新日: 2026-03-28

## 要旨

この調査では、あなたの構想を「知識をモデル重みから切り離し、数式・正規表現・論理・プログラムなどの明示表現に保持し、言語モデル本体は自然言語との変換・説明・推論オーケストレーションを担うアーキテクチャ」と解釈した。

結論を先に書くと、2026年3月時点でこの方向はすでに複数の研究系統として進んでいる。ただし、研究はまだ一つの統一アーキテクチャには収束していない。進展は主に次の3本立てで進んでいる。

1. 知識を重みの外に出す系: RAG、REALM、RETRO、kNN-LM、KBLaM など
2. 推論を数式・プログラム・論理式に落として外部ソルバに委ねる系: PAL、PoT、Logic-LM、SymbCoT、Autoformalization など
3. 正規表現・有限オートマトンで出力や局所構造を制御する系: REI、ReLM、Automata-based Decoding、Logically Constrained Decoding など

現状で最も成熟しているのは 1 であり、次が 2。3 は「書式・構文・局所制約」の制御ではかなり強いが、「世界知識そのもの」を正規表現だけで表す方向はまだ初期段階にある。したがって、あなたの発想は研究的に十分妥当だが、数式と正規表現だけを唯一の知識表現にするより、型付きの記号表現群の一部として使う方が現実的である。

## この構想を既存研究に対応づけると何になるか

既存研究の言葉に置き換えると、狙いは次の4点に分解できる。

- 知識の外部化: 事実や規則を重みではなく外部メモリや記号表現に置く
- 表現の明示化: 数式、正規表現、論理式、プログラム、オートマトンなどで知識を表す
- 推論の分業化: 自然言語の生成はLM、厳密計算や検証はソルバ/検証器に任せる
- 編集可能性の向上: 知識更新を再学習ではなく、外部表現の差し替えで行う

この意味で、あなたの構想は「RAGの先」「神経記号AI」「形式推論LM」「制約付きデコーディング」を横断している。

## 研究の進み具合

| 系統 | 成熟度 | 何ができるか | まだ弱い点 |
| --- | --- | --- | --- |
| 知識を重みから外に出す | 高い | 事実更新、出典付与、長尾知識の補完 | 内部知識との衝突、整合性、マルチホップ更新 |
| 数式・論理・プログラムへ写像する | 中程度 | 計算、論理推論、形式数学、検証可能な中間表現 | 自然言語から記号表現への変換精度、一般知識全体への拡張 |
| 正規表現/オートマトンを使う | 中程度 | 出力制約、書式保証、構文保証、局所パターン表現 | 意味論の表現力、世界知識の網羅性 |
| これらを統合した単一LM | 低い | 部分プロトタイプはある | 支配的設計がまだ無い |

## 1. 知識を重みから外に出す研究

### 1.1 基本線

- [Petroni et al., 2019, "Language Models as Knowledge Bases?"](https://aclanthology.org/D19-1250/) は、BERTがかなりの関係知識を内部に保持していることを示した。これは逆に言えば、「言語能力」と「知識記憶」が現在のLMではまだ強く混ざっていることの出発点でもある。
- [REALM, 2020](https://arxiv.org/abs/2002.08909) は、知識が重みに暗黙保存される問題を明示し、「より modular and interpretable な形」で知識を扱うために検索器を学習させた。
- [RAG, 2020](https://arxiv.org/abs/2005.11401) は、パラメトリック記憶と非パラメトリック記憶を組み合わせる一般形を提示し、知識集約タスクで強い結果を出した。
- [kNN-LM, 2020](https://arxiv.org/abs/1911.00172) は、訓練済みLMに近傍検索を後付けし、追加学習なしで希少パターンや事実知識に強くできることを示した。
- [RETRO, 2021/2022](https://arxiv.org/abs/2112.04426) は 2 兆トークン規模の外部DBを参照しつつ、GPT-3級性能を 25 倍少ないパラメータで達成可能だと示した。

### 1.2 最近の進展

- [KBLaM, 2024/2025](https://arxiv.org/abs/2410.10450) は、10K超の知識三つ組を外部知識ベースとしてLMに結合し、再学習なしで動的更新可能な方向を示した。RAGよりも知識注入をモデル内部の専用注意機構に近づけている点が面白い。
- [MQuAKE, 2023](https://arxiv.org/abs/2305.14795) は、「一つの事実を更新したなら、その波及先の推論も更新されるべき」という問題を作った。結果は厳しく、既存の重み編集法は単発事実の更新はできても、多段推論では崩れやすかった。対照的に、外部メモリに編集事実を置く MeLLo が大きく改善した。
- [MQuAKE-Remastered, ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/hash/f782860c2a5d8f675b0066522b8c2cf2-Abstract-Conference.html) は、元ベンチマーク自体に最大 33% または 76% の破損があったと報告しており、この分野では評価基盤もまだ流動的だと分かる。
- [Evaluating the External and Parametric Knowledge Fusion of LLMs, 2024](https://arxiv.org/abs/2405.19010) は、外部知識と内部知識の融合がまだ十分理解されていないこと、特に「内部知識の境界判定」と「不完全な外部知識の補完」が難所だと示した。
- [ConflictBank, 2024, preprint](https://arxiv.org/abs/2408.12076) は、外部知識と内部知識の衝突が幻覚の主要因の一つだと整理した。
- [ParamMute, 2025, preprint](https://arxiv.org/abs/2502.15543) は、RAG中に内部パラメトリック知識が勝ちすぎる問題を抑えるため、特定FFNの抑制で「内部知識を弱める」方向を提案した。

### 1.3 判断

この系統はかなり進んでいる。少なくとも「知識を完全に重みに埋め込む必要はない」という点は、研究的にはほぼ確立している。ただし「重み側の知識をどう抑制し、外部知識と整合させるか」は未解決である。

## 2. 数式・論理・プログラムに推論を逃がす研究

### 2.1 計算と推論の分離

- [PAL, 2022](https://arxiv.org/abs/2211.10435) は、自然言語問題をLMがプログラムへ変換し、実計算はPythonに任せる形を提案した。これは「言語モデルは説明器、計算器は外部」という明快な分業である。
- [Program of Thoughts, 2022/2023](https://arxiv.org/abs/2211.12588) は、CoTが「推論」と「計算」を同じ自然言語列で処理してしまうことを問題視し、計算を外部コンピュータへ切り分けた。平均で約12%の改善を報告している。
- [Binder, 2022/2023](https://arxiv.org/abs/2210.02875) は、自然言語タスクをプログラムに落とし、LM機能自体をAPIとして埋め込んだ。LMを「万能知識倉庫」ではなく「記号系を補助する意味解析器」として使う発想に近い。

### 2.2 論理式・形式表現への変換

- [Logic-LM, 2023](https://arxiv.org/abs/2305.12295) は、自然言語問題を記号論理へ写像し、決定的ソルバで推論する。標準プロンプト比で平均 39.2% 改善としており、記号化と検証の有効性を示した。
- [SymbCoT, ACL 2024](https://arxiv.org/abs/2405.18357) は、自然言語を記号表現へ変換し、論理規則に沿った中間推論列と検証器を導入した。論理推論の faithful さを重視する研究で、あなたの構想にかなり近い。
- [LogicSkills, 2026, preprint](https://arxiv.org/abs/2602.06533) は、論理推論を「妥当性判定」「形式化」「反例構成」に分解して評価した。その結果、一般的な指示追従LMは妥当性判定はできても、形式化や反例構成は弱いことが示された。つまり、自然言語のうまさと記号化能力は別スキルである。

### 2.3 数学の形式化と検証

- [Autoformalization with Large Language Models, 2022](https://arxiv.org/abs/2205.12615) は、自然言語の数学問題を Isabelle/HOL に形式化し、MiniF2F の証明率改善につなげた。
- [FormalMATH, 2025](https://arxiv.org/abs/2505.02735) は Lean4 ベースの 5,560 問ベンチマークを作り、最強モデルでも実用的サンプリング予算下では成功率 16.46% に留まると報告した。形式数学は依然としてかなり難しい。
- [A Neuro-Symbolic Approach for Reliable Proof Generation with LLMs, 2025/2026, preprint](https://arxiv.org/abs/2505.14479) は、類題検索と形式検証器フィードバックを組み合わせることで、幾何証明精度を 58% から 70% 改善した。

### 2.4 判断

数式や論理式を中間表現として使う方向は有望で、特に計算・証明・厳密推論では効果がはっきり出ている。ただし、一般常識や広い世界知識全体を「数式だけ」で保持するところまでは進んでいない。現実には、数式は一部ドメインで非常に強いが、汎用知識表現の全部にはならない。

## 3. 正規表現・有限オートマトンを使う研究

### 3.1 正規表現を制約言語として使う

- [Toward Unified Controllable Text Generation via Regular Expression Instruction, 2023](https://arxiv.org/abs/2309.10447) は、語彙制約、位置制約、長さ制約などを正規表現風の指示で統一的に表す REI を提案した。これは「表現層を自然言語から切り離す」方向の強い実例である。
- [Validating Large Language Models with ReLM, 2022/2023](https://arxiv.org/abs/2211.15458) は、正規表現を使ってLMを問い合わせ・検証する仕組みを提案した。知識表現そのものではないが、LMを形式言語で点検する発想として重要である。
- [Correct and Optimal: the Regular Expression Inference Challenge, 2023/2024](https://arxiv.org/abs/2308.07899) は、例集合から最小正規表現を合成する課題を提案し、これが依然として未解決に近いことを示した。つまり、「正規表現そのものを学ぶ」こともまだ難しい。

### 3.2 オートマトンをメモリや制約器として使う

- [Neuro-Symbolic Language Modeling with Automaton-augmented Retrieval, 2022](https://arxiv.org/abs/2201.12431) は、kNN-LM の外部データストア検索を有限オートマトン化し、検索回数を最大 83% 削減した。これは「外部知識をオートマトン構造へ圧縮する」という点で、あなたの発想にかなり近い。
- [Automata-based constraints for language model decoding, 2024](https://arxiv.org/abs/2407.08103) は、正規言語と決定性文脈自由言語に対して、LM出力を必ず妥当な形式に制約する方法を与えた。制約コンパイルを約 7,000 倍高速化し、正しさも主張している。
- [Logically Constrained Decoding, 2025](https://aclanthology.org/2025.mathnlp-main.11/) は、正規表現やCFGのような構文制約を越えて、「違法手」や「不正な証明ステップ」を出せないようにする論理制約デコーディングを示した。

### 3.3 判断

正規表現とオートマトンは、形式保証、局所パターン、構文、プロトコル、API呼び出し、抽出ルールには非常に強い。一方で、人間世界の因果・常識・時間変化・多段関係をそのまま表すには表現力が足りない。したがって、正規表現は「唯一の知識表現」より、「表現層の一部」または「制約層」として使う方が筋が良い。

## あなたの構想に対する総合評価

### かなり進んでいる部分

- 知識を外部メモリへ逃がす発想そのもの
- 形式表現を中間層に入れて推論を安定化する発想
- 外部検証器やソルバで厳密性を担保する発想
- 正規表現やオートマトンで出力制約を保証する発想

### まだ十分進んでいない部分

- 数式と正規表現だけで汎用知識を統一的に表すこと
- 自然言語から正しい形式表現へ高精度に写像すること
- 外部知識と内部知識の衝突解消
- 多段推論のたびに知識更新が一貫して波及すること
- これらを統合した、訓練・推論・更新・検証まで一貫した標準アーキテクチャ

### 一番重要な示唆

あなたの構想は「まだ無いものをゼロから夢想している」のではなく、既存の複数研究を統合した先にある。研究的には十分戦えるテーマである。ただし、勝ち筋は「数式+正規表現だけに閉じる」ことより、「型付き明示表現の層を持つ知識OSを作る」ことにあるように見える。

## 数式と正規表現をどう位置づけるべきか

現状の研究を踏まえると、次の整理が妥当である。

- 数式:
  物理法則、数量関係、最適化、因果の一部、形式数学に強い
- 正規表現/オートマトン:
  表記制約、構文、局所パターン、プロトコル、抽出規則に強い
- 論理式:
  命題・述語推論、制約充足、整合性検査に強い
- プログラム:
  計算過程、アルゴリズム、外部ツール呼び出しに強い
- 三つ組/グラフ:
  エンティティ関係、更新、参照、マルチホップに強い

つまり、もし本当に新しいモデルを作るなら、知識基盤は少なくとも

1. 数式AST
2. 正規表現/FSA
3. 論理式
4. 知識グラフ or 三つ組
5. 実行可能プログラム

の混成で設計する方が自然である。

## 研究開発としての次の一手

もしこのリポジトリで実験を始めるなら、いきなり汎用LMを目指すより、次の順で刻むのが良い。

1. 狭い領域を選ぶ
   例: 数学QA、ログ解析、構造化文書抽出、プロトコル生成
2. 知識表現を型付きで定義する
   例: `formula`, `regex`, `logic`, `triple`, `program`
3. LMの役割を限定する
   自然言語 → 記号表現、記号表現 → 自然言語、検索クエリ生成、失敗時の自己修正
4. 検証器を必ず入れる
   数式ソルバ、正規表現/FSA、SAT/SMT、Lean/Isabelle、Python
5. ベースラインを置く
   通常RAG、通常CoT、通常Tool-use と比較する
6. 評価を分離する
   形式化精度、推論精度、検証通過率、知識更新コスト、外部知識との整合率

## 現時点での短い結論

2026年3月時点で、知識と表現を分離するという考え自体はかなり進んでいる。ただし、それは単一の「完成した新言語モデル」としてではなく、RAG、神経記号推論、形式数学、制約付きデコーディングという別々の研究系統として進展している。あなたの構想の核心は十分に研究最前線と接続しているが、真に新規性が出るのは「数式と正規表現を核としつつ、他の記号表現も含む統一知識層をどう設計するか」にある。

## 参考文献

- Petroni et al., 2019, [Language Models as Knowledge Bases?](https://aclanthology.org/D19-1250/)
- Guu et al., 2020, [REALM: Retrieval-Augmented Language Model Pre-Training](https://arxiv.org/abs/2002.08909)
- Lewis et al., 2020, [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)
- Khandelwal et al., 2020, [Generalization through Memorization: Nearest Neighbor Language Models](https://arxiv.org/abs/1911.00172)
- Borgeaud et al., 2021/2022, [Improving language models by retrieving from trillions of tokens](https://arxiv.org/abs/2112.04426)
- Alon et al., 2022, [Neuro-Symbolic Language Modeling with Automaton-augmented Retrieval](https://arxiv.org/abs/2201.12431)
- Wu et al., 2022, [Autoformalization with Large Language Models](https://arxiv.org/abs/2205.12615)
- Cheng et al., 2022/2023, [Binding Language Models in Symbolic Languages](https://arxiv.org/abs/2210.02875)
- Gao et al., 2022, [PAL: Program-aided Language Models](https://arxiv.org/abs/2211.10435)
- Chen et al., 2022/2023, [Program of Thoughts Prompting](https://arxiv.org/abs/2211.12588)
- Kuchnik et al., 2022/2023, [Validating Large Language Models with ReLM](https://arxiv.org/abs/2211.15458)
- Pan et al., 2023, [Logic-LM](https://arxiv.org/abs/2305.12295)
- Zhong et al., 2023/2024, [MQuAKE](https://arxiv.org/abs/2305.14795)
- Valizadeh et al., 2023/2024, [Correct and Optimal: the Regular Expression Inference Challenge](https://arxiv.org/abs/2308.07899)
- Zheng et al., 2023, [Toward Unified Controllable Text Generation via Regular Expression Instruction](https://arxiv.org/abs/2309.10447)
- Zhang et al., 2024, [Evaluating the External and Parametric Knowledge Fusion of Large Language Models](https://arxiv.org/abs/2405.19010)
- Xu et al., 2024, [Faithful Logical Reasoning via Symbolic Chain-of-Thought](https://arxiv.org/abs/2405.18357)
- Koo et al., 2024, [Automata-based constraints for language model decoding](https://arxiv.org/abs/2407.08103)
- Su et al., 2024, [ConflictBank](https://arxiv.org/abs/2408.12076)
- Wang et al., 2024/2025, [KBLaM: Knowledge Base augmented Language Model](https://arxiv.org/abs/2410.10450)
- Yu et al., 2025, [FormalMATH](https://arxiv.org/abs/2505.02735)
- Huang et al., 2025, [ParamMute](https://arxiv.org/abs/2502.15543)
- ACL MathNLP 2025, [Logically Constrained Decoding](https://aclanthology.org/2025.mathnlp-main.11/)
- Sultan et al., 2025/2026, [A Neuro-Symbolic Approach for Reliable Proof Generation with LLMs](https://arxiv.org/abs/2505.14479)
- Rabern et al., 2026, [LogicSkills](https://arxiv.org/abs/2602.06533)
