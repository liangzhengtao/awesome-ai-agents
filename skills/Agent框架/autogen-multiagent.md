# AutoGen Multi-Agent Systems

> Build multi-agent conversations and workflows using Microsoft AutoGen — the leading framework for agent-to-agent collaboration.

## When to Use

| Scenario | Use AutoGen? |
|----------|-------------|
| Multiple agents debating/brainstorming | Yes |
| Code generation with execution feedback | Yes |
| Hierarchical team workflows | Yes |
| Single-agent tool calling | Overkill — use LangChain or custom |
| Real-time streaming conversations | Yes (v0.4+ supports async streaming) |
| Production deployment with observability | Yes (AutoGen + AG2 ecosystem) |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                   AutoGen Runtime                     │
│                                                       │
│  ┌─────────────┐    ┌─────────────┐                  │
│  │   Agent A    │◄──►│   Agent B    │                 │
│  │  (Manager)   │    │  (Worker)    │                 │
│  └──────┬──────┘    └──────┬──────┘                  │
│         │                   │                         │
│         ▼                   ▼                         │
│  ┌─────────────┐    ┌─────────────┐                  │
│  │   Agent C    │    │ Code Executor│                 │
│  │ (Critic/     │    │ (Docker/     │                 │
│  │  Reviewer)   │    │  Local)      │                 │
│  └─────────────┘    └─────────────┘                  │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Group Chat Manager                  │ │
│  │  (Speaker selection, termination, routing)       │ │
│  └─────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

## Code Template: Two-Agent Chat

```python
"""Two agents having a conversation to solve a task."""
from autogen import ConversableAgent

# Agent that generates solutions
assistant = ConversableAgent(
    name="assistant",
    system_message="You are a helpful AI assistant. Solve tasks step by step.",
    llm_config={"model": "gpt-4o", "temperature": 0},
)

# Agent that simulates the user
user_proxy = ConversableAgent(
    name="user_proxy",
    human_input_mode="NEVER",  # No human input — fully automated
    is_termination_msg=lambda msg: msg.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={"work_dir": "coding", "use_docker": False},
)

# Start the conversation
result = user_proxy.initiate_chat(
    assistant,
    message="Write a Python function to find all prime numbers up to 1000.",
    max_turns=5,
)
print(result.summary)
```

## Code Template: Group Chat with Multiple Agents

```python
"""Three-agent group chat: planner, coder, reviewer."""
from autogen import ConversableAgent, GroupChat, GroupChatManager

planner = ConversableAgent(
    name="planner",
    system_message="""You are a project planner. Given a task:
    1. Break it into clear subtasks
    2. Assign each subtask to the coder
    3. After the coder finishes, ask the reviewer to check
    Reply TERMINATE when all subtasks are complete.""",
    llm_config={"model": "gpt-4o"},
)

coder = ConversableAgent(
    name="coder",
    system_message="""You are an expert Python developer. Write clean, tested code
    for each subtask assigned by the planner. Include docstrings and type hints.""",
    llm_config={"model": "gpt-4o"},
    code_execution_config={"work_dir": "output", "use_docker": False},
)

reviewer = ConversableAgent(
    name="reviewer",
    system_message="""You are a senior code reviewer. For each piece of code:
    1. Check for correctness, edge cases, and style
    2. Suggest improvements or approve
    3. Report your verdict: APPROVED or NEEDS_FIXES""",
    llm_config={"model": "gpt-4o"},
)

# Create group chat
group_chat = GroupChat(
    agents=[planner, coder, reviewer],
    messages=[],
    max_round=15,
    speaker_selection_method="auto",  # LLM decides who speaks next
)

manager = GroupChatManager(
    groupchat=group_chat,
    llm_config={"model": "gpt-4o"},
)

# Start with the planner
planner.initiate_chat(
    manager,
    message="Build a REST API with FastAPI that manages a todo list with CRUD operations.",
)
```

## Code Template: Nested Chat for Quality Control

```python
"""Nested chat: main agent gets automatic code review after each response."""
from autogen import ConversableAgent

agent = ConversableAgent(
    name="developer",
    llm_config={"model": "gpt-4o"},
    code_execution_config={"work_dir": "dev_output"},
)

reviewer = ConversableAgent(
    name="reviewer",
    system_message="Review code for bugs, security issues, and performance. Be specific.",
    llm_config={"model": "gpt-4o"},
)

# Register nested chat — triggered after every agent response
agent.register_nested_chats(
    [
        {
            "recipient": reviewer,
            "message": lambda recipient, messages, sender, config: (
                f"Review this code:\n\n{messages[-1]['content']}"
            },
            "summary_method": "last_msg",
            "max_turns": 2,
        }
    ],
    trigger=lambda sender: sender.name == "user_proxy",
)

user_proxy = ConversableAgent(
    name="user_proxy",
    human_input_mode="NEVER",
    code_execution_config=False,
)

user_proxy.initiate_chat(agent, message="Write a function to detect SQL injection.")
```

## Code Template: Custom Speaker Selection

