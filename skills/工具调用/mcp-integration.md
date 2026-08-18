# MCP Server Integration for Agents

> Integrate Model Context Protocol (MCP) servers into your AI agents — the universal standard for connecting LLMs to external tools, data, and services.

## When to Use

| Scenario | Use MCP? |
|----------|---------|
| Want a standardized tool interface across providers | Yes |
| Building reusable tool servers for multiple agents | Yes |
| Integrating with Claude, Cursor, Kimi Code, etc. | Yes |
| Need tools discoverable via a protocol | Yes |
| Simple one-off tool calls | Function calling is simpler |
| High-frequency, low-latency tool calls | Direct function calling is faster |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    MCP Architecture                       │
│                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐           │
│  │  Agent   │    │  Agent   │    │  Agent   │           │
│  │ (Host)   │    │ (Host)   │    │ (Host)   │           │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘           │
│       │               │               │                  │
│       └───────────────┼───────────────┘                  │
│                       │ MCP Protocol                     │
│           ┌───────────┼───────────┐                      │
│           ▼           ▼           ▼                      │
│     ┌──────────┐ ┌──────────┐ ┌──────────┐             │
│     │ MCP      │ │ MCP      │ │ MCP      │             │
│     │ Server 1 │ │ Server 2 │ │ Server 3 │             │
│     │ (Files)  │ │ (DB)     │ │ (APIs)   │             │
│     └──────────┘ └──────────┘ └──────────┘             │
│                                                          │
│  Protocol Features:                                      │
│  • Tools (function calling)                              │
│  • Resources (data access)                               │
│  • Prompts (prompt templates)                            │
│  • Sampling (LLM delegation)                             │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Building an MCP Server

```python
"""A complete MCP server exposing tools and resources."""
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent, Resource
import json

server = Server("my-tools-server")

# Define available tools
@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="query_database",
            description="Execute a read-only SQL query against the database",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "SQL SELECT query",
                    },
                    "database": {
                        "type": "string",
                        "description": "Database name",
                        "default": "main",
                    },
                },
                "required": ["query"],
            },
        ),
        Tool(
            name="send_email",
            description="Send an email notification",
            inputSchema={
                "type": "object",
                "properties": {
                    "to": {"type": "string", "description": "Recipient email"},
                    "subject": {"type": "string", "description": "Email subject"},
                    "body": {"type": "string", "description": "Email body (markdown)"},
                },
                "required": ["to", "subject", "body"],
            },
        ),
    ]

# Implement tool handlers
@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "query_database":
        result = await execute_query(arguments["query"], arguments.get("database", "main"))
        return [TextContent(type="text", text=json.dumps(result))]

    elif name == "send_email":
        await send_email(
            to=arguments["to"],
            subject=arguments["subject"],
            body=arguments["body"],
        )
        return [TextContent(type="text", text="Email sent successfully")]

    raise ValueError(f"Unknown tool: {name}")

# Define available resources (data sources)
@server.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="db://main/schema",
            name="Database Schema",
            description="Current database table schema",
            mimeType="application/json",
        ),
    ]

@server.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "db://main/schema":
        schema = await get_database_schema()
        return json.dumps(schema)
    raise ValueError(f"Unknown resource: {uri}")

# Run the server
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream, server.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## Code Template: Connecting to an MCP Server (Client)

```python
"""Connect to an MCP server and use its tools."""
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from openai import OpenAI
import json

