# 議事録：ASUS GX10におけるOpenClawセキュア導入方針

## 1. 開催概要
* **日時:** 2026年2月20日
* **参加者:** ユーザー、Gemini（AIアシスタント）
* **目的:** ASUS Ascent GX10上で自律型AIエージェント「OpenClaw」を安全に稼働させるためのアーキテクチャ設計、および外部チャットチャネルとの連携手順の策定。

## 2. 意思決定プロセスと決定事項

### 2.1. 実行環境の分離（サンドボックス化）
* **課題:** OpenClawは強力な権限を持つため、ホストOS（DGX OS）への直接インストールはシステム破壊やデータセット破損のリスクが極めて高い。
* **決定:** Rootless Dockerコンテナを利用した隔離環境（ゼロトラスト・アーキテクチャ）を構築する。
* **対応:** * GX10固有のDocker BuildKit不具合を修正。
  * 専用のワークスペース（`~/openclaw-secure/workspace`）のみを作成・マウントし、エージェントの物理的なファイルアクセス範囲を厳格に限定する。

### 2.2. ネットワークと認証の最小化
* **課題:** 外部からの不正アクセスや、意図しない第三者からの操作を防ぐ必要がある。
* **決定:** 外部公開を遮断し、ローカルネットワーク内での稼働とトークン認証を必須とする。
* **対応:** セットアップスクリプトにて以下を選択。
  * `Gateway bind: lan`
  * `Gateway auth: token`
  * `Tailscale exposure: Off`

### 2.3. AIの自律性と権限（AgentSkills）の制限
* **課題:** AIが自律的にシステムシェルを操作したり、破壊的なコマンドを実行するリスクの排除。
* **決定:** 危険なスキルのブラックリスト化と、Human-in-the-loop（人間の承認）プロセスの強制。
* **対応:** `system_shell` などのシステム権限に関わるスキルを無効化し、重要操作時にはチャット上での承認を必須（`REQUIRE_APPROVAL=true`）とする。

### 2.4. チャットチャネルとのセキュアな連携方式
* **課題:** ファイアウォールに穴を開けることなく、Telegram、Slack、Discord等の外部アプリから安全に指示を出せるようにする。
* **決定:** Inbound（Webhook等による外部からのアクセス）ではなく、GX10側からポーリング/Socket Modeを行うOutbound通信方式を採用する。
* **対応:** 各チャットアプリでBotを作成し、取得したトークンをCLIまたは `.env` 経由でコンテナに登録。その後、必ず手動でのペアリング承認（`approve`）を行う。

---

## 3. 成果物（各種設定スクリプト・コードリファレンス）

### 成果物A: ホスト環境修正・隔離ディレクトリ準備コマンド
```bash
# Docker BuildKitの不具合修正（GX10固有対応）
sudo systemctl stop docker.service
sudo mv /var/lib/docker/buildkit /var/lib/docker/buildkit-bad
sudo systemctl start docker.service

# 隔離ワークスペースの作成と権限絞り込み
mkdir -p ~/openclaw-secure/workspace
chmod 700 ~/openclaw-secure/workspace
```

### 成果物B: 権限制限設定（openclaw.json スニペット）
```json
"skills": {
  "disable": [
    "system_shell",
    "financial_transactions",
    "network_scan"
  ]
}
// ※併せて環境変数等で REQUIRE_APPROVAL=true を運用し、承認プロセスを強制する
```

### 成果物C: チャネル連携用環境変数（.env スニペット）
```env
# Slack連携用（Socket Mode）
SLACK_APP_TOKEN="xapp-..."
SLACK_BOT_TOKEN="xoxb-..."

# Discord連携用（Message Content Intentの有効化が必須）
DISCORD_BOT_TOKEN="YOUR_DISCORD_BOT_TOKEN"
```

### 成果物D: Telegram連携用 CLIコマンドリファレンス
```bash
# トークンの登録
docker compose exec openclaw-gateway openclaw channels add telegram --token "<Bot Token>"

# ペアリングの承認
docker compose exec openclaw-gateway openclaw pairing approve telegram <ペアリングコード>
```
