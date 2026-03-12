# AIネイティブ・プロジェクト管理体制 設計仕様書

**Version:** 1.0
**Last Updated:** 2026-02-19
**Author:** Project Manager

---

## 1. 概要と目的
本プロジェクトは、GitHub ProjectsとGoogle Workspace（GWS）を統合し、AI（Gemini, NotebookLM, Claude Code）をワークフローの中核に据えた「次世代型プロジェクト管理体制」を構築する。
情報の透明性を担保しつつ、ビジネスサイドと開発サイドの分断をなくし、非同期コミュニケーションによる高速な意思決定を実現することを目的とする。

## 2. ツール構成と役割定義

| カテゴリ | ツール | 役割 |
| :--- | :--- | :--- |
| **司令塔** | **GitHub Projects** | タスクの一元管理（Single Source of Truth）。すべての指示はここから始まる。 |
| **知識基盤** | **Google Drive / Docs** | 企画書、議事録、資料置き場。NotebookLMのデータソースとなる。 |
| **連絡網** | **Google Chat** | 日々の会話、Botによる進捗通知。GitHubの更新通知を受け取る。 |
| **実行部隊** | **Human & AI** | 人間（Biz/Dev）とAI（Claude Code/Gemini）が協働してタスクを消化する。 |

---

## 3. リポジトリ構成（2-Box Model）

リポジトリは「情報の機密性」と「成果物の性質」に基づき、以下の2つのみで構成する。
**※実装の違い（API版 vs Ollama版など）でリポジトリを分割せず、コード内のアダプターパターンで対応する。**

### A. `project-name-internal` (Private)
* **用途:** 内部統制、戦略、AIへのコンテキスト提供（The Brain）
* **格納物:**
    * 企画書、要件定義書 (Markdown推奨)
    * 会議議事録 (Google Driveからエクスポート)
    * AI用プロンプトテンプレート
    * デザインドラフト
* **アクセス権:** 全社員（BizはWrite権限、DevはRead権限）

### B. `product-name-core` (Public/Private)
* **用途:** アプリケーション本体、最終成果物（The Body）
* **格納物:**
    * ソースコード
    * リリースノート
    * ユーザー向けドキュメント
* **アクセス権:** 開発チーム（DevはWrite権限、BizはRead権限）

---

## 4. チームと権限設計 (GitHub Teams)

個人への権限付与は行わず、必ずTeamを通して管理する。

| チーム名 | 構成員 | Internal Repo | Core Repo | 備考 |
| :--- | :--- | :--- | :--- | :--- |
| **Admin** | PM, Tech Lead | **Admin** | **Admin** | 設定変更、メンバー管理 |
| **Biz-Team** | 企画, マーケ, 経営 | **Write** | **Read** | コードは触れないが仕様は書ける |
| **Dev-Team** | エンジニア | **Read** | **Write** | 仕様を読み、コードを書く |

---

## 5. 自動化とレポーティング (GAS連携)

GitHubにログインしないメンバー（ステークホルダー）向けに、Google Workspace上で情報を可視化する。

### A. 自動レポートシステム (GAS実装)
Google Apps Script (GAS) を用い、GitHub APIから情報を取得して以下を生成する。

1.  **日報 (Daily):**
    * **通知先:** Google Chat (Project Space)
    * **内容:** 本日完了(Done)したタスク一覧、新規Issue。
2.  **週報 (Weekly):**
    * **通知先:** Google Docs (自動生成) + Chatにリンク通知
    * **内容:** 週間完了タスク要約、翌週の予定、未消化タスクのアラート。
    * **AI加工:** Gemini APIにより「今週のハイライト」を自動要約して冒頭に記載。
3.  **月報 (Monthly):**
    * **通知先:** Google Docs / Slides
    * **内容:** プロジェクト全体進捗、リリース機能一覧。経営報告用。

---

## 6. AI統合ワークフロー

1.  **インプット:** 関連資料・ニュースをGoogle Driveに保存。
2.  **整理:** **NotebookLM** がDriveを読み込み、要約・インサイトを生成。
3.  **タスク化:** 人間が要約を `internal` リポジトリにIssue/Specとして登録。
4.  **実装:** **Claude Code** が `internal` のSpecを読み、`core` リポジトリにコードを実装。
5.  **報告:** GASが実装結果をChat/Docsにレポート。

---

## 7. フェーズ別導入・アップグレード計画

組織の成長とセキュリティ要件に合わせて、以下のトリガーに基づきプランを移行する。

### Phase 1: PoC / 立ち上げ期
* **プラン:** **GitHub Free**
* **対象:** コアメンバー 2〜3名
* **運用ルール:**
    * Organizationを作成し、無料枠でPrivateリポジトリを使用。
    * Google連携はGASによる「レポート通知」のみ実装。
    * 閲覧者（報告対象）はGitHubに招待せず、Google Docs/Chatで報告を見る。
* **コスト:** $0

### Phase 2: チーム拡大・運用本格化
* **プラン:** **GitHub Team ($4/user/month)**
* **移行トリガー:**
    * 開発者が増え、コードの品質担保が必要になった（「勝手にマージさせない」制限）。
    * 誤操作によるデータ消失を防ぐ「ブランチ保護」が必要になった。
    * コードレビュー（承認）をシステムで強制したくなった。
* **運用変化:**
    * `main` ブランチに保護設定を適用。
    * Wiki機能の活用開始。

### Phase 3: 組織化・ガバナンス強化
* **プラン:** **GitHub Enterprise ($21/user/month)**
* **移行トリガー:**
    * メンバーが20名を超え、入退社の管理が煩雑になった。
    * ISMS取得や上場準備などで「アクセスログ（監査ログ）」の提出が必要になった。
    * **「Googleアカウントを削除したらGitHubも入れなくする（SAML SSO）」** が必須要件になった。
* **運用変化:**
    * Google WorkspaceとのSAML SSO連携を有効化。
    * IPアドレス制限（社内ネットワークからのみアクセス許可）などの適用。
