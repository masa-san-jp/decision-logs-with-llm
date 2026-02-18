from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama
import sys

llm = ChatOllama(model="qwen2.5-coder:7b")

pragmatic = Agent(role='現実主義者', goal='現実的なプラン提案', backstory='コスト重視', llm=llm)
# ... (他のエージェント)

def discussion_loop(user_idea, iterations=3):
    plan = user_idea
    for i in range(iterations):
        task1 = Task(description=f'プラン{plan}を現実的に議論', agent=pragmatic)
        # ...
        crew = Crew(agents=[...], tasks=[...], memory=True)
        interim = crew.kickoff()
        print(f"Round {i+1} 質問: {interim['questions']}")
        user_input = sys.stdin.readline().strip()
        plan = interim['updated_plan'] + user_input
    final_plan = crew.kickoff()
    return final_plan + " (テスト設計含む)"
