---
layout: post
title: "claude subagents で開発プロセスを再定義・エージェント化する"
date: 2025-08-10 09:00:00 +0900
categories: LLM
published: true
use_toc: false
---



`roocode` の `/orchestrator` を subagents で代替しつつほしいロールを組んでいたのだが、概ね仕上がってきたので一旦まとめ。

## TL;DR

1. カスタムコマンド `/orchestrator` で依頼
2. 以下のagentが順番に処理
  - Research: `./.claude/agents/researcher.md`
  - Requirements: `./.claude/agents/fn-reqs.md`
  - Architecture: `./.claude/agents/architect.md`
  - Task Breakdown: `./.claude/agents/task-tailor.md`
  - TDD Implementation: `./.claude/agents/tdd.md`
  - Quality Assurance: `./.claude/agents/qa.md`

* <https://github.com/ktrysmt/dotfiles/blob/master/.claude/commands/orchestrator.md>
* <https://github.com/ktrysmt/dotfiles/tree/master/.claude/agents>

agent間の情報連携は issue file を作らせ、agent同士で読み書きさせることでカバー。

agent単体でも呼び出し可(後述)。


## Issue File による状態管理

全ての agent は `.claude/issues/{yyyy-mm-dd}-<kebab-case-summary>.md` という共有ファイルを使用。ファイル名に日付が付くことで project の履歴管理が可能。このファイルには各段階の進捗、成果物、課題が記録され、project 全体の状況を透明化。

各 agent は自身の section を持ち、作業完了時に status を `In Progress` から `Completed` に更新することで、次の agent への handoff を明確化。

## 各エージェントの役割と特徴

### 1. Researcher Agent (`@researcher`)
role:
- 市場調査と技術分析による project 基盤作り

市場調査、競合分析、technology landscape の調査を担当。単なる情報収集ではなく、functional requirements の策定に必要な strategic insight を提供することに重点。

output:
- 市場調査結果と競合分析
- 技術選択肢の feasibility assessment
- implementation の complexity 評価

### 2. Functional Requirements Agent (`@fn-reqs`)
role:
- 構造化された機能要件の定義

researcher の調査結果を元に、scattered な要件を coherent な functional group に整理。特徴的なのは interactive problem decomposition により、ambiguous な要件を具体的で testable な acceptance criteria に変換すること。

output:
- 構造化された functional requirements document
- feature priority matrix
- dependency mapping

### 3. Architect Agent (`@architect`)
role:
- scalable で maintainable なシステム architecture の設計

functional requirements を technical architecture に変換し、component 設計から deployment strategy まで comprehensive に策定。technology stack の選択では、researcher の調査結果と requirements の constraint を両方考慮した pragmatic な判断。

output:
- system architecture diagram
- component specification
- API documentation と database design

### 4. Task Tailor Agent (`@task-tailor`)
role:
- architecture の TDD-friendly な task への分解

architecture を実装可能な単位に分解する critical な段階を担当。各 task は2-4時間で完了可能な size に調整され、clear な test scenario と acceptance criteria が付与。dependency を最小化することで、parallel development を可能化。

output:
- TDD-ready task definition
- test scenario と mock requirement
- task dependency mapping

### 5. TDD Agent (`@tdd`)
role:
- 厳密な Test-Driven Development による実装

Kent Beck の TDD principle に従い、Red-Green-Refactor cycle を徹底。「Tidy First」approach により、structural change と behavioral change を分離し、code quality を維持しながら implementation を推進。

output:
- passing unit test を伴う実装済み component
- integration point の documentation
- code quality metric

### 6. QA Agent (`@qa`)
role:
- 包括的な品質保証と production readiness の評価

individual component から system level まで、全層の quality validation を実施。integration testing、performance benchmark、security assessment を経て、production deployment への最終的な sign-off を実行。

output:
- comprehensive QA report
- production readiness assessment
- monitoring と deployment recommendation

## Criteria の管理

各段階間には明確な validation criteria を設定。例えば、Architecture から Task Breakdown への移行では：

- System architecture が diagram で文書化済み
- Technology stack が選択され正当化済み
- API specification が完成済み
- Database design が確定済み
- Deployment strategy が定義済み
- Implementation roadmap が timeline と共に作成済み

これらの criteria を満たさない限り、次の段階には進行不可。

## 実際の活用方法

### `/orchestrator` による全自動実行

6段階すべてを自動実行する場合は `/orchestrator` 使う。

```bash
/orchestrator "オンライン書店システムを開発したい。ユーザー認証、書籍検索、カート機能、決済機能が必要"
```

これにより、以下の流れで各 agent が順次実行される。

1. `@researcher` が市場調査と技術分析を実施
2. `@fn-reqs` が機能要件を構造化
3. `@architect` が system architecture を設計
4. `@task-tailor` が TDD-friendly な task に分解
5. `@tdd` が各 task を実装
6. `@qa` が包括的な品質保証を実行

### 個別 Agent の直接実行

特定の段階のみ実行したい場合は個別 agent を呼び出し。

```bash
@researcher "e-commerce platform の市場調査を実施してください"
@fn-reqs "調査結果を元に機能要件を構造化してください"
@architect "要件に基づいて scalable な architecture を設計してください"
@task-tailor "architecture を TDD-friendly な task に分解してください"
@tdd "user authentication task を厳密な TDD で実装してください"
@qa "実装済み system の包括的な品質保証を実施してください"
```

個別 agent 呼び出し時の Issue File は、
- 既存ファイルがある場合: `.claude/issues/{yyyy-mm-dd}-<kebab-case-summary>.md` から前段階の成果物を読み取り、該当 section を更新
- 既存ファイルがない場合: **必ず新規 Issue File を作成**し、日付付きのファイル名で project 管理を開始


## 感想

作ってみたものの `/orchestrator` を呼ぶタイミングがあまりなく、 `@task-tailor` や `@researcher` など個別に呼びがちであった。
開発プロセスのメタ認知が進んだというか、解像度をあまり落とさずにagentに書かせられる感覚がある。vibe丸投げとeditorで補完使って書くのの中間といった感じ。

