# Task Decomposition and Delegation

> Break complex tasks into manageable subtasks and delegate them to the right agents — the key skill that makes multi-agent systems actually useful.

## When to Use

| Scenario | Use Task Decomposition? |
|----------|------------------------|
| Task is too complex for a single agent | Yes |
| Different subtasks need different expertise | Yes |
| Subtasks can run in parallel | Yes |
| Need clear accountability per subtask | Yes |
| Simple, single-step tasks | No — just use one agent |
| Task structure is unknown upfront | Yes, with dynamic decomposition |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Task Decomposition Flow                      │
│                                                          │
│  Complex Task                                            │
│      │                                                   │
│      ▼                                                   │
│  ┌──────────────────┐                                   │
│  │  Decomposer      │                                   │
│  │  (LLM + Strategy)│                                   │
│  └────────┬─────────┘                                   │
│           │                                              │
│           ▼                                              │
│  ┌──────────────────────────────────────┐               │
│  │  Task Graph / Dependency Tree        │               │
│  │                                      │               │
│  │  Task A ──► Task B ──► Task D       │               │
│  │  Task C ──┘           │              │               │
│  │                       ▼              │               │
│  │                    Task E            │               │
│  └──────────────────────────────────────┘               │
│           │                                              │
│           ▼                                              │
│  ┌──────────────────────────────────────┐               │
│  │  Executor                            │               │
│  │  ┌────────┐ ┌────────┐ ┌────────┐  │               │
│  │  │Agent 1 │ │Agent 2 │ │Agent 3 │  │               │
│  │  │(Code)  │ │(Search)│ │(Review)│  │               │
│  │  └────────┘ └────────┘ └────────┘  │               │
│  └──────────────────────────────────────┘               │
│           │                                              │
│           ▼                                              │
│  ┌──────────────────┐                                   │
│  │  Aggregator       │                                   │
│  │  (Combine Results)│                                   │
│  └──────────────────┘                                   │
└─────────────────────────────────────────────────────────┘
```

## Code Template: LLM-Powered Task Decomposer

```python
"""Decompose complex tasks into subtask graphs using LLM."""
from pydantic import BaseModel, Field
from openai import OpenAI

class Subtask(BaseModel):
    id: str
    description: str
    dependencies: list[str] = Field(default_factory=list, description="IDs of tasks that must complete first")
    assigned_role: str = Field(description="Best role for this task: coder, researcher, reviewer, writer")
    estimated_complexity: str = Field(description="low, medium, high")

class TaskPlan(BaseModel):
    goal: str
    subtasks: list[Subtask]
    execution_order: list[list[str]] = Field(
        description="Groups of task IDs that can run in parallel. Execute groups in order."
    )

class TaskDecomposer:
    def __init__(self):
        self.client = OpenAI()

    def decompose(self, task: str) -> TaskPlan:
        """Break a complex task into an execution plan."""
        response = self.client.beta.chat.completions.parse(
            model="gpt-4o",
            messages=[
                {
                    "role": "system",
                    "content": """You are a task decomposition expert. Break complex tasks into:
                    1. Small, independently executable subtasks
                    2. Clear dependency ordering
                    3. Role assignments for each subtask
                    4. Parallel execution groups

                    Be specific and actionable. Each subtask should take a single agent < 10 minutes.""",
                },
                {"role": "user", "content": f"Decompose this task:\n\n{task}"},
            ],
            response_format=TaskPlan,
        )
        return response.choices[0].message.parsed
```

## Code Template: Task Executor with Dependency Resolution

```python
"""Execute a task plan with proper dependency ordering."""
import asyncio
from dataclasses import dataclass
from enum import Enum

class TaskStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass
class TaskResult:
    task_id: str
    status: TaskStatus
    output: str
    agent_name: str

