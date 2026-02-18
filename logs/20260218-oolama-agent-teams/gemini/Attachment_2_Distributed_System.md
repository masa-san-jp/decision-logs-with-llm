```markdown
# 別添資料2: 分散型エージェントシステム（Mac + ASUS）

## 概要
Mac（高性能モデル/指揮官）から、ネットワーク経由でASUS（実行用モデル/作業員）のOllamaを操作する実装パターン。
`base_url` を指定することでリモート推論を実現する。

## コード (Python)

```python
import os
from langchain_ollama import ChatOllama
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 設定
ASUS_IP = "192.168.1.XX"  # ASUSのIPアドレス

# --- モデル定義 ---

# 【Mac上のモデル】: 指揮官 (Brain)
# 128GBメモリを活かした70Bモデル
manager_llm = ChatOllama(
    model="llama3.3:70b", 
    temperature=0.1,
    base_url="http://localhost:11434"
)

# 【ASUS上のモデル】: 実働部隊 (Muscle)
# ネットワーク経由で実行
worker_llm = ChatOllama(
    model="gpt-oss-20b", 
    temperature=0,
    base_url=f"http://{ASUS_IP}:11434"
)

# --- ノード定義 ---

def supervisor_node(state):
    """Macが計画を立てる"""
    task = state["task"]
    prompt = f"タスク '{task}' のための詳細な実装指示を書いてください。"
    response = manager_llm.invoke(prompt)
    return {"plan": response.content}

def worker_node(state):
    """ASUSが実装する"""
    plan = state["plan"]
    prompt = f"指示に従いコードを実装せよ: {plan}"
    response = worker_llm.invoke(prompt)
    return {"code_draft": response.content}

def reviewer_node(state):
    """Macがレビューする"""
    code = state["code_draft"]
    prompt = f"このコードをレビューせよ: {code}"
    response = manager_llm.invoke(prompt)
    return {"review_result": response.content}

# --- グラフ構築 ---
# (StateGraphの定義は省略。Supervisor -> Worker -> Reviewer の順で接続)
