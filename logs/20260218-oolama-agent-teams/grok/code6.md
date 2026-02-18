from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama
import sys

llm = ChatOllama(model="qwen2.5-coder:3b", temperature=0.7, base_url="http://localhost:11434/v1")

pragmatic = Agent(role='現実主義者', goal='現実的なプラン提案', backstory='コスト・実現性を重視', llm=llm, verbose=True)
# ... (他のエージェント)

def poc_discussion_loop(user_idea, iterations=3):
    plan = user_idea
    for i in range(iterations):
        task1 = Task(description=f'プラン{plan}を現実的に議論・批判', agent=pragmatic)
        # ...
        crew = Crew(agents=[...], tasks=[...], memory=True, verbose=2)
        interim = crew.kickoff()
        print(f"Round {i+1} 議論結果: {interim}")
        print("チームからの質問: (例: 予算は？ リソースは？)")
        user_input = sys.stdin.readline().strip()
        plan += f" 更新: {user_input}"
    final_plan = crew.kickoff()
    return final_plan

result = poc_discussion_loop("シンプルなWebアプリを作りたい")
print("最終プラン: ", result)
