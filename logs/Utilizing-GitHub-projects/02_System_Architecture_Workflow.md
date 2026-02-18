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
