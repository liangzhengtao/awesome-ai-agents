# Orchestrating Multiple Agents

> Design and implement orchestration patterns for coordinating multiple agents — from simple sequential pipelines to dynamic, self-organizing agent swarms.

## When to Use

| Scenario | Use Agent Orchestration? |
|----------|-------------------------|
| Multiple agents working toward a shared goal | Yes |
| Complex workflows with conditional branching | Yes |
| Need monitoring, error handling, and recovery | Yes |
| Agents with different specializations | Yes |
| Single agent with tools | No — use a simple agent loop |
| Two agents talking back and forth | Use direct communication |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│               Orchestration Patterns                      │
│                                                          │
│  1. Sequential Pipeline                                  │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐            │
│  │Agent1│──►│Agent2│──►│Agent3│──►│Agent4│            │
│  └──────┘   └──────┘   └──────┘   └──────┘            │
│                                                          │
│  2. Parallel Fan-Out / Fan-In                            │
│  ┌──────┐   ┌──────┐                                   │
│  │Agent1│──┐│      │   ┌──────────┐                    │
│  └──────┘  │├─►│Aggregator│                            │
│  ┌──────┐  ││      │   └──────────┘                    │
│  │Agent2│──┤│      │                                    │
│  └──────┘  ││      │                                    │
│  ┌──────┐  ││      │                                    │
│  │Agent3│──┘│      │                                    │
│  └──────┘   └──────┘                                   │
│                                                          │
│  3. Conditional Router                                   │
│  ┌──────┐   ┌────────┐   ┌──────┐                      │
│  │Input │──►│ Router  │──►│Agent A│ (if code)           │
│  └──────┘   │ (LLM)  │──►│Agent B│ (if research)       │
│             └────────┘──►│Agent C│ (if writing)         │
│                          └──────┘                       │
│                                                          │
│  4. Orchestrator-Worker                                  │
│  ┌────────────────┐                                     │
│  │  Orchestrator   │──── Dynamically assigns ────┐     │
│  │  (Manager LLM)  │      tasks to workers        │     │
│  └────────────────┘                               │     │
│        ┌──────┬──────┬──────┐                     │     │
│        ▼      ▼      ▼      ▼                     ▼     │
│    ┌──────┐┌──────┐┌──────┐┌──────┐              │     │
│    │ W1   ││ W2   ││ W3   ││ W4   │◄─────────────┘     │
│    └──────┘└──────┘└──────┘└──────┘                     │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Sequential Pipeline Orchestrator

```python
"""Orchestrate agents in a sequential pipeline with logging."""
from dataclasses import dataclass, field
from typing import Callable
import time

@dataclass
class StepResult:
    step: str
    agent: str
    input_summary: str
    output: str
    duration: float
    success: bool

class PipelineOrchestrator:
    def __init__(self):
        self.steps: list[tuple[str, "Agent", Callable]] = []  # (name, agent, transform)
        self.results: list[StepResult] = []

    def add_step(self, name: str, agent: "Agent", transform: Callable = None):
        """Add a step to the pipeline. Transform converts previous output to next input."""
        self.steps.append((name, agent, transform or (lambda x: x)))
        return self

    async def execute(self, initial_input: str) -> str:
        """Execute the pipeline from start to finish."""
        current_input = initial_input

        for step_name, agent, transform in self.steps:
            start = time.time()
            try:
                output = await agent.execute(current_input)
                self.results.append(StepResult(
                    step=step_name, agent=agent.name,
                    input_summary=current_input[:100],
                    output=output, duration=time.time() - start,
                    success=True,
                ))
                current_input = transform(output)
            except Exception as e:
                self.results.append(StepResult(
                    step=step_name, agent=agent.name,
                    input_summary=current_input[:100],
                    output=str(e), duration=time.time() - start,
                    success=False,
                ))
                raise

        return current_input

    def report(self) -> str:
        """Generate an execution report."""
        lines = ["Pipeline Execution Report:"]
        for r in self.results:
            status = "✓" if r.success else "✗"
            lines.append(f"  {status} [{r.step}] {r.agent} ({r.duration:.1f}s)")
        return "\n".join(lines)
```

