# Agent-to-Agent Communication

> Design protocols and patterns for agents to exchange information, negotiate, and coordinate — the foundation of any multi-agent system.

## When to Use

| Scenario | Use Agent Communication? |
|----------|-------------------------|
| Two or more agents need to share information | Yes |
| Agents need to negotiate or reach consensus | Yes |
| Building a multi-agent pipeline | Yes |
| Agents run on different processes/machines | Yes |
| Single agent with multiple tools | No — use function calling |
| One-way delegation (no feedback loop) | Consider task decomposition |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Communication Patterns                       │
│                                                          │
│  1. Direct (Point-to-Point)                              │
│  ┌────────┐         ┌────────┐                          │
│  │Agent A │────────►│Agent B │                          │
│  └────────┘         └────────┘                          │
│                                                          │
│  2. Broadcast                                            │
│  ┌────────┐         ┌────────┐                          │
│  │Agent A │────┬────►│Agent B │                          │
│  └────────┘    │     └────────┘                          │
│                │     ┌────────┐                          │
│                └────►│Agent C │                          │
│                      └────────┘                          │
│                                                          │
│  3. Hub-and-Spoke                                        │
│  ┌────────┐    ┌─────────┐    ┌────────┐               │
│  │Agent A │───►│  Hub /   │◄───│Agent B │               │
│  └────────┘    │  Broker  │    └────────┘               │
│  ┌────────┐    │          │    ┌────────┐               │
│  │Agent C │───►│          │◄───│Agent D │               │
│  └────────┘    └─────────┘    └────────┘               │
│                                                          │
│  4. Pub/Sub                                              │
│  ┌────────┐    ┌──────────┐    ┌────────┐              │
│  │Publisher│───►│  Message  │◄───│Subscriber│            │
│  └────────┘    │  Bus      │    └────────┘              │
│  ┌────────┐    │          │    ┌────────┐              │
│  │Publisher│───►│          │◄───│Subscriber│            │
│  └────────┘    └──────────┘    └────────┘              │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Direct Agent Communication

```python
"""Agents communicating through a shared message bus."""
from dataclasses import dataclass, field
from typing import Callable
from datetime import datetime
import uuid

@dataclass
class Message:
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    sender: str = ""
    recipient: str = ""
    content: str = ""
    msg_type: str = "text"  # text, task, result, error
    metadata: dict = field(default_factory=dict)
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())

class MessageBus:
    """Central message bus for agent communication."""
    def __init__(self):
        self.agents: dict[str, "Agent"] = {}
        self.message_log: list[Message] = []
        self.handlers: dict[str, list[Callable]] = {}

    def register(self, agent_name: str, agent: "Agent"):
        self.agents[agent_name] = agent

    def send(self, message: Message):
        """Route a message to its recipient."""
        self.message_log.append(message)

        if message.recipient == "*":
            # Broadcast
            for name, agent in self.agents.items():
                if name != message.sender:
                    agent.receive(message)
        elif message.recipient in self.agents:
            self.agents[message.recipient].receive(message)

        # Notify handlers
        for handler in self.handlers.get("all", []):
            handler(message)

    def subscribe(self, event: str, handler: Callable):
        self.handlers.setdefault(event, []).append(handler)

class Agent:
    """Agent that communicates through the message bus."""
    def __init__(self, name: str, system_prompt: str, bus: MessageBus):
        self.name = name
        self.system_prompt = system_prompt
        self.bus = bus
        self.inbox: list[Message] = []
        self.bus.register(name, self)

    def send_to(self, recipient: str, content: str, msg_type: str = "text"):
        msg = Message(sender=self.name, recipient=recipient, content=content, msg_type=msg_type)
        self.bus.send(msg)

    def broadcast(self, content: str):
        self.send_to("*", content)

    def receive(self, message: Message):
        self.inbox.append(message)
        self.process_message(message)

    def process_message(self, message: Message):
        """Override this to handle incoming messages."""
        pass
```

## Code Template: Negotiation Protocol

