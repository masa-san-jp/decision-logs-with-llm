# 別添資料E：Mac簡易版プロトタイプコード

**対応フェーズ**: 第5フェーズ（Mac簡易版プロトタイプの実装）  
**ファイル名**: `dual_team.py`  
**依存パッケージ**: なし（Python 3.8+ 標準ライブラリのみ）  
**コード行数**: 約800行

---

## 概要

Dual-Teamシステムのフルサイクルを1ファイルで実現するプロトタイプ。外部パッケージに一切依存せず、OllamaのREST APIを `urllib` で直接叩く。全エージェントの思考過程がトークンストリーミングでリアルタイム表示される。

## 特徴

- **ゼロ依存**: `pip install` 不要。`python3 dual_team.py` だけで動く
- **トークンストリーミング**: Ollama `stream: true` で1トークンずつリアルタイム表示
- **カラー出力**: 8エージェントを色分け表示
- **議事録・ログ自動保存**: `plans/` ディレクトリにJSON/MD形式で保存
- **Git branch隔離**: 実行チームは自動でbranchを切る
- **言語非依存**: `--- FILE: path ---` 形式でどの言語のコードも書き込み可能

## セットアップ

```bash
ollama pull qwen2.5-coder:7b    # 軽量テスト用
python3 dual_team.py             # 起動
```

## モデル切り替え

```bash
AGENT_MODEL=gpt-oss:20b python3 dual_team.py           # 本番モデル
AGENT_MODEL=qwen2.5-coder:7b python3 dual_team.py      # 軽量テスト
OLLAMA_HOST=http://gx10.local:11434 python3 dual_team.py  # GX10リモート
```

## カラー対応表

| エージェント | 色 | アイコン |
|------------|----|----|
| 現実主義者 | 青 | 🔧 |
| 理想主義者 | マゼンタ | 🌟 |
| 効率主義者 | 黄 | 😴 |
| 管理主義者 | 赤 | 🛡️ |
| Coder | 緑 | 🔨 |
| Tester | シアン | 🧪 |
| Reviewer | 黄 | 📋 |
| Replanner | マゼンタ | 📝 |

## コード全文

本コードは `dual_team.py` として独立ファイル出力済み。

主要クラス構成：

### `PlanningMeeting` クラス

- `run_round(topic, human_input)` — 会議ラウンド1回実行。4ペルソナ順に意見を取得
- `generate_plan(topic)` — 全ラウンドの議事録からファシリテーターが最終プランを合成
- `run(idea)` — 会議全体を管理（最低3ラウンド、最大6ラウンド、人間参加型）

### `ExecutionTeam` クラス

- `run_coder()` — プランに従いコードを実装。`--- FILE: ---` 形式で出力
- `run_tester()` — テストコマンドを生成・実行。結果をサマリー
- `run_reviewer(code, test)` — VERDICT: BUG or DESIGN を判定
- `run_replanner(review)` — 修正版プランを生成。変更箇所に「【修正】」マーカー
- `write_files(output)` — LLM出力からファイルを抽出してディスクに書き込み
- `run()` — サイクル全体を管理（最大5サイクル）

### `main()` 関数

Phase 1 → 承認確認 → Phase 2 → レビューフェーズ → 再議論（オプション）の統合フロー。

### `call_llm_stream()` 関数

Ollama `/api/chat` エンドポイントへのストリーミングHTTPリクエスト。`urllib` のみ使用。レスポンスをチャンク読み取りしてJSONパースし、トークンごとにリアルタイム表示する。
