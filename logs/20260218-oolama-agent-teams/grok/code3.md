from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama

# LLM設定 (小モデルでギリギリテスト)
lead_llm = ChatOllama(model="qwen2.5-coder:7b", temperature=0.7)
coder_llm = ChatOllama(model="starcoder2:3b", temperature=0.5)
tester_llm = ChatOllama(model="codegemma:7b", temperature=0.3)

# エージェント定義
lead_agent = Agent(
    role='Team Lead',
    goal='タスクを分割し、チームをコーディネート',
    backstory='経験豊富なプロジェクトマネージャー。ステップバイステップで考える。',
    llm=lead_llm,
    verbose=True
)
# ... (同様)

# タスク定義 (共有メモリで連携)
task1 = Task(description='レガシーPythonコードを解析・分割', agent=lead_agent)
# ...

crew = Crew(agents=[lead_agent, coder_agent, tester_agent], tasks=[task1, task2, task3], memory=True)
result = crew.kickoff(inputs={'code_base': 'your_legacy_code.py'})
print(result)
