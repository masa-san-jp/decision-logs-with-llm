# システム構成・AIワークフロー図

## 1. 全体アーキテクチャ

```mermaid
graph TD
    User[ビジネスサイド/PM] -->|資料投入| Drive[Google Drive]
    User -->|タスク作成| GH_Proj[GitHub Projects]
    
    %% 知識形成フェーズ
    Drive -->|Sync| NLM[NotebookLM]
    NLM -->|要約・インサイト| User
    
    %% 開発フェーズ (GitHub)
    subgraph GitHub Organization
        GH_Proj -->|紐付け| Repo_Int[Internal Repo (Private)]
        GH_Proj -->|紐付け| Repo_Core[Core Repo (Public/Private)]
        
        Repo_Int -->|コンテキスト| Claude[Claude Code / AI Dev]
        Claude -->|コード実装| Repo_Core
    end
    
    %% レポーティングフェーズ (GAS)
    Repo_Core & GH_Proj -->|API取得| GAS[Google Apps Script]
    GAS -->|日報通知| Chat[Google Chat]
    GAS -->|週報作成| Docs[Google Docs]
    
    Docs -->|閲覧| Stakeholder[報告対象者]


## 2. リポジトリ構成詳細 (2-Box Model)

### A. Internal Repository (The Brain)

- 役割: プロジェクトの「脳」。AIに読ませるためのコンテキスト置き場。
- 内容:
    - /docs: 議事録、企画書 (Markdown推奨)
    - /prompts: AIへの指示書テンプレート
    - /design: UI/UXドラフト
- 運用: ここに変更が入ると、AIが新しい仕様を理解するトリガーとなる。

### B. Core Repository (The Body)

- 役割: プロジェクトの「体」。実際の製品コード。
- 設計方針: Single Repository / Adapter Pattern
    - API版とLocal LLM (Ollama) 版をリポジトリで分けない。
    - config.yaml 等の設定ファイルで動作モードを切り替える設計とする。
    - これにより、UI修正などが1回で済む保守性の高い構成にする。

### 3. 情報収集フロー

- Step 1: 気になる記事・ニュースを発見。
- Step 2:
    - 資料としてストック → Google Keep へ保存。
    - 即タスク化 → GitHub Issue へ登録（スマホの共有ボタン等）。
    - Step 3: Google Keepの内容は定期的にDocsへまとめ、NotebookLMに再学習させる。

