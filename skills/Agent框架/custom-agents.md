# Building Custom Agents from Scratch

> Build AI agents without heavy frameworks — just Python, an LLM API, and a loop. Understand the fundamentals before reaching for frameworks.

## When to Use

| Scenario | Build Custom? |
|----------|--------------|
| Learning how agents work | Yes (best way to learn) |
| Minimal dependencies / fast startup | Yes |
| Need full control over every decision | Yes |
| Simple tool-calling agent | Yes |
| Complex multi-agent workflows | Consider LangGraph or AutoGen |
| Rapid prototyping with many tools | Consider LangChain |
| Production with observability needs | Consider LangGraph + LangSmith |

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Custom Agent Loop                   │
│                                                      │
│  ┌────────┐   ┌─────────────┐   ┌───────────────┐  │
│  │  Input  │──►│  Think      │──►│  Decide       │  │
│  │  Parse  │   │  (LLM Call) │   │  (Act/Reply)  │  │
│  └────────┘   └─────────────┘   └───────┬───────┘  │
│                                          │           │
│                          ┌───────────────┼──────┐   │
│                          │               │      │   │
│                          ▼               ▼      ▼   │
│                    ┌──────────┐   ┌─────────┐       │
│                    │ Call Tool│   │ Final   │       │
│                    │ & Observe│   │ Answer  │       │
│                    └────┬─────┘   └─────────┘       │
│                         │                            │
│                         ▼                            │
│                   ┌───────────┐                      │
│                   │ Add to    │                      │
│                   │ History   │──► Loop back         │
│                   └───────────┘                      │
└─────────────────────────────────────────────────────┘
```

## Code Template: Minimal ReAct Agent

```python
"""A complete ReAct agent in ~80 lines of pure Python."""
import json
from openai import OpenAI

client = OpenAI()

# --- Tool definitions ---
TOOLS = {
    "search_web": lambda query: f"Search results for '{query}': [mock data]",
    "calculate": lambda expr: str(eval(expr)),
    "read_file": lambda path: open(path).read(),
}

TOOL_SCHEMAS = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for information",
            "parameters": {
                "type": "object",
                "properties": {"query": {"type": "string", "description": "Search query"}},
                "required": ["query"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Evaluate a math expression",
            "parameters": {
                "type": "object",
                "properties": {"expr": {"type": "string", "description": "Math expression"}},
                "required": ["expr"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read a file's contents",
            "parameters": {
                "type": "object",
                "properties": {"path": {"type": "string", "description": "File path"}},
                "required": ["path"],
            },
        },
    },
]

SYSTEM_PROMPT = "You are a helpful agent. Use tools when needed. Think step by step."

def run_agent(user_message: str, max_turns: int = 10) -> str:
    """Run the agent loop until it produces a final answer."""
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": user_message},
    ]

    for turn in range(max_turns):
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=TOOL_SCHEMAS,
            tool_choice="auto",
        )

        msg = response.choices[0].message
        messages.append(msg)

        # No tool calls — agent is done
        if not msg.tool_calls:
            return msg.content

        # Execute each tool call
        for tool_call in msg.tool_calls:
            fn_name = tool_call.function.name
            fn_args = json.loads(tool_call.function.arguments)

            if fn_name in TOOLS:
                result = TOOLS[fn_name](**fn_args)
            else:
                result = f"Error: unknown tool '{fn_name}'"

            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": str(result),
            })

    return "Agent reached max turns without a final answer."


if __name__ == "__main__":
    print(run_agent("What is 42 * 17 + the number of US states?"))
```

## Code Template: Agent with Conversation Memory

```python
"""Agent that maintains conversation history across turns."""
from dataclasses import dataclass, field
from typing import Callable
import json
from openai import OpenAI

@dataclass
class Agent:
    name: str
    system_prompt: str
    model: str = "gpt-4o"
    tools: dict[str, Callable] = field(default_factory=dict)
    tool_schemas: list[dict] = field(default_factory=list)
    max_turns: int = 10
    history: list[dict] = field(default_factory=list)
    client: OpenAI = field(default_factory=OpenAI)

    def __post_init__(self):
        if not self.history:
            self.history = [{"role": "system", "content": self.system_prompt}]

    def chat(self, user_message: str) -> str:
        """Send a message and get a response, preserving history."""
        self.history.append({"role": "user", "content": user_message})

        for _ in range(self.max_turns):
            response = self.client.chat.completions.create(
                model=self.model,
                messages=self.history,
                tools=self.tool_schemas or None,
                tool_choice="auto" if self.tool_schemas else None,
            )

            msg = response.choices[0].message
            self.history.append(msg)

            if not msg.tool_calls:
                return msg.content

            for tc in msg.tool_calls:
                fn = tc.function
                args = json.loads(fn.arguments)
                result = self.tools.get(fn.name, lambda **k: "Unknown tool")(**args)
                self.history.append({
                    "role": "tool",
                    "tool_call_id": tc.id,
                    "content": str(result),
                })

        return "Max turns reached."

    def reset(self):
        """Clear conversation history."""
        self.history = [{"role": "system", "content": self.system_prompt}]


# Usage
agent = Agent(
    name="helper",
    system_prompt="You are a helpful assistant with access to tools.",
    tools={"calculate": lambda expr: str(eval(expr))},
    tool_schemas=[{
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Evaluate a math expression",
            "parameters": {
                "type": "object",
                "properties": {"expr": {"type": "string"}},
                "required": ["expr"],
            },
        },
    }],
)