async def run_agent_with_mcp():
    # Connect to MCP server
    server_params = StdioServerParameters(
        command="python",
        args=["my_mcp_server.py"],
    )

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Discover available tools
            tools_response = await session.list_tools()

            # Convert MCP tools to OpenAI format
            openai_tools = []
            for tool in tools_response.tools:
                openai_tools.append({
                    "type": "function",
                    "function": {
                        "name": tool.name,
                        "description": tool.description,
                        "parameters": tool.inputSchema,
                    },
                })

            # Run agent with MCP tools
            client = OpenAI()
            messages = [
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": "Query all users from the database"},
            ]

            response = client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                tools=openai_tools,
            )

            msg = response.choices[0].message
            if msg.tool_calls:
                for tc in msg.tool_calls:
                    # Call MCP tool
                    result = await session.call_tool(
                        tc.function.name,
                        json.loads(tc.function.arguments),
                    )
                    messages.append({
                        "role": "tool",
                        "tool_call_id": tc.id,
                        "content": result.content[0].text,
                    })

                final = client.chat.completions.create(
                    model="gpt-4o", messages=messages
                )
                print(final.choices[0].message.content)
```

## Code Template: MCP Server with Resources and Prompts

```python
"""MCP server exposing resources and reusable prompts."""
from mcp.server import Server
from mcp.types import Prompt, PromptArgument, Resource

server = Server("knowledge-base")

# Prompt templates
@server.list_prompts()
async def list_prompts() -> list[Prompt]:
    return [
        Prompt(
            name="analyze_code",
            description="Analyze a code file for issues",
            arguments=[
                PromptArgument(
                    name="language",
                    description="Programming language",
                    required=True,
                ),
                PromptArgument(
                    name="focus",
                    description="Focus area: security, performance, style",
                    required=False,
                ),
            ],
        ),
    ]

@server.get_prompt()
async def get_prompt(name: str, arguments: dict) -> list:
    if name == "analyze_code":
        lang = arguments["language"]
        focus = arguments.get("focus", "general quality")
        return [{
            "role": "user",
            "content": f"Analyze the following {lang} code focusing on {focus}. "
                       f"Provide specific, actionable feedback with line references.",
        }]

# Dynamic resources
@server.list_resources()
async def list_resources():
    # Could list files, database tables, API endpoints, etc.
    return [
        Resource(uri="file:///project/README.md", name="Project README", mimeType="text/markdown"),
        Resource(uri="db://users/count", name="User Count", mimeType="application/json"),
    ]
```

## Code Template: Multi-Server Agent

```python
"""Agent connected to multiple MCP servers simultaneously."""
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

SERVERS = {
    "database": StdioServerParameters(command="python", args=["db_server.py"]),
    "email": StdioServerParameters(command="python", args=["email_server.py"]),
    "filesystem": StdioServerParameters(command="npx", args=["-y", "@modelcontextprotocol/server-filesystem", "/data"]),
}

async def run_multi_server_agent(query: str):
    all_tools = []

    # Connect to all servers and collect tools
    contexts = []
    for name, params in SERVERS.items():
        ctx = stdio_client(params)
        read, write = await ctx.__aenter__()
        session = ClientSession(read, write)
        await session.__aenter__()
        await session.initialize()

        tools = await session.list_tools()
        for tool in tools.tools:
            all_tools.append({
                "type": "function",
                "function": {
                    "name": f"{name}__{tool.name}",  # Namespace tools
                    "description": f"[{name}] {tool.description}",
                    "parameters": tool.inputSchema,
                },
            })
        contexts.append((name, session, ctx))

    # Agent loop with multi-server tools
    client = OpenAI()
    messages = [
        {"role": "system", "content": "You have access to database, email, and filesystem tools."},
        {"role": "user", "content": query},
    ]

    for turn in range(10):
        response = client.chat.completions.create(
            model="gpt-4o", messages=messages, tools=all_tools
        )
        msg = response.choices[0].message
        messages.append(msg)

        if not msg.tool_calls:
            return msg.content

        for tc in msg.tool_calls:
            server_name, tool_name = tc.function.name.split("__", 1)
            session = next(s for n, s, _ in contexts if n == server_name)
            result = await session.call_tool(tool_name, json.loads(tc.function.arguments))
            messages.append({
                "role": "tool", "tool_call_id": tc.id,
                "content": result.content[0].text,
            })