```python
"""Control which agent speaks next based on conversation state."""
from autogen import GroupChat

def custom_speaker_selection(last_speaker, group_chat):
    """Route to the right agent based on message content."""
    last_msg = group_chat.messages[-1]["content"].lower()

    if "needs_fixes" in last_msg:
        return group_chat.agent_by_name("coder")
    if "approved" in last_msg:
        return None  # End conversation
    if last_speaker.name == "planner":
        return group_chat.agent_by_name("coder")
    if last_speaker.name == "coder":
        return group_chat.agent_by_name("reviewer")
    return group_chat.agent_by_name("planner")

group_chat = GroupChat(
    agents=[planner, coder, reviewer],
    messages=[],
    speaker_selection_method=custom_speaker_selection,
)
```

## Patterns

### Pattern: Supervisor-Worker
```python
"""Supervisor delegates tasks, workers execute."""
supervisor = ConversableAgent(
    name="supervisor",
    system_message="You delegate tasks to workers and collect results. Never write code yourself.",
    llm_config={"model": "gpt-4o"},
)

workers = [
    ConversableAgent(name=f"worker_{i}", llm_config={"model": "gpt-4o"})
    for i in range(3)
]
```

### Pattern: Debate for Better Answers
```python
"""Two agents argue opposing sides, a judge picks the winner."""
debater_a = ConversableAgent(
    name="proponent",
    system_message="Argue FOR the proposal. Use evidence and examples.",
    llm_config={"model": "gpt-4o"},
)
debater_b = ConversableAgent(
    name="opponent",
    system_message="Argue AGAINST the proposal. Find flaws and counterexamples.",
    llm_config={"model": "gpt-4o"},
)
judge = ConversableAgent(
    name="judge",
    system_message="Listen to both sides. Declare a winner with reasoning.",
    llm_config={"model": "gpt-4o"},
)
```

### Pattern: Async Multi-Agent Pipeline
```python
"""Run agents asynchronously for better throughput."""
import asyncio
from autogen import ConversableAgent

async def run_pipeline(tasks: list[str]):
    agent = ConversableAgent(
        name="worker",
        llm_config={"model": "gpt-4o"},
    )
    results = await asyncio.gather(*[
        agent.a_initiate_chat(agent, message=task, max_turns=1)
        for task in tasks
    ])
    return [r.summary for r in results]
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Infinite loops | Agents keep replying without terminating | Always set `max_round` and `is_termination_msg` |
| Cost explosion | Group chats generate many LLM calls | Use cheaper models for simple agents |
| Speaker confusion | Wrong agent speaks next | Use explicit `speaker_selection_method` |
| Code execution risk | Agents run arbitrary code | Use Docker containers or sandboxed environments |
| Message overflow | Long conversations exceed context | Implement message truncation or summarization |
| Monologue | Agent talks to itself repeatedly | Set `allowed_or_disallowed_speaker_transitions` |

## Best Practices

1. **Set `max_round` on every group chat** — prevents runaway conversations
2. **Use `human_input_mode="NEVER"` for automated pipelines** — but `"ALWAYS"` for interactive use
3. **Containerize code execution** — always prefer `use_docker=True` in production
4. **Use termination messages** — standardize on `TERMINATE` across all agents
5. **Log conversations** — AutoGen has built-in logging; use it for debugging
6. **Test agent pairs independently** — before wiring them into a group chat

## Dependencies

```bash
pip install autogen-agentchat autogen-ext
# For code execution in Docker
pip install docker
# For structured output
pip install pydantic
```

## See Also

- [Agent Communication](../多Agent协作/agent-communication.md) — Communication protocols
- [Task Decomposition](../多Agent协作/task-decomposition.md) — Breaking tasks into subtasks
- [CrewAI Agents](./crewai-agents.md) — Alternative multi-agent framework
- [Agent Orchestration](../多Agent协作/agent-orchestration.md) — High-level orchestration patterns

---

## 中文版本

### 使用场景

- 多个 agent 辩论/头脑风暴
- 带执行反馈的代码生成
- 层级式团队工作流
- 需要生产级可观测性的多 agent 部署

> 单 agent tool-calling 使用 LangChain 或自定义方案即可，无需 AutoGen。

### 核心步骤

1. **双 Agent 对话** — 使用 `ConversableAgent` 创建 assistant 和 user_proxy，`initiate_chat` 启动对话
2. **群聊** — 使用 `GroupChat` + `GroupChatManager` 实现多 agent 群聊，LLM 自动选择发言者
3. **嵌套聊天** — 使用 `register_nested_chats` 在 agent 响应后自动触发代码审查
4. **自定义发言选择** — 实现 `custom_speaker_selection` 函数根据对话内容路由到正确的 agent
5. **异步执行** — 使用 `asyncio.gather` 并行运行多个 agent 对话

### 模板说明

- 双 Agent 对话 — assistant + user_proxy 的基础对话模板
- 群聊 — planner + coder + reviewer 三 agent 协作完成 FastAPI 开发
- 嵌套聊天 — developer agent 自动触发 code reviewer 的质量控制
- 辩论模式 — proponent + opponent + judge 的辩论求解模式

### 常见陷阱

1. **无限循环** — agent 持续回复不终止，始终设置 `max_round` 和 `is_termination_msg`
2. **成本爆炸** — 群聊产生大量 LLM 调用，简单 agent 使用便宜模型
3. **发言者混乱** — 错误的 agent 发言，使用显式 `speaker_selection_method`
4. **代码执行风险** — agent 运行任意代码，生产环境使用 Docker 容器沙箱
5. **消息溢出** — 长对话超出上下文窗口，实现消息截断或摘要
