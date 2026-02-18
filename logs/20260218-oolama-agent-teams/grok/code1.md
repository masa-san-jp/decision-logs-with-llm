# コードバージョン1: 初期CrewAI + Ollama例

```python
from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama

lead_llm = ChatOllama(model="qwen2.5-coder:7b")
coder_llm = ChatOllama(model="deepseek-coder:6.7b")
# ... (同様に設定)

lead_agent = Agent(role='Team Lead', goal='タスク分割', llm=lead_llm)
# タスク定義、Crew実行
crew = Crew(agents=[lead_agent, ...], tasks=[...])
result = crew.kickoff()