```python
"""Agents negotiate to reach consensus on a plan."""
from openai import OpenAI

class NegotiatingAgent(Agent):
    def __init__(self, name: str, role: str, preferences: list[str], bus: MessageBus):
        super().__init__(name, f"You are {name}, role: {role}", bus)
        self.role = role
        self.preferences = preferences
        self.proposals: list[dict] = []
        self.client = OpenAI()

    def propose(self, topic: str) -> str:
        """Generate a proposal based on role and preferences."""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "system", "content": f"{self.system_prompt}\nPreferences: {self.preferences}"},
                {"role": "user", "content": f"Make a proposal for: {topic}"},
            ],
        )
        proposal = response.choices[0].message.content
        self.broadcast(f"PROPOSAL: {proposal}")
        return proposal

    def evaluate(self, proposal: str) -> dict:
        """Evaluate another agent's proposal."""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "system", "content": f"{self.system_prompt}\nPreferences: {self.preferences}"},
                {"role": "user", "content": f"Evaluate this proposal. Return ACCEPT or COUNTER with reasoning.\n\n{proposal}"},
            ],
        )
        result = response.choices[0].message.content
        return {"agent": self.name, "decision": result}

def run_negotiation(agents: list[NegotiatingAgent], topic: str, max_rounds: int = 3):
    """Run a multi-round negotiation."""
    # Round 1: Each agent proposes
    proposals = [agent.propose(topic) for agent in agents]

    for round_num in range(max_rounds):
        # Each agent evaluates all proposals
        evaluations = {}
        for agent in agents:
            for proposal in proposals:
                eval_result = agent.evaluate(proposal)
                evaluations.setdefault(agent.name, []).append(eval_result)

        # Check for consensus
        acceptances = sum(
            1 for evals in evaluations.values()
            for e in evals if "ACCEPT" in e["decision"]
        )
        total = len(evaluations) * len(proposals)

        if acceptances / total > 0.7:
            return {"status": "consensus", "round": round_num, "proposals": proposals}

        # Agents revise proposals based on feedback
        proposals = []
        for agent in agents:
            feedback = "\n".join(e["decision"] for e in evaluations.get(agent.name, []))
            revised = agent.client.chat.completions.create(
                model="gpt-4o",
                messages=[
                    {"role": "system", "content": agent.system_prompt},
                    {"role": "user", "content": f"Revise your proposal based on feedback:\n{feedback}"},
                ],
            )
            proposals.append(revised.choices[0].message.content)

    return {"status": "no_consensus", "proposals": proposals}
```

## Code Template: Event-Driven Communication

```python
"""Agents communicate through typed events."""
from dataclasses import dataclass
from typing import Any
import asyncio

@dataclass
class Event:
    type: str
    source: str
    data: Any
    timestamp: str = ""

class EventBus:
    def __init__(self):
        self.subscribers: dict[str, list[Callable]] = {}
        self.event_log: list[Event] = []

    def subscribe(self, event_type: str, handler: Callable):
        self.subscribers.setdefault(event_type, []).append(handler)

    async def publish(self, event: Event):
        self.event_log.append(event)
        for handler in self.subscribers.get(event_type := event.type, []):
            await handler(event)
        for handler in self.subscribers.get("*", []):  # Wildcard
            await handler(event)

# Usage
bus = EventBus()

async def code_writer_handler(event: Event):
    if event.type == "task.assigned":
        # Write code based on task
        result = await write_code(event.data["requirements"])
        await bus.publish(Event(type="code.written", source="coder", data={"code": result}))

async def reviewer_handler(event: Event):
    if event.type == "code.written":
        review = await review_code(event.data["code"])
        await bus.publish(Event(type="code.reviewed", source="reviewer", data={"review": review}))

bus.subscribe("task.assigned", code_writer_handler)
bus.subscribe("code.written", reviewer_handler)
```

## Patterns