## Code Template: Fan-Out / Fan-In Orchestrator

```python
"""Execute agents in parallel, then aggregate results."""
import asyncio
from typing import Any

class FanOutFanInOrchestrator:
    def __init__(self):
        self.workers: list[tuple[str, "Agent"]] = []
        self.aggregator: "Agent" = None

    def add_worker(self, name: str, agent: "Agent"):
        self.workers.append((name, agent))
        return self

    def set_aggregator(self, agent: "Agent"):
        self.aggregator = agent
        return self

    async def execute(self, task: str) -> str:
        """Fan out to all workers, then fan in to aggregator."""
        # Fan-out: execute all workers in parallel
        async def run_worker(name: str, agent: "Agent"):
            try:
                result = await agent.execute(task)
                return {"name": name, "status": "success", "output": result}
            except Exception as e:
                return {"name": name, "status": "error", "output": str(e)}

        results = await asyncio.gather(*[
            run_worker(name, agent) for name, agent in self.workers
        ])

        # Fan-in: aggregate results
        aggregated_input = "\n\n".join(
            f"--- {r['name']} ({r['status']}) ---\n{r['output']}"
            for r in results
        )

        if self.aggregator:
            return await self.aggregator.execute(
                f"Aggregate these parallel results for task: {task}\n\n{aggregated_input}"
            )
        return aggregated_input
```

## Code Template: Dynamic Router Orchestrator

```python
"""Route tasks to specialized agents based on content analysis."""
from pydantic import BaseModel
from openai import OpenAI

class RoutingDecision(BaseModel):
    reasoning: str
    selected_agent: str
    confidence: float

class RouterOrchestrator:
    def __init__(self):
        self.agents: dict[str, "Agent"] = {}
        self.client = OpenAI()

    def register(self, name: str, agent: "Agent", capabilities: str):
        """Register an agent with its capabilities description."""
        self.agents[name] = {"agent": agent, "capabilities": capabilities}

    async def execute(self, task: str) -> str:
        """Route task to the most appropriate agent."""
        # Build routing prompt
        agent_descriptions = "\n".join(
            f"- {name}: {info['capabilities']}"
            for name, info in self.agents.items()
        )

        routing = self.client.beta.chat.completions.parse(
            model="gpt-4o-mini",  # Use cheap model for routing
            messages=[
                {
                    "role": "system",
                    "content": f"Route this task to the best agent.\n\nAvailable agents:\n{agent_descriptions}",
                },
                {"role": "user", "content": task},
            ],
            response_format=RoutingDecision,
        )

        decision = routing.choices[0].message.parsed
        selected = self.agents.get(decision.selected_agent)

        if not selected:
            # Fallback to first agent
            selected = list(self.agents.values())[0]

        return await selected["agent"].execute(task)
```

## Code Template: Orchestrator-Worker with Dynamic Task Assignment

