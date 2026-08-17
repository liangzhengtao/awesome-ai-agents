# CrewAI Agent Orchestration

> Build role-playing AI agent teams with CrewAI — where each agent has a defined role, goal, and backstory, and they collaborate through structured tasks.

## When to Use

| Scenario | Use CrewAI? |
|----------|------------|
| Role-based agent collaboration | Yes |
| Content generation pipelines (research → write → edit) | Yes |
| Business process automation | Yes |
| Simple single-agent tasks | Overkill — use LangChain or custom |
| Fine-grained control over agent dialogue | Consider AutoGen |
| Complex conditional routing | Consider LangGraph |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                       Crew                            │
│                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │    Agent 1    │  │    Agent 2    │  │  Agent 3   │ │
│  │  Role: Coder  │  │  Role: Critic │  │  Role: PM  │ │
│  │  Goal: Write  │  │  Goal: Review │  │  Goal: Plan│ │
│  │  Tools: [...] │  │  Tools: [...] │  │  Tools: [] │ │
│  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
│         │                  │                 │        │
│         ▼                  ▼                 ▼        │
│  ┌──────────────────────────────────────────────────┐│
│  │              Task Pipeline                        ││
│  │  Task 1 ──► Task 2 ──► Task 3                   ││
│  │  (plan)    (code)     (review)                   ││
│  └──────────────────────────────────────────────────┘│
│                                                       │
│  ┌──────────────────────────────────────────────────┐│
│  │              Process: Sequential / Hierarchical   ││
│  └──────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────┘
```

## Code Template: Basic Crew with Sequential Tasks

```python
"""A three-agent crew: researcher, writer, editor."""
from crewai import Agent, Task, Crew, Process

# Define agents with roles
researcher = Agent(
    role="Senior Research Analyst",
    goal="Find comprehensive, accurate information on the given topic",
    backstory="""You are an experienced researcher who digs deep into topics.
    You always verify facts from multiple sources and provide citations.""",
    verbose=True,
    allow_delegation=False,
)

writer = Agent(
    role="Technical Writer",
    goal="Write clear, engaging content based on research findings",
    backstory="""You are a skilled writer who makes complex topics accessible.
    You use clear structure, examples, and avoid jargon.""",
    verbose=True,
    allow_delegation=False,
)

editor = Agent(
    role="Senior Editor",
    goal="Ensure content is polished, accurate, and well-structured",
    backstory="""You are a meticulous editor with 20 years of experience.
    You check facts, fix grammar, and improve flow.""",
    verbose=True,
    allow_delegation=False,
)

# Define tasks
research_task = Task(
    description="Research the latest developments in AI agents for 2026.",
    expected_output="A detailed research brief with key findings and sources.",
    agent=researcher,
)

writing_task = Task(
    description="Write a 1500-word blog post based on the research.",
    expected_output="A complete blog post in markdown format.",
    agent=writer,
    context=[research_task],  # Depends on research
)

editing_task = Task(
    description="Edit and polish the blog post for publication.",
    expected_output="A final, publication-ready blog post.",
    agent=editor,
    context=[writing_task],  # Depends on writing
)

# Create and run the crew
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, writing_task, editing_task],
    process=Process.SEQUENTIAL,
    verbose=True,
)

result = crew.kickoff()
print(result.raw)
```

## Code Template: Crew with Custom Tools

```python
"""CrewAI agent with custom tools for web search and file I/O."""
from crewai import Agent, Task, Crew, Process
from crewai.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

# Define tool input schema
class SearchInput(BaseModel):
    query: str = Field(description="Search query string")

# Custom tool implementation
class WebSearchTool(BaseTool):
    name: str = "web_search"
    description: str = "Search the web for current information"
    args_schema: Type[BaseModel] = SearchInput

    def _run(self, query: str) -> str:
        # Replace with real API (Tavily, SerpAPI, etc.)
        import requests
        response = requests.get(
            "https://api.tavily.com/search",
            params={"q": query, "max_results": 5},
            headers={"Authorization": "Bearer YOUR_API_KEY"},
        )
        return response.json()

class FileWriterInput(BaseModel):
    filename: str = Field(description="Output filename")
    content: str = Field(description="File content to write")

class FileWriterTool(BaseTool):
    name: str = "write_file"
    description: str = "Write content to a file"
    args_schema: Type[BaseModel] = FileWriterInput

    def _run(self, filename: str, content: str) -> str:
        with open(filename, "w") as f:
            f.write(content)
        return f"Written to {filename}"

