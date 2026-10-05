# WoDD — Workstream-driven Development

- **Status:** Draft v0.1
- **Type:** Framework Specification
- **Origin:** LoDD / test_AEIOU での実践から再構成

---

# 1. このフレームワークについて

## 1.1 WoDDとは

WoDD（Workstream-driven Development）は、AIを利用した開発作業を、

**Workstream → Phase → Task**

の単位で管理し、各段階で必要な事実・判断・完了条件を外部化するための軽量な作業フレームワークである。

WoDDは、AIの思考方法そのものを固定しない。

代わりに、

- 何を達成するのか
- 現在何が分かっているのか
- 何を変更するのか
- 何を変更しないのか
- 何をもって完了とするのか
- どの判断を後で再評価するのか

を明示する。

実装方法そのものは、これらの境界内でAIまたは人間が判断する。

---

# 2. WoDDが解決する問題

WoDDは主に以下の問題を対象とする。

- 現状を十分確認しないまま実装を開始する
- AIの推測を既知の事実として扱う
- 暗黙の既存挙動を壊したままリファクタリングする
- 作業中にスコープが拡大する
- 必要以上の抽象化・汎用化・再設計を行う
- Taskの完了をPhaseやWorkstreamの完了と誤認する
- 過去の設計判断が固定化され、前提が変わっても再評価されない
- 未確認事項が完了事項に紛れ込む
- AIが生成した成果を、人間が全体として再評価する地点がなくなる

---

# 3. WoDDが解決しない問題

WoDDは以下を保証しない。

- 正しいアーキテクチャを自動的に生成すること
- AIの誤りやハルシネーションを完全に防止すること
- AIのファイルアクセスを物理的に封鎖すること
- 全作業を自動化すること
- Planner / Worker / Judge 等の特定のAgent構成
- 特定の開発手法、テスト手法、言語、IDEへの依存

必要ならプロジェクト側で追加する。

---

# 4. 設計思想

> **AIを固定するのではなく、仕事の境界と判断点を固定する。**

WoDDでは、AIが実装方法を考える余地を残す。

一方で、AIが苦手としやすい以下の判断を作業構造として外部化する。

- Enough — どこまでやれば十分か
- Scope — どこまでが今回の仕事か
- Evidence — 何を根拠としているか
- Done — 何を確認したら終わりか
- Revisit — いつ過去の判断を疑うか

---

# 5. 基本原則

## 5.1 Workstream Principle

ひとつのレビュー可能な目的を `Workstream` とする。

Workstreamは複数のPhaseを持つことができる。

---

## 5.2 Facts before Plan

実装計画より先に、必要な現状確認を行う。

推測を事実として扱わない。

この活動を **Fieldwork** と呼ぶ。

---

## 5.3 Characterize before Change

既存挙動を維持する必要がある変更では、変更前の挙動を可能な範囲で再現可能な形にする。

この活動を **Characterize** と呼ぶ。

現在の挙動をCharacterizeすることは、その挙動を望ましい仕様として承認することを意味しない。

---

## 5.4 Explicit Scope

今回変更する範囲と、変更しない範囲を区別する。

必要性が発見されても、現在のScopeに含まれない変更は自動的に追加しない。

---

## 5.5 Phase as Decision Boundary

Phaseは単なる工程区分ではない。

**作業結果を確認し、次へ進むか、方針を変えるか、止めるかを判断する単位**である。

Taskがすべて完了しても、自動的にPhase完了とはしない。

---

## 5.6 Task as Execution Unit

Taskは、独立して実行・確認可能な最小の作業単位とする。

Taskには少なくともGateを持たせる。

---

## 5.7 Decisions are Revisitable

重要な判断には理由と根拠を残す。

さらに、

**どの条件が変わったら判断を再評価するか**

を可能な限り記録する。

---

## 5.8 Evidence-backed Completion

「実装した」「AIが成功と言った」だけでは完了としない。

Gateを満たした根拠を確認する。

未実施・未確認・不明は、それぞれ明示する。

---

## 5.9 Minimum Necessary Structure

WoDD自身の運用を目的化しない。