```python
"""Manager agent that dynamically creates and assigns tasks."""
from openai import OpenAI

class OrchestratorWorker:
    def __init__(self, workers: dict[str, "Agent"]):
        self.workers = workers
        self.client = OpenAI()
        self.execution_log: list[dict] = []

    async def execute(self, goal: str) -> str:
        """Let the orchestrator LLM drive the workflow."""
        messages = [
            {
                "role": "system",
                "content": f"""You are an orchestrator managing these workers:
{chr(10).join(f'- {name}: {agent.description}' for name, agent in self.workers.items())}

To assign a task, respond with JSON:
{{"action": "assign", "worker": "name", "task": "description"}}

To finish, respond with:
{{"action": "complete", "summary": "final answer"}}

Think step by step. Assign tasks based on worker capabilities.""",
            },
            {"role": "user", "content": f"Goal: {goal}"},
        ]

        for turn in range(15):  # Max 15 turns
            response = self.client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                response_format={"type": "json_object"},
            )

            import json
            decision = json.loads(response.choices[0].message.content)
            messages.append({"role": "assistant", "content": json.dumps(decision)})

            if decision["action"] == "complete":
                return decision["summary"]

            if decision["action"] == "assign":
                worker_name = decision["worker"]
                task_desc = decision["task"]

                if worker_name in self.workers:
                    result = await self.workers[worker_name].execute(task_desc)
                    self.execution_log.append({
                        "worker": worker_name, "task": task_desc, "result": result[:200],
                    })
                    messages.append({
                        "role": "user",
                        "content": json.dumps({"worker": worker_name, "result": result}),
                    })
                else:
                    messages.append({
                        "role": "user",
                        "content": json.dumps({"error": f"Unknown worker: {worker_name}"}),
                    })

        return "Orchestrator reached max turns."
```

## Code Template: State Machine Orchestrator

```python
"""Orchestrate agents using a finite state machine."""
from enum import Enum
from typing import Callable

class State(Enum):
    ANALYZE = "analyze"
    PLAN = "plan"
    EXECUTE = "execute"
    REVIEW = "review"
    REVISE = "revise"
    COMPLETE = "complete"
    ERROR = "error"

class StateMachineOrchestrator:
    def __init__(self):
        self.agents: dict[State, "Agent"] = {}
        self.transitions: dict[State, dict[str, State]] = {}
        self.state_data: dict = {}

    def register(self, state: State, agent: "Agent", transitions: dict[str, State]):
        self.agents[state] = agent
        self.transitions[state] = transitions

    async def execute(self, initial_input: str) -> str:
        current_state = State.ANALYZE
        self.state_data["input"] = initial_input

        while current_state != State.COMPLETE and current_state != State.ERROR:
            agent = self.agents.get(current_state)
            if not agent:
                current_state = State.ERROR
                break

            # Execute current state's agent
            result = await agent.execute(str(self.state_data))
            self.state_data[current_state.value] = result

            # Determine next state
            eval_response = self._evaluate_transition(current_state, result)
            next_state_name = self.transitions.get(current_state, {}).get(eval_response, State.ERROR)
            current_state = next_state_name

        if current_state == State.COMPLETE:
            return self.state_data.get("review", self.state_data.get("execute", "No result"))
        return f"Error: reached {current_state.value} state"

    def _evaluate_transition(self, current_state: State, result: str) -> str:
        """Use LLM to decide which transition to take."""
        transitions = self.transitions.get(current_state, {})
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "system",
                "content": f"Based on this result, choose a transition: {list(transitions.keys())}\n\nResult: {result[:500]}",
            }],
        )
        # Parse the chosen transition
        for key in transitions:
            if key.lower() in response.choices[0].message.content.lower():
                return key
        return list(transitions.keys())[0]  # Default to first
```

## Patterns

### Pattern: Supervised Execution
```python
"""Supervisor monitors and intervenes when agents struggle."""
async def supervised_execute(task: str, worker: Agent, supervisor: Agent):
    result = await worker.execute(task)
    assessment = await supervisor.execute(f"Is this correct?\nTask: {task}\nResult: {result}")
    if "incorrect" in assessment.lower():
        # Retry with supervisor feedback
        result = await worker.execute(f"Redo with feedback: {assessment}")
    return result
```

### Pattern: Voting / Consensus
```python
"""Multiple agents vote on the best answer."""
async def consensus_execute(task: str, agents: list[Agent], threshold: float = 0.6):
    results = await asyncio.gather(*[agent.execute(task) for agent in agents])
    # Use LLM to find the majority answer
    judge = OpenAI()
    response = judge.chat.completions.create(
        model="gpt-4o",
        messages=[{
            "role": "user",
            "content": f"These agents answered the same question. Find the consensus:\n\n" +
                       "\n".join(f"Agent {i}: {r}" for i, r in enumerate(results)),
        }],
    )
    return response.choices[0].message.content
```