# Agent with tools
analyst = Agent(
    role="Market Analyst",
    goal="Analyze market trends and produce a report",
    backstory="You are a data-driven analyst with access to web search.",
    tools=[WebSearchTool(), FileWriterTool()],
    verbose=True,
)

task = Task(
    description="Research and write a market analysis for AI agent tools in 2026.",
    expected_output="A saved markdown report with market data.",
    agent=analyst,
    output_file="market_report.md",
)

crew = Crew(agents=[analyst], tasks=[task], verbose=True)
result = crew.kickoff()
```

## Code Template: Hierarchical Process with Manager

```python
"""Hierarchical crew where a manager delegates to workers."""
from crewai import Agent, Task, Crew, Process

manager = Agent(
    role="Engineering Manager",
    goal="Coordinate the team to deliver a working feature",
    backstory="You are an experienced manager who delegates effectively.",
    allow_delegation=True,  # Manager can delegate
    verbose=True,
)

backend_dev = Agent(
    role="Backend Developer",
    goal="Write clean Python backend code",
    backstory="You specialize in FastAPI and database design.",
    verbose=True,
)

frontend_dev = Agent(
    role="Frontend Developer",
    goal="Build responsive UI components",
    backstory="You specialize in React and TypeScript.",
    verbose=True,
)

feature_task = Task(
    description="Build a user authentication feature with login, signup, and JWT tokens.",
    expected_output="Working backend API and frontend components.",
    agent=manager,  # Manager owns the task
)

crew = Crew(
    agents=[manager, backend_dev, frontend_dev],
    tasks=[feature_task],
    process=Process.HIERARCHICAL,
    manager_llm="gpt-4o",
    verbose=True,
)

result = crew.kickoff()
```

## Code Template: Async Crew Execution

```python
"""Run crew asynchronously for better performance."""
import asyncio
from crewai import Agent, Task, Crew, Process

async def run_analysis(topic: str):
    crew = Crew(
        agents=[researcher, writer, editor],
        tasks=[
            Task(description=f"Research: {topic}", agent=researcher),
            Task(description="Write article", agent=writer),
            Task(description="Edit article", agent=editor),
        ],
        verbose=True,
    )
    return await crew.kickoff_async()

# Run multiple crews in parallel
async def main():
    topics = ["AI agents", "Quantum computing", "CRISPR"]
    results = await asyncio.gather(*[run_analysis(t) for t in topics])
    for r in results:
        print(r.raw[:200])

asyncio.run(main())
```

## Patterns

### Pattern: Fact-Checking Pipeline
```python
fact_checker = Agent(
    role="Fact Checker",
    goal="Verify every claim in the content",
    backstory="You cross-reference claims against known sources.",
    tools=[WebSearchTool()],
)
```

### Pattern: Iterative Refinement
```python
# Run the same crew multiple times for improvement
for iteration in range(3):
    result = crew.kickoff()
    # Feed output back as input for next iteration
    writing_task.description = f"Improve this draft:\n{result.raw}"
```

### Pattern: Output as File
```python
task = Task(
    description="Write the quarterly report",
    expected_output="A well-formatted markdown report",
    agent=writer,
    output_file="reports/q3_report.md",  # Auto-saves to file
)
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Role overlap | Agents duplicate each other's work | Define clear, non-overlapping responsibilities |
| Delegation loops | Manager keeps delegating back and forth | Set `max_iter` and `max_rpm` limits |
| Generic backstories | Vague roles produce generic output | Write specific, detailed backstories |
| No output validation | Agent output may not match expected format | Use `output_pydantic` or `output_json` |
| Token waste | Agents over-explain in verbose mode | Use `verbose=False` in production |

## Best Practices

1. **Write detailed backstories** — they dramatically affect output quality
2. **Use `context` parameter on tasks** — makes data flow explicit between tasks
3. **Set `max_rpm`** — rate-limits API calls to avoid throttling
4. **Use `output_pydantic` for structured outputs** — enforce schemas
5. **Test individual agents first** — before composing them into a crew
6. **Use `Process.HIERARCHICAL` for complex projects** — let the manager decide

## Dependencies

```bash
pip install crewai crewai-tools
# For hierarchical process with manager LLM
pip install langchain-openai
```

## See Also

- [AutoGen Multi-Agent](./autogen-multiagent.md) — Alternative multi-agent framework
- [Agent Orchestration](../多Agent协作/agent-orchestration.md) — High-level patterns
- [Task Decomposition](../多Agent协作/task-decomposition.md) — Breaking tasks apart
- [API Orchestration](../工具调用/api-orchestration.md) — Tool integration patterns
