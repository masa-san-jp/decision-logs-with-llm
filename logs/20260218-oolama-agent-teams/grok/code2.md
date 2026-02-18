from langgraph.graph import StateGraph, END
from typing import TypedDict

class State(TypedDict):
    code: str
    errors: str
    review: str
    plan: str

def coder(state): return {"code": llm.invoke(f"プラン{state['plan']}でコード生成")}
def tester(state): return {"errors": llm.invoke(f"テスト: {state['code']}")}
def reviewer(state): return {"review": llm.invoke(f"レビュー: {state['code'] + state['errors']}")}
def replanner(state): return {"plan": llm.invoke(f"修正プラン: {state['review']}") if state['errors'] else END}

graph = StateGraph(State)
graph.add_node("coder", coder)
# ... add edges with conditions (e.g., if errors, to replanner)
compiled = graph.compile()
final_code = compiled.invoke({"plan": final_plan})
