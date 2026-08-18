# Function Calling and Tool Use

> Master the function calling protocol that powers all modern AI agents — from OpenAI and Anthropic to Gemini and open-source models.

## When to Use

| Scenario | Use Function Calling? |
|----------|----------------------|
| Agent needs to interact with external systems | Yes |
| Structured data extraction from text | Yes |
| Database queries from natural language | Yes |
| Real-time API integrations | Yes |
| Simple text generation | No — just use chat completions |
| Complex multi-step reasoning | Yes, combined with a ReAct loop |

## Architecture

```
┌───────────────────────────────────────────────────────┐
│                    LLM Provider API                    │
│                                                        │
│  Input:                                                │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ Messages  │  │ Tool Schemas │  │ tool_choice    │  │
│  └──────────┘  └──────────────┘  └────────────────┘  │
│                                                        │
│  Output:                                               │
│  ┌──────────────────────────────────────────────┐     │
│  │ Response                                      │     │
│  │  ├─ content: "Here's what I found..."        │     │
│  │  └─ tool_calls: [                            │     │
│  │       { id: "call_abc",                      │     │
│  │         function: {                          │     │
│  │           name: "get_weather",               │     │
│  │           arguments: '{"city":"Tokyo"}' } }] │     │
│  └──────────────────────────────────────────────┘     │
│                                                        │
│  Tool Execution (your code):                           │
│  ┌──────────────────────────────────────────────┐     │
│  │ 1. Parse tool_calls                           │     │
│  │ 2. Execute functions                          │     │
│  │ 3. Send results back as tool messages         │     │
│  └──────────────────────────────────────────────┘     │
└───────────────────────────────────────────────────────┘
```

## Code Template: OpenAI Function Calling

```python
"""Complete function calling example with OpenAI."""
import json
from openai import OpenAI

client = OpenAI()

# Step 1: Define tools using JSON Schema
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name, e.g. 'Tokyo'",
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit",
                    },
                },
                "required": ["city"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "search_products",
            "description": "Search for products in the catalog",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"},
                    "max_price": {"type": "number", "description": "Max price filter"},
                    "category": {
                        "type": "string",
                        "enum": ["electronics", "clothing", "food"],
                    },
                },
                "required": ["query"],
            },
        },
    },
]

# Step 2: Implement the actual functions
def get_weather(city: str, unit: str = "celsius") -> dict:
    # Replace with real API call
    return {"city": city, "temp": 22, "unit": unit, "condition": "sunny"}

def search_products(query: str, max_price: float = None, category: str = None):
    return [{"name": f"Product matching '{query}'", "price": 29.99}]

# Step 3: Map function names to implementations
FUNCTION_MAP = {
    "get_weather": get_weather,
    "search_products": search_products,
}

# Step 4: Agent loop
messages = [
    {"role": "system", "content": "You are a helpful shopping assistant."},
    {"role": "user", "content": "What's the weather in Tokyo? And find me electronics under $50."},
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice="auto",
)

msg = response.choices[0].message
messages.append(msg)

# Step 5: Execute tool calls and send results back
if msg.tool_calls:
    for tool_call in msg.tool_calls:
        fn_name = tool_call.function.name
        fn_args = json.loads(tool_call.function.arguments)
        result = FUNCTION_MAP[fn_name](**fn_args)

        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps(result),
        })

    # Get final response with tool results
    final = client.chat.completions.create(model="gpt-4o", messages=messages)
    print(final.choices[0].message.content)
```

## Code Template: Anthropic Tool Use

```python
"""Function calling with Anthropic Claude."""
import anthropic

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_stock_price",
        "description": "Get the current stock price for a ticker symbol",
        "input_schema": {
            "type": "object",
            "properties": {
                "ticker": {"type": "string", "description": "Stock ticker, e.g. AAPL"},
            },
            "required": ["ticker"],
        },
    },
]

def get_stock_price(ticker: str) -> dict:
    return {"ticker": ticker, "price": 189.50, "currency": "USD"}

# Initial request
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's Apple's stock price?"}],
)

# Process tool use blocks
for block in response.content:
    if block.type == "tool_use":
        result = get_stock_price(**block.input)
        # Send tool result back
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "What's Apple's stock price?"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result),
                    }],
                },
            ],
        )

print(response.content[0].text)
```

## Code Template: Parallel Tool Calls

```python
"""Handle multiple tool calls in a single LLM response."""
messages = [{"role": "user", "content": "Weather in Tokyo and London?"}]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
)

msg = response.choices[0].message
messages.append(msg)

# Execute ALL tool calls (parallel in a single turn)
if msg.tool_calls:
    for tc in msg.tool_calls:
        fn = json.loads(tc.function.arguments)
        # In production, use asyncio.gather or ThreadPoolExecutor
        result = get_weather(**fn)
        messages.append({
            "role": "tool",
            "tool_call_id": tc.id,
            "content": json.dumps(result),
        })

final = client.chat.completions.create(model="gpt-4o", messages=messages)
```

