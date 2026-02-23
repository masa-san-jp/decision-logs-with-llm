# AIネイティブ・プロジェクト管理体制 検討会議事録

**日時:** 2026-02-19
**テーマ:** ビジネスサイド主導によるGitHub/GWS活用とAI開発フローの構築
**参加者:** プロジェクトオーナー（ビジネスサイド）、AIアシスタント（Gemini）

## 1. 背景と目的
* ビジネスサイド（PM/企画）が開発プロセスにより深く関与し、透明性を高めたい。
* Notion、Dropbox、GitHub、Google Workspaceとツールが散在しており、コストと管理コストを最適化したい。
* 最新のAIツール（NotebookLM, Claude Code等）を前提とした、少人数で高速に回る開発体制を作りたい。

## 2. 主な決定事項

### 2.1 ツールスタックの刷新（断捨離）
* **決定:** ツールを「GitHub」と「Google Workspace (GWS)」の2つに集約する。
* **理由:**
    * **Notion:** タスク管理機能はGitHub Projectsへ移行。ドキュメント機能はGoogle Docsへ移行。
    * **Dropbox:** ファイル管理をGoogle Driveへ完全移行し、コスト削減と検索性向上を図る。
    * **Webクリッパー:** 情報収集は「Google Keep（参考資料）」と「GitHub Issue（タスク）」に使い分ける。

### 2.2 リポジトリとプロジェクト構成
* **決定:** 「2つの箱（リポジトリ）」と「1つの机（プロジェクト）」で管理する。
    * `Internal Repo` (Private): 企画、要件、議事録など「AIのコンテキスト」となる情報。
    * `Core Repo` (Public/Private): 成果物となるソースコード。
* **決定:** 実装違い（API版 vs Ollama版）でリポジトリを分割せず、コード内の「Adapterパターン」で切り替える（保守コスト削減）。

### 2.3 AIネイティブ・ワークフローの確立
* **フロー:** Google Drive (資料) → NotebookLM (要約) → GitHub Issue (タスク化) → Claude Code (実装) というパイプラインを構築する。
* **狙い:** ビジネスサイドは「良質なコンテキスト（情報）」を投入することに集中し、実装はAIとエンジニアに任せる。

### 2.4 ガバナンスとコスト戦略
* **初期 (PoC):** GitHub Freeプランで開始。閲覧者にはGitHubアカウントを発行せず、GASによる自動レポート（Google Chat/Docs）で共有する。
* **拡大期:** セキュリティ（ブランチ保護）が必要になった段階でTeamプランへ移行。
* **完成形:** Google Workspace連携（SAML SSO）が必須となる規模でEnterpriseプランへ移行。

## 3. 今後のアクションアイテム (ToDo)
1.  **無料Organizationの作成:** `my-poc-org` 等を作成し、Internal/Coreリポジトリを用意する。
2.  **GASの試作:** GitHubのタスクをGoogle Chatに通知するスクリプトをClaude Codeに書かせて実装する。
3.  **情報集約:** 既存のNotion/DropboxのデータをGoogle Driveへ移行開始する。

---
**添付資料:**
* 01_ツール移行・統合計画書.md
* 02_システム構成・ワークフロー図.md
* 03_プロジェクト管理体制_設計仕様書.md
