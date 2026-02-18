# 別添資料C：CrewAI版Agent Teams実装コード

**対応フェーズ**: 第3フェーズ（ハードウェア環境最適化）  
**ファイル名**: `agent_team.py`  
**依存パッケージ**: `crewai`, `crewai-tools`, `langchain-ollama`, `filelock`

---

## 概要

CrewAI + Ollamaによる4チームテンプレート実装。gpt-oss:20b向けにreasoning effort制御を組み込んでいる。

## セットアップ

```bash
python3 -m venv agent-teams && source agent-teams/bin/activate
pip install crewai crewai-tools langchain-ollama filelock
```

## 使い方

```bash
python agent_team.py review  'レビュー対象のコードや説明'
python agent_team.py debug   'バグの症状・ログ・再現手順'
python agent_team.py feature '機能仕様'
python agent_team.py research '調査テーマ'

# GX10をリモートで使う場合
OLLAMA_HOST=http://gx10.local:11434 python agent_team.py review '...'
```

## コード全文

```python
#!/usr/bin/env python3
"""
ローカルAgent Teams — gpt-oss:20b + ASUS GX10 最適化版
使い方:
  python agent_team.py review  'レビュー対象のコードや説明'
  python agent_team.py debug   'バグの症状・ログ・再現手順'
  python agent_team.py feature '機能仕様'
  python agent_team.py research '調査テーマ'

環境変数:
  OLLAMA_HOST  — Ollamaの接続先 (default: http://localhost:11434)
  AGENT_MODEL  — 使用モデル (default: gpt-oss:20b)
"""

import os
import sys
import json
from datetime import datetime
from crewai import Agent, Task, Crew, Process
from langchain_ollama import ChatOllama

CONFIG = {
    "model": os.environ.get("AGENT_MODEL", "gpt-oss:20b"),
    "ollama_url": os.environ.get("OLLAMA_HOST", "http://localhost:11434"),
    "context_window": 32768,
    "output_dir": "agent_team_output",
}

def create_llm():
    return ChatOllama(
        model=CONFIG["model"],
        base_url=CONFIG["ollama_url"],
        temperature=0.1,
        num_ctx=CONFIG["context_window"],
    )

def make_agent(role, goal, backstory, reasoning="medium", delegation=False):
    llm = create_llm()
    enhanced_backstory = f"Reasoning: {reasoning}\n\n{backstory}"
    return Agent(
        role=role, goal=goal, backstory=enhanced_backstory,
        llm=llm, verbose=True, allow_delegation=delegation,
    )

def team_review(input_text):
    lead = make_agent("レビューリーダー",
        "3人の専門家のレビュー結果を統合し、優先度付きの改善レポートを作成する",
        "チームのコードレビューを統括するシニアエンジニア。各指摘を優先度・影響度で整理する。",
        reasoning="high", delegation=True)
    security = make_agent("セキュリティレビュアー",
        "セキュリティ脆弱性を網羅的に検出する",
        "OWASP Top 10に精通。インジェクション、認証バイパス、情報漏洩を見逃さない。",
        reasoning="medium")
    perf = make_agent("パフォーマンスレビュアー",
        "パフォーマンスボトルネックを特定する",
        "N+1クエリ、不要なメモリアロケーション、O(n²)アルゴリズムを検出する。",
        reasoning="medium")
    test = make_agent("テスト品質レビュアー",
        "テストカバレッジと品質を評価する",
        "テスト欠落、脆弱なアサーション、境界値テスト不足を指摘する。",
        reasoning="low")

    tasks = [
        Task(description=f"セキュリティ観点でレビュー:\n\n{input_text}\n\n出力: [重要度] [問題] [該当箇所] [修正案]",
             expected_output="セキュリティレビューレポート", agent=security),
        Task(description=f"パフォーマンス観点でレビュー:\n\n{input_text}\n\n出力: [影響度] [問題] [計算量] [改善案]",
             expected_output="パフォーマンスレビューレポート", agent=perf),
        Task(description=f"テスト品質観点でレビュー:\n\n{input_text}\n\n出力: [カテゴリ] [テスト内容] [コード例]",
             expected_output="テスト品質レビューレポート", agent=test),
        Task(description="3名の結果を統合。構成: 1.サマリー 2.即対応(高) 3.次スプリント(中) 4.推奨(低) 5.品質スコアA-F",
             expected_output="統合レビューレポート", agent=lead),
    ]
    return Crew(agents=[lead, security, perf, test], tasks=tasks, process=Process.sequential, verbose=True)

def team_debug(input_text):
    lead = make_agent("デバッグリーダー",
        "各調査員の仮説を評価し、最も有力な根本原因を特定する",
        "本番障害調査をリードしてきたSRE。証拠の強さを客観評価する。",
        reasoning="high", delegation=True)
    layers = [
        ("データ層調査員", "DB・キャッシュ・ストレージ層を調査",
         "接続プール枯渇、デッドロック、インデックス不足を重点調査。"),
        ("アプリケーション層調査員", "アプリロジックを調査",
         "メモリリーク、スレッドセーフティ違反、リソース解放漏れを重点調査。"),
        ("インフラ層調査員", "インフラ・環境を調査",
         "リソース制限、ネットワーク障害、DNS、設定不整合を重点調査。"),
    ]
    investigators = [make_agent(r, g, b, reasoning="medium") for r, g, b in layers]
    tasks = [
        Task(description=f"バグ報告:\n{input_text}\n\n出力: 1.仮説 2.支持証拠 3.反証 4.追加調査手順 5.確信度0-100%",
             expected_output="仮説検証レポート", agent=inv)
        for inv in investigators
    ]
    tasks.append(Task(
        description="3名の仮説を比較。出力: 1.確信度ランキング 2.最有力原因 3.修正案 4.再発防止策",
        expected_output="根本原因分析レポート", agent=lead))
    return Crew(agents=[lead]+investigators, tasks=tasks, process=Process.sequential, verbose=True)

def team_feature(input_text):
    arch = make_agent("アーキテクト", "技術設計書を作成する",
        "クリーンアーキテクチャとDDDの実践者。", reasoning="high")
    back = make_agent("バックエンドエンジニア", "設計書に基づきバックエンドを実装する",
        "型安全性、エラーハンドリング、テスト容易性を重視する。", reasoning="medium")
    front = make_agent("フロントエンドエンジニア", "設計書に基づきフロントエンドを実装する",
        "アクセシビリティ、レスポンシブ、ユーザビリティを重視する。", reasoning="medium")
    rev = make_agent("テックリード", "実装全体の品質を検証し統合する",
        "フロント・バックの統合ポイントを厳しくチェック。", reasoning="high")
    tasks = [
        Task(description=f"機能要件:\n{input_text}\n\n設計書を作成: API仕様、データモデル、コンポーネント構成",
             expected_output="技術設計書", agent=arch),
        Task(description="設計書に基づきバックエンド実装: ルーター、サービス層、テスト",
             expected_output="バックエンドコード", agent=back),
        Task(description="設計書・API仕様に基づきフロントエンド実装: UIコンポーネント、APIクライアント",
             expected_output="フロントエンドコード", agent=front),
        Task(description="全成果物を統合レビュー。修正済み完成コードを出力。",
             expected_output="統合レビュー+完成コード", agent=rev),
    ]
    return Crew(agents=[arch, back, front, rev], tasks=tasks, process=Process.sequential, verbose=True)

def team_research(input_text):
    lead = make_agent("リサーチリーダー", "調査結果を統合し推奨事項を含むレポートを作成する",
        "バイアスを排除した客観的分析を行う。", reasoning="high", delegation=True)
    researchers = [make_agent(f"調査員{c}", "割り当てテーマを深く掘り下げる",
        "メリット・デメリット両面を公平に評価。", reasoning="medium")
        for c in "ABC"]
    tasks = [
        Task(description=f"テーマ: {input_text}\n\n「賛成側」を調査: 採用理由、成功事例、定量メリット",
             expected_output="賛成側レポート", agent=researchers[0]),
        Task(description=f"テーマ: {input_text}\n\n「反対側・リスク」を調査: 不採用理由、失敗事例",
             expected_output="反対側レポート", agent=researchers[1]),
        Task(description=f"テーマ: {input_text}\n\n「代替案」を調査: 3つの代替（比較表付き）",
             expected_output="代替案レポート", agent=researchers[2]),
        Task(description="統合レポート: 1.サマリー 2.比較マトリクス 3.推奨案 4.実行計画 5.保留ポイント",
             expected_output="意思決定レポート", agent=lead),
    ]
    return Crew(agents=[lead]+researchers, tasks=tasks, process=Process.sequential, verbose=True)

TEAMS = {"review": team_review, "debug": team_debug, "feature": team_feature, "research": team_research}

def main():
    if len(sys.argv) < 3:
        print(__doc__)
        sys.exit(0)
    team_type, input_text = sys.argv[1], " ".join(sys.argv[2:])
    if team_type not in TEAMS:
        print(f"エラー: '{team_type}' は未定義。利用可能: {', '.join(TEAMS.keys())}")
        sys.exit(1)
    print(f"{'='*60}\nAgent Team: {team_type}\nモデル: {CONFIG['model']}\n"
          f"Ollama: {CONFIG['ollama_url']}\n開始: {datetime.now().isoformat()}\n{'='*60}")
    crew = TEAMS[team_type](input_text)
    result = crew.kickoff()
    os.makedirs(CONFIG["output_dir"], exist_ok=True)
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    out = os.path.join(CONFIG["output_dir"], f"{team_type}_{ts}.md")
    with open(out, "w") as f:
        f.write(f"# Agent Team: {team_type}\n\n**入力**: {input_text[:300]}\n\n"
                f"**モデル**: {CONFIG['model']}\n**日時**: {datetime.now().isoformat()}\n\n---\n\n{result}")
    print(f"\n完了。結果: {out}")

if __name__ == "__main__":
    main()
```