```

## Patterns

### Pattern: Tool Namespacing
```python
# Prefix tools with server name to avoid collisions
f"{server_name}__{tool_name}"
```

### Pattern: Health Check
```python
@server.list_tools()
async def list_tools():
    """Also serves as a health check — if this fails, server is down."""
    return [...]
```

### Pattern: Tool Discovery Agent
```python
"""Agent that explores available MCP tools before acting."""
async def discover_and_act(session, query):
    tools = await session.list_tools()
    tool_summary = "\n".join(f"- {t.name}: {t.description}" for t in tools.tools)

    plan = client.chat.completions.create(
        model="gpt-4o",
        messages=[{
            "role": "user",
            "content": f"Available tools:\n{tool_summary}\n\nPlan how to answer: {query}",
        }],
    )
    # Then execute the plan...
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Server crashes | MCP server process dies | Add health checks and auto-restart |
| Slow transport | stdio is fast, HTTP adds latency | Use stdio for local, SSE for remote |
| Tool name collisions | Multiple servers with same tool names | Namespace with server prefix |
| Schema drift | Server updates break clients | Pin server versions; validate schemas |
| Security | MCP server exposes sensitive operations | Authenticate and authorize tool calls |
| Large resources | Reading huge files via resource | Implement pagination and streaming |

## Proven Patterns

1. **Namespace tool names** — prefix with server name when connecting to multiple servers
2. **Use stdio for local servers** — faster and more reliable than HTTP/SSE
3. **Implement graceful error handling** — return error messages, don't crash
4. **Version your tool schemas** — breaking changes should be versioned
5. **Test tools independently** — write unit tests for each tool handler
6. **Use resources for read-only data** — tools for actions, resources for data

## See Also

- [Function Calling](./function-calling.md) — The underlying mechanism
- [API Orchestration](./api-orchestration.md) — Combining multiple APIs
- [Custom Agents](../Agent框架/custom-agents.md) — Building agent loops
- [Long-Term Memory](../记忆系统/long-term-memory.md) — MCP as a memory source

---

## 中文版本

### 使用场景

- 需要跨 provider 的标准化工具接口
- 为多个 agent 构建可复用的工具服务器
- 集成 Claude、Cursor、Kimi Code 等支持 MCP 的客户端
- 需要通过协议发现工具的能力

> 简单一次性工具调用使用 function calling 更简单；高频低延迟场景直接函数调用更快。

### 核心步骤

1. **构建 MCP Server** — 使用 `mcp.server.Server` 定义 tools（功能调用）和 resources（数据访问）
2. **实现工具处理器** — 在 `call_tool` 中根据工具名分发到具体实现
3. **连接 MCP Server** — 使用 `stdio_client` 连接本地服务器，`list_tools()` 发现可用工具
4. **转换为 OpenAI 格式** — 将 MCP tool schema 转换为 OpenAI function calling 格式
5. **多服务器连接** — 同时连接多个 MCP server，使用命名空间前缀避免工具名冲突

### 模板说明

- MCP Server — 定义 query_database 和 send_email 工具的完整服务器
- MCP Client — 连接服务器、发现工具、执行 agent 循环的完整客户端
- Resources 和 Prompts — 暴露数据源和可复用 prompt 模板
- 多服务器 Agent — 同时连接 database、email、filesystem 三个服务器

### 常见陷阱

1. **服务器崩溃** — MCP server 进程异常退出，添加健康检查和自动重启
2. **工具名冲突** — 多个服务器有同名工具，使用 `server_name__tool_name` 命名空间
3. **Schema 漂移** — 服务器更新破坏客户端兼容性，锁定服务器版本并验证 schema
4. **安全风险** — MCP server 暴露敏感操作，对工具调用进行认证和授权
5. **大资源读取** — 通过 resource 读取大文件导致性能问题，实现分页和流式传输