必要のない文書、Agent、テスト、ログ、抽象化を追加しない。

---

# 6. 基本構造

標準構造は以下とする。

```text
repo/
└─ plans/
   └─ <workstream>/
      ├─ overview.md
      ├─ decisions.md
      │
      ├─ phase1/
      │  ├─ overview.md
      │  ├─ tasks.md
      │  └─ review.md
      │
      ├─ phase2/
      │  └─ ...
      │
      └─ ...
```

必要な場合のみ追加する。

```text
phaseN/
├─ fieldwork.md
└─ characterization.md
```

Fieldwork / Characterization は**概念として存在することが重要であり、常に独立ファイルにする必要はない**。

内容が小さい場合は `overview.md` に含めてもよい。

---

# 7. Workstream

Workstreamは、ひとまとまりの目的と判断履歴を保持する単位である。

例:

```text
plans/
├─ minimal_mcp_001/
├─ refactor_form1cs_001/
└─ python_extension_001/
```

## `overview.md`

最低限、以下を持つ。

```markdown
# <workstream>

## Goal

このWorkstreamで達成すること。

## Context

この仕事が必要になった背景。

## Scope

今回扱う範囲。

## Non-goals

今回は扱わないこと。

## Known Facts

確認済みの重要な事実。

## Unknowns

現在未確認の事項。

## Completion Criteria

Workstream全体を完了と判断する条件。

## Current Phase

現在進行中のPhase。
```

---

# 8. Fieldwork

## 8.1 定義

Fieldworkとは、

**計画や実装に必要な現場の事実を収集する活動**

である。

対象には以下を含む。

- 現在のコード
- テスト
- 実際の実行結果
- API挙動
- 依存関係
- データフロー
- 実行環境
- 既存ドキュメント
- 過去の変更履歴
- 外部仕様

Fieldworkの目的は、設計案を作ることではない。

まず、

**何が事実で、何が推測で、何がまだ分からないか**

を分ける。

---

## 8.2 Fieldworkの出力

最低限、以下を区別する。

```markdown
## Facts
確認できた事実。

## Unknowns
まだ確認できていないこと。

## Observations
設計判断に影響しそうな観測事項。

## Candidate Boundaries
現在見えている変更境界候補。

## Evidence
コード位置、テスト、実測、ログ等。
```

---

## 8.3 Fieldwork Gate

Fieldworkは、

> 次の判断を重大な推測に依存せず行える

状態になれば終了してよい。

すべてを理解する必要はない。

未確認事項が今回の判断に影響しないなら残してよい。

逆に、重要なUnknownが残る場合は無理にPlanへ進まない。

---

# 9. Characterize

## 9.1 定義

Characterizeとは、

**変更対象について、現在観測できる挙動を変更前の基準として固定する活動**

である。

主に以下で使用する。

- リファクタリング
- 移植
- API置換
- 内部構造の分離
- バグ修正
- 互換性維持
- レガシーコード変更
- 外部システムとの接続変更

---

## 9.2 Characterizeと仕様の違い

Characterizationは、

> 現在こう動いている

を記録する。

Specificationは、

> 今後こう動くべきである

を定義する。

両者を混同しない。

現行挙動に問題がある場合は、

```text
Observed Behavior
        ↓
Characterization
        ↓
Change Candidate
        ↓
Decision
        ↓
New Specification
```

として扱う。

---

## 9.3 Characterizationの分類

観測した挙動は可能な限り以下へ分類する。

### Preserve

今回の変更で維持する挙動。

### Change Candidate

現状は確認できたが、変更すべき可能性がある挙動。

現在の変更に便乗して修正しない。

### Unknown

まだ再現または確認できていない挙動。

完了扱いしない。

---

## 9.4 Characterizationの形式

形式は限定しない。

必要に応じて以下を使用できる。

- characterization test
- fixture
- smoke test
- 実行結果表
- before/after snapshot
- ログ
- 入出力例
- 操作手順
- 実機確認記録

---

## 9.5 Characterization Gate

対象変更について、

- 維持すべき既存挙動が識別されている
- 変更候補がPreserveと混在していない
- 必要な挙動に再現または確認方法がある
- Unknownが明示されている

