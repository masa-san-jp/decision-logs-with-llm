# 別添資料1: 基本的なローカルエージェント構造

## 概要
単一マシン内で「マネージャー（指示役）」と「コーダー（作業役）」が連携する、最も基本的なLangGraphの実装パターン。
Ollama上の異なるモデル（Llama 3.1 と Qwen 2.5 Coder）を使い分ける例。

## コード (Python)

```python
from langchain_ollama import ChatOllama
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

# 1. 共有ステートの定義
class AgentState(TypedDict):
    messages: List[str]  # 会話履歴
    next_agent: str      # 次のターン

# 2. モデルの定義
# マネージャー: 指示出しが得意なモデル
manager_llm = ChatOllama(model="llama3.1", temperature=0)
# コーダー: コーディング特化モデル
coder_llm = ChatOllama(model="qwen2.5-coder", temperature=0)

# 3. ノード（役割）の定義
def manager_node(state):
    # 履歴を見て判断
    response = manager_llm.invoke(f"状況を判断して指示を出せ: {state['messages']}")
    # 実際はここで終了判定ロジックなどが入る
    return {"next_agent": "coder", "messages": [response.content]}

def coder_node(state):
    # 指示に従って作業
    instruction = state['messages'][-1]
    code = coder_llm.invoke(f"コードを書け: {instruction}")
    return {"next_agent": "manager", "messages": [code.content]}

# 4. グラフ構築
workflow = StateGraph(AgentState)
workflow.add_node("manager", manager_node)
workflow.add_node("coder", coder_node)

workflow.set_entry_point("manager")
workflow.add_edge("manager", "coder")
workflow.add_edge("coder", "manager") # ループ構造

app = workflow.compile()