class TaskExecutor:
    def __init__(self, agents: dict[str, "Agent"]):
        self.agents = agents  # role -> agent mapping
        self.results: dict[str, TaskResult] = {}

    async def execute_plan(self, plan: TaskPlan) -> dict[str, TaskResult]:
        """Execute tasks respecting dependency order."""
        task_map = {t.id: t for t in plan.subtasks}

        for parallel_group in plan.execution_order:
            # Run all tasks in this group concurrently
            tasks = []
            for task_id in parallel_group:
                task = task_map[task_id]
                agent = self.agents.get(task.assigned_role)
                if not agent:
                    self.results[task_id] = TaskResult(
                        task_id=task_id, status=TaskStatus.FAILED,
                        output=f"No agent for role: {task.assigned_role}", agent_name="none",
                    )
                    continue
                tasks.append(self._execute_single(task, agent))

            await asyncio.gather(*tasks)

            # Check if any failed — abort downstream
            for task_id in parallel_group:
                if self.results.get(task_id, TaskResult("", TaskStatus.PENDING, "", "")).status == TaskStatus.FAILED:
                    return self.results

        return self.results

    async def _execute_single(self, task: Subtask, agent: "Agent"):
        """Execute a single subtask with dependency context."""
        # Gather dependency outputs
        dep_context = ""
        if task.dependencies:
            dep_outputs = []
            for dep_id in task.dependencies:
                if dep_id in self.results:
                    dep_outputs.append(f"[{dep_id}]: {self.results[dep_id].output}")
            dep_context = "\n\nPrevious task results:\n" + "\n".join(dep_outputs)

        prompt = f"Task: {task.description}{dep_context}"

        try:
            result = await agent.execute(prompt)
            self.results[task.id] = TaskResult(
                task_id=task.id, status=TaskStatus.COMPLETED,
                output=result, agent_name=agent.name,
            )
        except Exception as e:
            self.results[task.id] = TaskResult(
                task_id=task.id, status=TaskStatus.FAILED,
                output=str(e), agent_name=agent.name,
            )
```

## Code Template: Recursive Decomposition

```python
"""Dynamically decompose tasks as they're executed — handles unknown structure."""
class RecursiveDecomposer:
    def __init__(self, decomposer: TaskDecomposer, executor: TaskExecutor, max_depth: int = 3):
        self.decomposer = decomposer
        self.executor = executor
        self.max_depth = max_depth

    async def execute(self, task: str, depth: int = 0) -> str:
        """Decompose and execute, recursing for complex subtasks."""
        if depth >= self.max_depth:
            return await self._direct_execute(task)

        plan = self.decomposer.decompose(task)

        # Check if any subtask is still too complex
        for subtask in plan.subtasks:
            if subtask.estimated_complexity == "high":
                # Recursively decompose this subtask
                subtask.description = await self.execute(subtask.description, depth + 1)

        results = await self.executor.execute_plan(plan)
        return self._aggregate(results)

    async def _direct_execute(self, task: str) -> str:
        """Execute a task directly without further decomposition."""
        from openai import OpenAI
        client = OpenAI()
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": task}],
        )
        return response.choices[0].message.content

    def _aggregate(self, results: dict[str, TaskResult]) -> str:
        """Combine all subtask results into a final answer."""
        completed = [r for r in results.values() if r.status == TaskStatus.COMPLETED]
        return "\n\n".join(f"--- {r.task_id} ({r.agent_name}) ---\n{r.output}" for r in completed)
```

## Code Template: Self-Refining Decomposer

```python
"""Decomposer that learns from execution failures."""
class AdaptiveDecomposer:
    def __init__(self):
        self.client = OpenAI()
        self.failure_log: list[dict] = []

    def decompose_with_feedback(self, task: str) -> TaskPlan:
        """Incorporate past failures into decomposition strategy."""
        feedback = ""
        if self.failure_log:
            recent_failures = self.failure_log[-5:]
            feedback = "\n\nPast failures to avoid:\n" + "\n".join(
                f"- {f['task']}: {f['reason']}" for f in recent_failures
            )

        response = self.client.beta.chat.completions.parse(
            model="gpt-4o",
            messages=[
                {
                    "role": "system",
                    "content": f"""Decompose tasks with awareness of past failures.
                    Make subtasks more specific and less ambiguous.
                    Avoid decomposition patterns that led to failures.{feedback}""",
                },
                {"role": "user", "content": task},
            ],
            response_format=TaskPlan,
        )
        return response.choices[0].message.parsed

    def record_failure(self, task: str, reason: str):
        self.failure_log.append({"task": task, "reason": reason})