状態になればCharacterizeを終了できる。

---

# 10. Phase

Phaseは、

**次の判断を行うための一まとまりの仕事**

とする。

Phaseは実装Phaseである必要はない。

例えば、

```text
Phase 1: Fieldwork
Phase 2: Characterization
Phase 3: 最小実装
Phase 4: 実環境検証
```

でもよい。

一方、小さい仕事なら同一Phase内に、

```text
Fieldwork
→ Characterize
→ Tasks
→ Review
```

を含めてもよい。

WoDDはPhase数を規定しない。

---

# 11. Phase Overview

`phaseN/overview.md` は最低限以下を持つ。

```markdown
# Phase N — <name>

## Goal

このPhaseで何を成立させるか。

## Inputs

このPhase開始時に利用可能な事実・判断。

## Scope

このPhaseで扱う範囲。

## Non-goals

このPhaseでは扱わないもの。

## Fieldwork

必要な調査。
既に十分なら省略理由を記載してもよい。

## Characterization

維持すべき既存挙動。
不要なら N/A と理由を記載する。

## Phase Gate

Phase完了を判断する条件。
```

---

# 12. Task

TaskはPhase内の実行単位である。

標準形式:

```markdown
## T001 — <name>

### Purpose

このTaskを行う理由。

### Change

変更する内容。

### Scope

変更対象。

### Gate

- [ ] このTaskが成功したと判断できる条件
```

必要なら以下を追加できる。

```markdown
### Read
### Write
### Constraints
### Evidence
```

これらは必須ではない。

---

# 13. Taskの粒度

Taskは可能な限り、

**一つの目的、一つの主要Gate**

で完結させる。

Task中に別の目的が必要になった場合は、

1. 現Taskに本当に必要か確認する
2. 必要ならTaskまたはPhaseを修正する
3. 不要なら別Taskまたは将来候補として残す

AIが発見したという理由だけでScopeを拡張しない。

---

# 14. Decision Log

`plans/<workstream>/decisions.md` はWorkstream内の重要判断を記録する。

すべての細かな実装判断を記録する必要はない。

以下に該当する判断を対象とする。

- 後続Phaseへ影響する
- Scopeを変える
- アーキテクチャへ影響する
- 対案が存在する
- 将来覆る可能性がある
- 後から「なぜそうしたか」が重要になる

標準形式:

```markdown
## D001 — <decision>

### Decision

採用した判断。

### Reason

なぜそう判断したか。

### Evidence

判断時点の根拠。

### Revisit when

この判断を再評価する条件。

### Status

active | revised | superseded
```

---

# 15. Decision Recheck

Decisionは書いた時点で永久固定しない。

各Phase Reviewで、

- 新しいFieldwork結果
- Characterization結果
- 実装結果
- 実測
- 新しい制約

によって既存Decisionの前提が変わっていないか確認する。

必要なら、

```text
keep
revise
supersede
unresolved
```

のいずれかとする。

過去のDecision本文を書き換えて履歴を消すのではなく、新しいDecisionから旧Decisionをsupersedeする。

---

# 16. Phase Review

Taskの完了後、Phase全体を再評価する。

`phaseN/review.md` の標準形式:

```markdown
# Phase N Review

## Result

このPhaseで実際に達成されたこと。

## Evidence

テスト、実機確認、差分、計測等。

## Unresolved

未確認・未解決事項。

## Unexpected Findings

当初想定していなかった発見。

## Decision Recheck

- D001: keep
- D002: revise
- D003: unresolved

## Scope Check

便乗変更や過剰実装が入っていないか。

## Conclusion

complete | continue | revise | stop

## Next

次に行うこと。
```

---

# 17. Phase完了とTask完了の分離

以下を明確に区別する。

```text
Task Done
    ≠
Phase Complete
    ≠
Workstream Complete
```

Taskは実行結果。

Phase Completeは、その結果をまとめて再評価した判断。

Workstream Completeは、Workstream全体のGoalとCompletion Criteriaを満たしたという最終判断である。

---

# 18. Workstream Completion Review

