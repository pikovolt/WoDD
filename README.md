# WoDD — Workstream-driven Development

WoDD（Workstream-driven Development）は、AIを利用した開発作業を、**仕事の単位・境界・判断点**で管理するための軽量な作業フレームワークです。

AIの思考方法や実装手順そのものを細かく固定するのではなく、

- 何を達成するのか
- 現在何が分かっているのか
- 何を変更し、何を変更しないのか
- 何をもって完了とするのか
- どの判断を後で再評価するのか

を外部化します。

> **AIを固定するのではなく、仕事の境界と判断点を固定する。**

## Core flow

```text
Workstream
    ↓
Fieldwork
    ↓
Characterize
    ↓
Decision / Scope
    ↓
Phase
    ↓
Tasks
    ↓
Execute
    ↓
Gate
    ↓
Review
    ↓
continue / revise / stop
```

WoDDでは、仕様が最初から正しいことを前提にしません。

まず **Fieldwork** で現状を調べ、必要に応じて **Characterize** で変更前の挙動を基準として固定します。その事実をもとにScopeと判断を決め、小さなTaskとして実行し、Phaseごとに結果を再評価します。

## Core concepts

### Workstream

ひとつのレビュー可能な目的を持つ仕事の単位です。複数のPhaseと、その間の判断履歴を保持します。

### Fieldwork

計画や実装の前に、現場の事実を集める活動です。

コード、テスト、実行結果、API挙動、依存関係、環境、履歴などを確認し、**Fact / Unknown / Observation** を分けます。

### Characterize

変更対象について、現在観測できる挙動を変更前の基準として固定する活動です。

Characterizationは「現在こう動いている」を記録するものであり、「今後もそう動くべき」という仕様承認とは区別します。

### Phase

単なる工程区分ではなく、**次へ進むか、方針を変えるか、止めるかを判断する単位**です。

Taskがすべて完了しても、自動的にPhase完了とはしません。

### Task

Phase内の独立して実行・確認できる作業単位です。原則として、一つの目的と主要なGateを持ちます。

### Gate

TaskまたはPhaseが成功したと判断するための確認条件です。

「実装した」「AIが成功と言った」だけでは完了とせず、確認可能な根拠を使います。

### Decision

後続作業へ影響する重要な判断を記録します。

判断そのものだけでなく、理由・根拠・**Revisit when（どの条件が変わったら再評価するか）** を残します。

### Review

Phase終了時に、結果、未解決事項、想定外の発見、既存Decisionを見直し、次の行動を決めます。

```text
Task Done
    ≠
Phase Complete
    ≠
Workstream Complete
```

## Minimal structure

```text
repo/
└─ plans/
   └─ <workstream>/
      ├─ overview.md
      ├─ decisions.md
      └─ phase1/
         ├─ overview.md
         ├─ tasks.md
         └─ review.md
```

FieldworkやCharacterizationの情報量が増えた場合だけ、必要に応じて分離します。

```text
phaseN/
├─ fieldwork.md
└─ characterization.md
```

WoDDは文書を増やすこと自体を目的にしません。小さなWorkstreamでは、FieldworkやCharacterizationを `overview.md` に直接記載して構いません。

## Principles

- **Facts before Plan** — 推測を事実として扱わない。
- **Characterize before Change** — 維持すべき現行挙動を把握してから変更する。
- **Explicit Scope** — 今回やることと、やらないことを分ける。
- **Phase as Decision Boundary** — Phaseを人間が判断を取り戻す地点として使う。
- **Decisions are Revisitable** — 過去の判断を永久固定しない。
- **Evidence-backed Completion** — 未確認事項を完了扱いしない。
- **Minimum Necessary Structure** — フレームワーク運用そのものを目的化しない。

## LoDDとの違い

WoDDは、LoDD（Locality-Oriented / Lock-down Driven Development）の単純な縮小版ではありません。

LoDDが主に、**定義済みの仕様に従ってAIを局所的に実装させること**を扱っていたのに対し、WoDDは、

**Fieldwork → Characterize → Decision → Execution → Review**

によって、**何を仕様として固定するか自体を段階的に決めること**を扱います。

強制的なContext Boundary、Read/Write制限、Agent構成、Knowledge階層などはWoDD Coreでは必須ではありません。必要なプロジェクトだけ追加します。

## Specification

詳細な定義、各概念のGate、Decision Log、Phase Review、標準フローについては [WoDD_Reference.md](./WoDD_Reference.md) を参照してください。

---

**Status:** Draft v0.1
