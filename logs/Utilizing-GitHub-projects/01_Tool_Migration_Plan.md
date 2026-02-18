# ツール移行・統合計画書

## 1. 移行の基本方針
* **脱サイロ化:** 情報が分散している現状（Notion, Dropbox, Google Drive）を解消する。
* **エンジニアとの共通言語化:** 開発の「聖域」であるGitHubにビジネスサイドも参画する。
* **コスト最適化:** 重複機能を持つツール（Dropbox等）を解約する。

## 2. ツールマッピング・移行表

| カテゴリ | 旧ツール | **新ツール** | 移行・運用のポイント |
| :--- | :--- | :--- | :--- |
| **タスク管理** | Notion | **GitHub Projects** | ・カンバン、ガントチャートを使用<br>・エンジニアの作業とリアルタイム連動させる |
| **ファイル管理** | Dropbox | **Google Drive** | ・Google Workspaceに一元化<br>・NotebookLMのソースとして活用しやすくする |
| **ドキュメント** | Notion / Word | **Google Docs** | ・議事録や仕様書はMarkdownまたはDocsで作成<br>・GitHubのInternalリポジトリで版管理 |
| **Wiki / ナレッジ** | Notion | **Google Docs + GitHub** | ・ストック情報はDocs、フロー情報はGitHub Issue<br>・検索はDriveの横断検索を利用 |
| **Webクリップ** | Notion Clipper | **Google Keep / Issue** | ・「あとで読む」→ Keep<br>・「タスク化する」→ GitHub Issue (拡張機能利用) |
| **チャット** | Slack / Discord | **Google Chat** | ・GitHub通知を集約<br>・GASによる自動レポート通知の実装 |

## 3. 廃止予定ツールと解約フロー
1.  **Dropbox:** 全ファイルをGoogle Drive（共有ドライブ）へ移行完了後、解約。
2.  **Notion:** タスクデータのCSVエクスポート → GitHub Projectsへのインポート検証後、解約（またはFreeプランへダウングレード）。