## Code Template: Forced Tool Use

```python
"""Force the LLM to always call a specific tool first."""
# Always call a tool (never answer directly)
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice={"type": "function", "function": {"name": "get_weather"}},
)

# Require any tool call
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice="required",
)
```

## Patterns

### Pattern: Dynamic Tool Registration
```python
"""Register tools at runtime based on user context."""
def get_tools_for_user(user_role: str) -> list[dict]:
    all_tools = {
        "admin": [delete_user_tool, modify_settings_tool],
        "viewer": [search_tool, read_tool],
    }
    return all_tools.get(user_role, [search_tool])
```

### Pattern: Tool Result Validation
```python
"""Validate tool results before returning to LLM."""
def safe_tool_call(fn, args: dict, schema: dict) -> str:
    # Validate arguments against schema
    try:
        validate(args, schema)
        result = fn(**args)
        # Validate result is serializable
        json.dumps(result)
        return json.dumps(result)
    except Exception as e:
        return json.dumps({"error": str(e)})
```

### Pattern: Tool Caching
```python
"""Cache tool results to avoid redundant calls."""
from functools import lru_cache

@lru_cache(maxsize=128)
def cached_search(query: str) -> str:
    """Same query won't hit the API twice."""
    return json.dumps(search_api(query))
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Schema mismatch | LLM generates invalid arguments | Use strict JSON Schema; validate with Pydantic |
| Tool name typo | LLM hallucinates function names | Use `tool_choice` to force valid names |
| Missing `tool` message | Forget to send tool results back | Always append tool result before next call |
| Timeout on slow tools | API calls hang the agent | Set timeouts on all external calls |
| Argument injection | User crafts input to exploit tools | Sanitize and validate all tool arguments |
| Provider differences | OpenAI vs Anthropic schemas differ | Abstract behind a unified interface |

## Proven Patterns

1. **Write clear tool descriptions** — the LLM relies on them to decide when to call tools
2. **Use strict JSON Schema** — define `required` fields, `enum` constraints, and type annotations
3. **Validate arguments with Pydantic** — catch malformed inputs before execution
4. **Return structured results** — JSON is better than free-form text for tool results
5. **Handle errors gracefully** — return error messages, don't crash the agent loop
6. **Log every tool call** — essential for debugging and cost tracking

## Provider Comparison

| Feature | OpenAI | Anthropic | Gemini |
|---------|--------|-----------|--------|
| Parallel tool calls | Yes | Yes | Yes |
| Forced tool use | Yes (`tool_choice`) | Yes (`tool_choice`) | Yes |
| Streaming tool calls | Yes | Yes | Yes |
| Schema format | JSON Schema | JSON Schema | OpenAPI-like |
| Max tools | 128 | 64 | 64 |

## See Also

- [MCP Integration](./mcp-integration.md) — Standardized tool protocol
- [API Orchestration](./api-orchestration.md) — Combining multiple APIs
- [Custom Agents](../Agent框架/custom-agents.md) — Using tools in agent loops
- [RAG Memory](../记忆系统/rag-memory.md) — Retrieval as a tool

---

## 中文版本

### 使用场景

- Agent 需要与外部系统交互
- 从文本中提取结构化数据
- 用自然语言查询数据库
- 实时 API 集成

> 简单文本生成不需要 function calling，直接使用 chat completions。

### 核心步骤

1. **定义工具 Schema** — 使用 JSON Schema 描述工具的参数、类型、必填项和描述
2. **实现工具函数** — 编写实际的函数实现，映射函数名到函数
3. **Agent 循环** — 发送消息 + tools 给 LLM，解析 tool_calls，执行函数，将结果作为 tool message 返回
4. **并行工具调用** — 处理单个 LLM 响应中的多个 tool_calls
5. **强制工具使用** — 使用 `tool_choice` 参数强制 LLM 调用特定工具

### 模板说明

- OpenAI Function Calling — 完整的定义工具 → 实现 → 循环 → 返回结果流程
- Anthropic Tool Use — Claude 的 tool_use 协议和 tool_result 返回格式
- 并行工具调用 — 单个响应中处理多个 tool_calls
- 强制工具使用 — `tool_choice` 的三种用法（auto、required、指定工具名）

### 常见陷阱

1. **Schema 不匹配** — LLM 生成无效参数，使用严格 JSON Schema + Pydantic 验证
2. **工具名拼写错误** — LLM 幻觉出不存在的函数名，使用 `tool_choice` 强制有效名称
3. **遗漏 tool message** — 忘记将工具结果发回 LLM，导致对话上下文断裂
4. **慢工具超时** — API 调用挂起阻塞 agent，为所有外部调用设置超时
5. **Provider 差异** — OpenAI vs Anthropic schema 格式不同，抽象为统一接口
