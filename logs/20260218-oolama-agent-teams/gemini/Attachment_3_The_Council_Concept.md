```markdown
# 別添資料3: "The Council" (評議会) とスマホ連携

## 概要
4つの異なる人格（現実主義、理想主義、効率主義、管理主義）を持たせたエージェントによる合議制システム。
Streamlitを使用し、スマートフォンから議論への参加・承認を行うインターフェースの概念実証。

## コード概念 (Python + Streamlit)

```python
import streamlit as st
from langgraph.graph import StateGraph
from typing import TypedDict

# 1. 評議会のステート
class CouncilState(TypedDict):
    topic: str
    conversation_log: list
    consensus_reached: bool

# 2. 人格ノードの定義 (簡略化)
def pragmatist_agent(state):
    return {"conversation_log": ["現実主義者: リソースと期限の観点から..."]}

def visionary_agent(state):
    return {"conversation_log": ["理想主義者: ユーザー体験を最高にするために..."]}

# 3. Streamlit UI
st.title("🏛️ The Agent Council")

if "history" not in st.session_state:
    st.session_state.history = []

user_input = st.chat_input("何を作りたいですか？")

if user_input:
    st.session_state.history.append({"role": "user", "content": user_input})
    
    # ここでLangGraphを実行し、4人の議論を生成
    # 実際は graph.stream() で回して各エージェントの発言を表示する
    
    st.write("--- 議論開始 ---")
    st.chat_message("assistant", avatar="👿").write("現実主義者: その機能は工数がかかりすぎる。")
    st.chat_message("assistant", avatar="😇").write("理想主義者: しかし、それこそが差別化要因だ。")
    st.chat_message("assistant", avatar="🤓").write("効率主義者: 既存ライブラリXを使えば一瞬だ。")
    
    # 議長によるまとめ
    st.info("議長: マスター、ライブラリXの使用を許可しますか？")
    
    # ユーザーの承認ボタン（次のアクションへ）
    if st.button("プランを承認して工場へ送る"):
        st.success("承認されました。ASUS工場へ指示を送信中...")
        # ASUSへのAPIコール処理