```

## Patterns

### Pattern: Map-Reduce Decomposition
```python
"""Split a task into parallel map tasks, then reduce."""
async def map_reduce(task: str, data: list[str], mapper_agent, reducer_agent):
    # Map: process each data item in parallel
    map_results = await asyncio.gather(*[
        mapper_agent.execute(f"Task: {task}\nData: {item}")
        for item in data
    ])

    # Reduce: combine all results
    combined = "\n\n".join(f"[Result {i}]: {r}" for i, r in enumerate(map_results))
    return await reducer_agent.execute(f"Combine these results:\n{combined}")
```

### Pattern: Hierarchical Decomposition
```python
"""Manager decomposes, workers execute, manager reviews."""
async def hierarchical_execute(goal: str, manager, workers):
    plan = manager.decompose(goal)
    results = {}

    for group in plan.execution_order:
        group_results = await asyncio.gather(*[
            workers[subtask.assigned_role].execute(subtask.description)
            for subtask in plan.subtasks if subtask.id in group
        ])
        for task_id, result in zip(group, group_results):
            results[task_id] = result

    # Manager reviews final output
    return await manager.review(results)
```

### Pattern: Iterative Refinement
```python
"""Execute, evaluate, and refine until quality threshold met."""
async def iterative_refine(task: str, max_iterations: int = 3):
    for i in range(max_iterations):
        result = await execute_task(task)
        quality = await evaluate_quality(result)
        if quality["score"] >= 0.8:
            return result
        task = f"Improve this: {result}\nIssues: {quality['feedback']}"
    return result
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Over-decomposition | Tasks split too fine, overhead dominates | Set minimum task complexity threshold |
| Circular dependencies | Task A depends on B depends on A | Detect cycles before execution |
| Vague subtask descriptions | Agent doesn't know what to do | Require specific, actionable descriptions |
| Missing dependencies | Task runs before its inputs are ready | Validate dependency graph before execution |
| Result quality | Subtask results don't combine well | Include aggregation instructions in plan |
| Cost explosion | Too many subtasks = too many LLM calls | Limit decomposition depth and breadth |

## Proven Patterns

1. **Be specific** — "Write a function that does X" is better than "Handle the backend"
2. **Validate the dependency graph** — check for cycles and missing references before execution
3. **Set complexity thresholds** — don't decompose tasks that are already simple enough
4. **Include aggregation instructions** — tell the decomposer how results should be combined
5. **Use structured output** — Pydantic models prevent hallucinated task IDs
6. **Log everything** — track which agent executed which task and the result

## See Also

- [Agent Communication](./agent-communication.md) — How agents exchange information
- [Agent Orchestration](./agent-orchestration.md) — High-level coordination
- [CrewAI Agents](../Agent框架/crewai-agents.md) — Built-in task pipeline
- [AutoGen Multi-Agent](../Agent框架/autogen-multiagent.md) — Group chat decomposition

---

## 中文版本

### 使用场景

- 任务过于复杂，单个 agent 无法完成
- 不同子任务需要不同专业能力
- 子任务可以并行执行
- 需要每个子任务有明确的责任人

> 简单单步任务直接使用一个 agent 即可。

### 核心步骤

1. **LLM 任务分解** — 使用 Pydantic 结构化输出定义 Subtask（id、description、dependencies、assigned_role）
2. **依赖解析执行** — 按 execution_order 分组，同组任务 asyncio.gather 并行执行，异组按序执行
3. **递归分解** — 对高复杂度子任务递归分解，直到达到最大深度后直接执行
4. **自适应分解** — 记录执行失败历史，下次分解时避免导致失败的模式
5. **Map-Reduce** — 将任务拆分为并行 map 任务处理数据，再 reduce 合并结果

### 模板说明

- TaskDecomposer — LLM 驱动的任务分解器，输出结构化 TaskPlan
- TaskExecutor — 带依赖解析的任务执行器，支持并行执行和失败中止
- RecursiveDecomposer — 递归分解 + 执行，处理未知结构的复杂任务
- AdaptiveDecomposer — 从失败中学习的自适应分解器

### 常见陷阱

1. **过度分解** — 任务拆得太细，开销占比过大，设置最小任务复杂度阈值
2. **循环依赖** — Task A 依赖 B 又依赖 A，执行前检测环路
3. **子任务描述模糊** — Agent 不知道做什么，要求具体可操作的描述
4. **依赖缺失** — 任务在输入未就绪时就执行，执行前验证依赖图
5. **成本爆炸** — 子任务过多 = LLM 调用过多，限制分解深度和广度