### Pattern: Request-Response
```python
"""Agent A asks Agent B, waits for response."""
async def request_response(sender, recipient, query: str) -> str:
    response_channel = asyncio.Event()
    result = {}

    def handler(msg):
        result["response"] = msg.content
        response_channel.set()

    recipient.on_message(handler)
    sender.send_to(recipient.name, query)
    await response_channel.wait()
    return result["response"]
```

### Pattern: Pipeline Communication
```python
"""Agents pass results through a pipeline."""
class Pipeline:
    def __init__(self, agents: list[Agent]):
        self.agents = agents

    async def execute(self, input_data: str) -> str:
        current = input_data
        for agent in self.agents:
            current = await agent.process(current)
        return current
```

### Pattern: Blackboard (Shared State)
```python
"""Agents read/write to a shared blackboard."""
class Blackboard:
    def __init__(self):
        self.state: dict[str, Any] = {}
        self.lock = asyncio.Lock()

    async def write(self, key: str, value: Any, author: str):
        async with self.lock:
            self.state[key] = {"value": value, "author": author, "timestamp": datetime.now()}

    async def read(self, key: str) -> Any:
        return self.state.get(key)

    def snapshot(self) -> dict:
        return dict(self.state)
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Message storms | Agents triggering each other endlessly | Set max message depth per turn |
| Deadlocks | Two agents waiting for each other | Use timeouts on all requests |
| Lost messages | Async handlers drop messages | Use persistent queues (Redis, etc.) |
| Schema drift | Agents expect different message formats | Define strict message schemas |
| Unbounded inbox | Messages accumulate without processing | Set inbox size limits |

## Proven Patterns

1. **Define message schemas** — use Pydantic or typed dicts for all messages
2. **Log all communication** — essential for debugging multi-agent issues
3. **Use timeouts** — never wait forever for another agent's response
4. **Implement circuit breakers** — if an agent is unresponsive, route around it
5. **Keep messages small** — pass references (IDs, URIs), not large payloads
6. **Version your protocols** — message formats will evolve

## See Also

- [Task Decomposition](./task-decomposition.md) — Breaking tasks into subtasks
- [Agent Orchestration](./agent-orchestration.md) — High-level coordination
- [AutoGen Multi-Agent](../Agent框架/autogen-multiagent.md) — Framework implementation
- [Long-Term Memory](../记忆系统/long-term-memory.md) — Shared memory between agents

---

## 中文版本

### 使用场景

- 两个或多个 agent 需要共享信息
- Agent 需要协商或达成共识
- 构建多 agent 流水线
- Agent 运行在不同进程/机器上

> 单 agent 多工具使用 function calling；单向委派（无反馈）使用任务分解。

### 核心步骤

1. **消息总线** — 实现 MessageBus 中央消息路由，支持点对点和广播通信
2. **协商协议** — 多轮协商：各 agent 提出方案 → 评估 → 基于反馈修订 → 达成共识
3. **事件驱动通信** — 使用 EventBus 发布/订阅类型化事件，解耦 agent 之间的直接依赖
4. **请求-响应模式** — Agent A 向 Agent B 发送请求，等待响应，使用 asyncio.Event 同步
5. **黑板模式** — 共享状态空间，agent 读写黑板上的数据，使用锁保证并发安全

### 模板说明

- MessageBus — 注册 agent、路由消息、广播、事件订阅的完整消息总线
- NegotiatingAgent — 多轮协商协议，支持提案、评估、修订、共识检测
- EventBus — 异步事件驱动通信，支持类型化事件和通配符订阅
- 通信模式 — 请求-响应、流水线、黑板（共享状态）

### 常见陷阱

1. **消息风暴** — Agent 互相触发无止境，设置每轮最大消息深度
2. **死锁** — 两个 agent 互相等待对方响应，为所有请求设置超时
3. **消息丢失** — 异步处理器丢弃消息，使用持久队列（Redis 等）
4. **Schema 漂移** — Agent 期望不同消息格式，定义严格的消息 schema（Pydantic）
5. **无界收件箱** — 消息累积不处理，设置收件箱大小限制
