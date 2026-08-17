# LangChain Agents

> Build production-ready AI agents using LangChain's agent framework with tool calling, memory, and multi-step reasoning.

## When to Use

| Scenario | Use LangChain Agents? |
|----------|----------------------|
| Need tool-calling with structured outputs | Yes |
| Rapid prototyping of conversational agents | Yes |
| Complex multi-step workflows | Yes |
| Need fine-grained control over LLM calls | Consider custom agents |
| Multi-agent collaboration | Consider AutoGen or CrewAI |
| Simple chatbot without tools | Overkill — use LangChain ChatModel directly |

## Architecture

```
┌─────────────────────────────────────────────┐
│                User Input                    │
└─────────────────┬───────────────────────────┘
                  ▼
┌─────────────────────────────────────────────┐
│            Agent Executor                    │
│  ┌───────────────────────────────────────┐  │
│  │         Agent (LLM + Prompt)          │  │
│  │  ┌─────────┐  ┌──────────────────┐   │  │
│  │  │ System  │  │  Chat History    │   │  │
│  │  │ Prompt  │  │  (Memory)        │   │  │
│  │  └─────────┘  └──────────────────┘   │  │
│  └───────────────────────────────────────┘  │
│                    │                         │
│        ┌───────────┼───────────┐             │
│        ▼           ▼           ▼             │
│  ┌──────────┐ ┌─────────┐ ┌──────────┐     │
│  │  Tool 1  │ │ Tool 2  │ │ Tool N   │     │
│  │ (Search) │ │ (Code)  │ │ (Custom) │     │
│  └──────────┘ └─────────┘ └──────────┘     │
└─────────────────────────────────────────────┘
```

## Code Template: Basic Agent with Tools

```python
"""LangChain agent with custom tools — ready to run."""
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage
from langgraph.prebuilt import create_react_agent

# 1. Define tools
@tool
def search_web(query: str) -> str:
    """Search the web for current information."""
    # Replace with real search API (Tavily, SerpAPI, etc.)
    return f"Results for: {query}"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))  # Use a safe math parser in production
    except Exception as e:
        return f"Error: {e}"

@tool
def read_file(filepath: str) -> str:
    """Read the contents of a local file."""
    with open(filepath, "r") as f:
        return f.read()

# 2. Initialize LLM
llm = ChatOpenAI(model="gpt-4o", temperature=0)

# 3. Create agent (LangGraph-based in 2025+)
tools = [search_web, calculate, read_file]
agent = create_react_agent(llm, tools)

# 4. Run
response = agent.invoke({
    "messages": [HumanMessage(content="What is the population of Tokyo?")]
})
print(response["messages"][-1].content)
```

## Code Template: Agent with Memory

```python
"""Agent with persistent conversation memory."""
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver

llm = ChatOpenAI(model="gpt-4o", temperature=0)
tools = [search_web, calculate]

# Use MemorySaver for in-process persistence
# Swap with SqliteSaver or PostgresSaver for production
memory = MemorySaver()
agent = create_react_agent(llm, tools, checkpointer=memory)

# Each conversation needs a unique thread_id
config = {"configurable": {"thread_id": "user-123"}}

# First turn
r1 = agent.invoke(
    {"messages": [HumanMessage(content="My name is Alice.")]},
    config=config,
)
print(r1["messages"][-1].content)

# Second turn — agent remembers
r2 = agent.invoke(
    {"messages": [HumanMessage(content="What's my name?")]},
    config=config,
)
print(r2["messages"][-1].content)  # "Your name is Alice."
```

## Code Template: Custom Agent with System Prompt

```python
"""Agent with a tailored system persona."""
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage
from langgraph.prebuilt import create_react_agent

SYSTEM_PROMPT = """You are a senior data analyst agent.
When given a dataset path:
1. Read the file
2. Compute summary statistics
3. Identify anomalies
4. Return a structured report in JSON

Always explain your reasoning step by step."""

llm = ChatOpenAI(model="gpt-4o", temperature=0)
agent = create_react_agent(
    llm,
    tools=[read_file, calculate, search_web],
    state_modifier=SystemMessage(content=SYSTEM_PROMPT),
)

response = agent.invoke({
    "messages": [HumanMessage(content="Analyze /data/sales_q3.csv")]
})
```

## Code Template: Streaming Agent Responses

```python
"""Stream agent reasoning steps in real time."""
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent

llm = ChatOpenAI(model="gpt-4o", temperature=0, streaming=True)
agent = create_react_agent(llm, tools=[search_web, calculate])

# Stream token-by-token
for event in agent.stream(
    {"messages": [HumanMessage(content="Summarize AI trends in 2026")]},
    stream_mode="messages",
):
    msg = event.get("messages", [None])[-1]
    if msg and hasattr(msg, "content") and msg.content:
        print(msg.content, end="", flush=True)
```

## Patterns

### Pattern: Tool Routing with Structured Output
```python
from pydantic import BaseModel, Field

class AgentDecision(BaseModel):
    """Structured output for routing decisions."""
    reasoning: str = Field(description="Step-by-step reasoning")
    action: str = Field(description="Chosen action: search, calculate, or respond")
    tool_input: str = Field(description="Input for the chosen tool")

llm_with_structure = llm.with_structured_output(AgentDecision)
```

### Pattern: Retry and Fallback
```python
from langchain_core.runnables import RunnableLambda

def safe_tool_invocation(tool, input_str: str) -> str:
    """Invoke a tool with automatic retry on failure."""
    for attempt in range(3):
        try:
            return tool.invoke(input_str)
        except Exception as e:
            if attempt == 2:
                return f"Tool failed after 3 attempts: {e}"
```

### Pattern: Parallel Tool Calls
```python
# LangGraph agents support parallel tool calls out of the box
# The LLM can request multiple tools in a single response
response = agent.invoke({
    "messages": [HumanMessage(
        content="Search for Python 3.13 features AND calculate 42 * 17"
    )]
})
# Both tools execute in the same step
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Infinite loops | Agent keeps calling tools without converging | Set `recursion_limit` on AgentExecutor |
| Tool hallucination | LLM invents tool names or parameters | Use `with_structured_output()` for strict schemas |
| Context overflow | Long conversations exceed token limit | Use summarization memory or sliding window |
| Blocking tools | Slow API calls block the entire agent | Use `async` tool implementations |
| Error swallowing | Tool errors silently ignored | Add explicit error handling in tools |
| Cost explosion | Unbounded tool calls in a loop | Set `max_iterations` and track token usage |

## Best Practices

1. **Always define tool docstrings** — the LLM uses them to decide which tool to call
2. **Use Pydantic for tool inputs** — gives the LLM structured schemas with descriptions
3. **Set recursion limits** — default is 25, lower it for cost-sensitive applications
4. **Log everything** — use LangSmith for tracing agent decisions and tool calls
5. **Test with adversarial inputs** — agents can be tricked into calling tools maliciously
6. **Prefer LangGraph over legacy AgentExecutor** — LangGraph is the officially recommended approach

## Dependencies

```bash
pip install langchain langchain-openai langgraph langchain-core
# Optional: langsmith for tracing
pip install langsmith
```

## See Also

- [Custom Agents](./custom-agents.md) — Build agents without frameworks
- [Function Calling](../工具调用/function-calling.md) — Underlying tool-use mechanism
- [RAG Memory](../记忆系统/rag-memory.md) — Add knowledge retrieval to agents
- [Task Decomposition](../多Agent协作/task-decomposition.md) — Break complex tasks into subtasks