### Pattern: Checkpointing and Recovery
```python
"""Save progress so long-running orchestrations can resume."""
import json

class CheckpointOrchestrator:
    def __init__(self, checkpoint_path: str = "checkpoint.json"):
        self.checkpoint_path = checkpoint_path
        self.state = self._load_checkpoint()

    def save_checkpoint(self, step: int, data: dict):
        with open(self.checkpoint_path, "w") as f:
            json.dump({"step": step, "data": data}, f)

    def _load_checkpoint(self) -> dict:
        try:
            with open(self.checkpoint_path) as f:
                return json.load(f)
        except FileNotFoundError:
            return {"step": 0, "data": {}}
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| No error recovery | One agent failure kills everything | Implement try/except per agent with fallbacks |
| Unbounded orchestration | Orchestrator never terminates | Set max turns and timeouts |
| Single point of failure | Orchestrator LLM goes down | Add fallback orchestrator |
| Cost explosion | Dynamic routing generates many LLM calls | Use cheaper models for routing decisions |
| State explosion | Too many states in state machine | Keep state machine simple; decompose if needed |
| No observability | Can't debug what agents are doing | Log every state transition and agent output |

## Proven Patterns

1. **Start with sequential** — most tasks don't need complex orchestration
2. **Add parallelism where it helps** — only for truly independent subtasks
3. **Use cheap models for routing** — `gpt-4o-mini` for decisions, `gpt-4o` for work
4. **Implement checkpointing** — long orchestrations should be resumable
5. **Log everything** — every state transition, agent call, and result
6. **Set hard limits** — max turns, max agents, max cost per orchestration

## See Also

- [Agent Communication](./agent-communication.md) — How agents exchange messages
- [Task Decomposition](./task-decomposition.md) — Breaking tasks into subtasks
- [CrewAI Agents](../Agent框架/crewai-agents.md) — Built-in orchestration framework
- [AutoGen Multi-Agent](../Agent框架/autogen-multiagent.md) — Group chat orchestration

---

## 中文版本

### 使用场景

- 多个 agent 协作完成共同目标
- 复杂工作流带条件分支
- 需要监控、错误处理和恢复
- 不同专业能力的 agent 协同工作

> 单 agent 多工具使用简单 agent 循环；两个 agent 来回对话使用直接通信。

### 核心步骤

1. **顺序流水线** — Agent1 → Agent2 → Agent3 依次执行，带日志和错误处理
2. **扇出/扇入** — 多个 agent 并行执行同一任务，aggregator 合并结果
3. **动态路由** — LLM 分析任务内容，将任务路由到最合适的 agent（用便宜模型做路由决策）
4. **编排器-Worker** — 管理器 LLM 动态创建和分配任务给 worker，支持多轮协调
5. **状态机编排** — 使用有限状态机（Analyze → Plan → Execute → Review → Complete）驱动 agent 流转

### 模板说明

- PipelineOrchestrator — 顺序流水线，支持步骤间数据转换和执行报告
- FanOutFanInOrchestrator — 并行扇出 + 聚合器扇入
- RouterOrchestrator — LLM 驱动的动态任务路由，使用 Pydantic 结构化输出
- StateMachineOrchestrator — 有限状态机驱动的 agent 编排

### 常见陷阱

1. **无错误恢复** — 一个 agent 失败导致全部终止，为每个 agent 实现 try/except + 降级
2. **无界编排** — 编排器永不终止，设置最大轮次和超时
3. **单点故障** — 编排器 LLM 宕机，添加备用编排器
4. **成本爆炸** — 动态路由产生大量 LLM 调用，路由决策用便宜模型
5. **无可观测性** — 无法调试 agent 行为，记录每个状态转换和 agent 输出
