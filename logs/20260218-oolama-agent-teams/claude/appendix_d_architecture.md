# 別添資料D：Dual-Teamアーキテクチャ設計書

**対応フェーズ**: 第4フェーズ（Dual-Teamアーキテクチャの構想）

---

## 概要

プランニングチーム（4ペルソナ会議）と実行チーム（Coder/Tester/Reviewer/Replanner）による2フェーズ構成のエージェントシステム設計書。ペルソナ定義、会議プロトコル、実行サイクル、Web API設計を含む。

本資料の全文は `dual-team-architecture.md` として別途出力済み。ここでは構造のみ記載する。

---

## 設計書の構成

### 1. 全体アーキテクチャ図

Phase 1（プランニング会議） → 承認 → Phase 2（実行サイクル） → レビュー → 再議論 のフルループ。

### 2. Phase 1: プランニングチーム

#### ペルソナ定義（4名）

各ペルソナは以下の要素をすべて定義：
- 名前・アイコン
- ロール・ゴール
- バックストーリー（思考の癖、口調を含む）
- 他メンバーへの態度（誰と同盟し、誰に突っ込むか）
- reasoning effort設定

| ペルソナ | 核心的な問い | reasoning |
|---------|------------|-----------|
| 現実主義者 | 「動くの？」 | high |
| 理想主義者 | 「本質は？」 | high |
| 効率主義者 | 「めんどい」 | medium |
| 管理主義者 | 「壊れたら？」 | high |

#### 会議プロトコル

- Round 1: 各自初期意見 → 相互批判 → ユーザーへ質問
- Round 2: 回答を踏まえ再議論 → 合意/未合意を整理 → 判断を求める
- Round 3: 収束 → プランドラフト → テスト設計組み込み → 最終確認
- 最終合議: 全員が承認/条件付き/反対を表明

#### 実装コード

`planning_team.py` — PlanningMeetingクラス（インタラクティブ会議）、議事録JSON保存、プランMarkdown生成。

### 3. Phase 2: 実行チーム

#### サイクル構造

Coder → Tester → (全pass → 完了) / (失敗 → Reviewer → Replanner → Coderに戻る)

#### 各エージェントの役割

| エージェント | 入力 | 出力 | 判断 |
|------------|------|------|------|
| Coder | プラン + 現在のコード | ファイル（`--- FILE:` 形式） | — |
| Tester | プラン + コード | テスト結果（pass/fail） | — |
| Reviewer | コード + テスト結果 | VERDICT: BUG or DESIGN | バグかプラン問題かを区別 |
| Replanner | プラン + レビュー結果 | 修正版プラン | 変更箇所に「【修正】」マーカー |

#### 実装コード

`execution_team.py` — ExecutionTeamクラス、Git branch隔離、ファイル抽出書き込み、自動commit。

### 4. 統合オーケストレーター

`orchestrator.py` — Phase 1 → 承認 → Phase 2 → レビュー → 再議論のフルループを管理。

### 5. スマートフォン参加（Web API）

`web_bridge.py` — FastAPI + Swagger UIでGX10上に軽量APIサーバーを構築。Tailscale等でVPNを張れば外出先からも参加可能。

### 6. ファイル構成

```
agent-teams/
├── orchestrator.py
├── planning_team.py
├── execution_team.py
├── web_bridge.py
├── personas.py
├── plans/
└── README.md
```