Workstreamを完了する前に最低限確認する。

- Goalを満たしたか
- Completion Criteriaを満たしたか
- 未確認事項を完了扱いしていないか
- Scope外変更が混入していないか
- activeなDecisionの前提が現在も成立するか
- Characterization対象を意図せず変更していないか
- 残件が明示されているか

AIがすべてのTaskをDoneとしたことだけを理由にWorkstreamを完了しない。

---

# 19. 標準フロー

WoDDの基本フローは以下とする。

```text
Workstream
    │
    ▼
Phase Goal
    │
    ▼
Fieldwork
    │
    ├─ 重要なUnknownあり ──→ Fieldwork継続
    │
    ▼
Characterize
    │
    │  ※既存挙動維持が必要な場合
    ▼
Scope / Decisions
    │
    ▼
Tasks
    │
    ▼
Execute
    │
    ▼
Task Gates
    │
    ▼
Phase Review
    │
    ├─ continue
    ├─ revise
    ├─ stop
    └─ next phase
            │
            ▼
      Decision Recheck
            │
            ▼
    Workstream Completion
```

これは固定されたウォーターフォールではない。

新しい事実が発見された場合はFieldworkへ戻ってよい。

---

# 20. Fieldwork / Characterize / Plan の関係

三者は次のように区別する。

| 概念 | 問い |
|---|---|
| Fieldwork | 実際にはどうなっているか |
| Characterize | 現在の挙動のうち何を変更前基準として固定するか |
| Plan | その事実を踏まえて何を変えるか |

順序を逆転させないことを原則とする。

特に、

**Planを立ててから、そのPlanを正当化するためにFieldworkを行わない。**

---

# 21. WoDDにおける仕様

WoDDでは仕様を一度に完成させることを要求しない。

仕様は、

```text
Fieldwork
    ↓
Characterization
    ↓
Decision
    ↓
Phase
    ↓
Implementation / Verification
    ↓
Review
```

を通じて段階的に確定してよい。

したがって、初期の仕様にはUnknownが存在してよい。

Unknownを推測で埋めることより、Unknownとして残すことを優先する。

---

# 22. LoDDとの関係

WoDDはLoDDの後継バージョンではない。

目的が異なる。

## LoDD

主に、

> 定義済みの仕様に従ってAIを局所的に動かす

ことを扱う。

## WoDD

主に、

> 何を仕様として固定すべきかを調査し、段階的に判断しながら仕事を進める

ことを扱う。

WoDDはLoDDから以下を引き継ぐ。

- Workstream
- Phase
- Task
- Scope
- Decision Log
- Gate
- 局所的な作業単位

一方、以下はWoDD Coreでは必須としない。

- 強制的なContext Boundary
- Read/Write以外Deny
- interfaces/ の必須化
- knowledge/ の必須化
- iterations/ の必須化
- Debt Marker
- Agent構成
- New Chat強制
- Lock-down Rules

必要なプロジェクトでは追加してよい。

---

# 23. CoreとExtension

WoDD Coreは次の概念だけで成立する。

```text
Workstream
Fieldwork
Characterize
Phase
Task
Gate
Decision
Review
```

それ以外はExtensionである。

プロジェクトが必要としない仕組みを、WoDDを採用したという理由だけで導入しない。

---

# 24. 最小構成

最小のWoDDは、実際には以下だけでも成立する。

```text
plans/
└─ <workstream>/
   ├─ overview.md
   ├─ decisions.md
   └─ phase1/
      ├─ overview.md
      ├─ tasks.md
      └─ review.md
```

FieldworkとCharacterizationは必要量だけ `overview.md` に記載する。

情報量が増えた場合だけ、

```text
fieldwork.md
characterization.md
```

へ分離する。

---

# 25. 核心

WoDDの目的は文書を増やすことではない。

また、AIを細かく管理することでもない。

目的は、

> **現実を確認し、変更前の基準を把握し、仕事を適切な単位に区切り、判断を後から再評価できる状態でAIに実行させること**

である。

AIにはHowを考えさせる。

WoDDは、

**What / Boundary / Evidence / Enough / Revisit**

を保持する。