print(agent.chat("What is 2+2?"))
print(agent.chat("Now multiply that by 10"))  # Remembers the previous answer
```

## Code Template: Streaming Agent with Observability

```python
"""Agent with streaming output and decision logging."""
import json
from dataclasses import dataclass, field
from openai import OpenAI

@dataclass
class AgentLog:
    turn: int
    role: str
    content: str
    tool_calls: list = field(default_factory=list)
    tool_results: list = field(default_factory=list)

class ObservableAgent:
    def __init__(self, system_prompt: str, tools: dict, schemas: list):
        self.client = OpenAI()
        self.system_prompt = system_prompt
        self.tools = tools
        self.schemas = schemas
        self.logs: list[AgentLog] = []

    def run(self, query: str, stream: bool = True) -> str:
        messages = [
            {"role": "system", "content": self.system_prompt},
            {"role": "user", "content": query},
        ]

        for turn in range(10):
            response = self.client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                tools=self.schemas,
                stream=stream,
            )

            # Collect streamed response
            if stream:
                collected = {}
                for chunk in response:
                    delta = chunk.choices[0].delta
                    if delta.content:
                        print(delta.content, end="", flush=True)
                    if delta.tool_calls:
                        for tc in delta.tool_calls:
                            idx = tc.index
                            if idx not in collected:
                                collected[idx] = {"id": "", "name": "", "args": ""}
                            if tc.id:
                                collected[idx]["id"] = tc.id
                            if tc.function and tc.function.name:
                                collected[idx]["name"] = tc.function.name
                            if tc.function and tc.function.arguments:
                                collected[idx]["args"] += tc.function.arguments

                if not collected:
                    print()
                    return "Done"

                # Process tool calls
                for idx, tc_data in collected.items():
                    args = json.loads(tc_data["args"])
                    result = self.tools[tc_data["name"]](**args)
                    print(f"\n[Tool: {tc_data['name']}({args}) → {result}]")
                    messages.append({
                        "role": "assistant",
                        "content": None,
                        "tool_calls": [{
                            "id": tc_data["id"],
                            "type": "function",
                            "function": {
                                "name": tc_data["name"],
                                "arguments": tc_data["args"],
                            },
                        }],
                    })
                    messages.append({
                        "role": "tool",
                        "tool_call_id": tc_data["id"],
                        "content": str(result),
                    })
```

## Code Template: Agent with Structured Output

```python
"""Force the agent to respond in a specific JSON schema."""
from pydantic import BaseModel
from openai import OpenAI

class AgentResponse(BaseModel):
    thinking: str
    action: str  # "use_tool" or "answer"
    tool_name: str | None = None
    tool_args: dict | None = None
    answer: str | None = None

def structured_agent(query: str) -> AgentResponse:
    client = OpenAI()
    response = client.beta.chat.completions.parse(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Decide: use a tool or answer directly."},
            {"role": "user", "content": query},
        ],
        response_format=AgentResponse,
    )
    return response.choices[0].message.parsed
```

## Patterns

### Pattern: Planner-Executor Split
```python
# Step 1: Plan with a strong model
plan = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": f"Create a plan for: {task}"}],
)

# Step 2: Execute each step with a cheaper model
for step in parse_plan(plan):
    result = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": step}],
    )
```

### Pattern: Tool Registry with Decorators
```python
TOOL_REGISTRY = {}

def tool(name: str, description: str, schema: dict):
    def decorator(fn):
        TOOL_REGISTRY[name] = fn
        fn._tool_schema = {
            "type": "function",
            "function": {"name": name, "description": description, "parameters": schema},
        }
        return fn
    return decorator

@tool("search", "Search the web", {"type": "object", "properties": {"q": {"type": "string"}}})
def search(q: str) -> str:
    return f"Results for {q}"
```

### Pattern: Self-Correcting Agent
```python
def self_correcting_agent(query: str, max_retries: int = 3):
    """Agent that reflects on errors and tries again."""
    for attempt in range(max_retries):
        result = run_agent(query)
        critique = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": f"Is this correct? {result}"}],
        )
        if "correct" in critique.choices[0].message.content.lower():
            return result
        query = f"Previous attempt was wrong: {result}. Critique: {critique.choices[0].message.content}. Try again."
    return result
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| No error handling | Tool failure crashes the agent | Wrap every tool call in try/except |
| Unbounded loops | Agent never terminates | Always enforce `max_turns` |
| Context window overflow | History grows without limit | Truncate or summarize old messages |
| Tool injection | User tricks agent into calling tools maliciously | Validate tool inputs, sanitize prompts |
| Blocking I/O | Synchronous tool calls block streaming | Use async tools with `asyncio` |
| Race conditions | Concurrent tool calls interfere | Use locks or sequential execution |

## Best Practices

1. **Start simple** — a basic ReAct loop handles 80% of use cases
2. **Add tools incrementally** — don't build a tool registry until you need one
3. **Use structured output for decisions** — Pydantic models prevent hallucinated tool names
4. **Log every LLM call and tool invocation** — essential for debugging
5. **Separate planning from execution** — use strong models to plan, cheaper ones to execute
6. **Test with adversarial inputs** — agents are vulnerable to prompt injection

## See Also

- [LangChain Agents](./langchain-agents.md) — Framework-based approach
- [Function Calling](../工具调用/function-calling.md) — Underlying tool mechanism
- [Long-Term Memory](../记忆系统/long-term-memory.md) — Add persistent memory
- [MCP Integration](../工具调用/mcp-integration.md) — Use MCP servers as tools
