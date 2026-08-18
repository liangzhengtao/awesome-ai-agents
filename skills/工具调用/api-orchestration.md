# API Orchestration Patterns

> Combine multiple APIs, services, and data sources into coherent agent workflows — the glue that makes agents actually useful in production.

## When to Use

| Scenario | Use API Orchestration? |
|----------|----------------------|
| Agent needs to call multiple APIs in sequence | Yes |
| Combining results from different services | Yes |
| Building data pipelines for agents | Yes |
| Error recovery across API boundaries | Yes |
| Single API call | Just use `requests` directly |
| Real-time streaming from multiple sources | Yes, with async patterns |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   API Orchestrator                        │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Request Router                       │    │
│  │  ┌───────┐  ┌───────────┐  ┌──────────────────┐ │    │
│  │  │ Auth  │  │  Rate     │  │  Circuit Breaker │ │    │
│  │  │ Token │  │  Limiter  │  │                  │ │    │
│  │  └───────┘  └───────────┘  └──────────────────┘ │    │
│  └──────────────────────┬──────────────────────────┘    │
│                         │                                │
│         ┌───────────────┼───────────────┐                │
│         ▼               ▼               ▼                │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │  API A     │  │  API B     │  │  API C     │        │
│  │  (Search)  │  │  (Weather) │  │  (Payments)│        │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘        │
│        │               │               │                 │
│        └───────────────┼───────────────┘                 │
│                        ▼                                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │           Response Aggregator                    │    │
│  │           Cache / Dedup / Transform              │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Resilient API Client

```python
"""Production-grade API client with retries, caching, and circuit breaker."""
import time
import httpx
from functools import lru_cache
from dataclasses import dataclass, field
from typing import Any

@dataclass
class CircuitBreaker:
    failure_threshold: int = 5
    recovery_timeout: float = 60.0
    failure_count: int = field(default=0, init=False)
    last_failure: float = field(default=0.0, init=False)
    state: str = field(default="closed", init=False)  # closed, open, half-open

    def record_failure(self):
        self.failure_count += 1
        self.last_failure = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "open"

    def record_success(self):
        self.failure_count = 0
        self.state = "closed"

    def allow_request(self) -> bool:
        if self.state == "closed":
            return True
        if self.state == "open":
            if time.time() - self.last_failure > self.recovery_timeout:
                self.state = "half-open"
                return True
            return False
        return True  # half-open

class ResilientAPIClient:
    def __init__(self, base_url: str, api_key: str, timeout: float = 30.0):
        self.base_url = base_url
        self.client = httpx.AsyncClient(
            base_url=base_url,
            headers={"Authorization": f"Bearer {api_key}"},
            timeout=timeout,
        )
        self.breaker = CircuitBreaker()
        self._cache: dict[str, tuple[float, Any]] = {}
        self._cache_ttl = 300  # 5 minutes

    async def request(self, method: str, path: str, **kwargs) -> dict:
        """Make an API request with retry and circuit breaker."""
        if not self.breaker.allow_request():
            raise RuntimeError(f"Circuit breaker open for {self.base_url}")

        cache_key = f"{method}:{path}:{kwargs}"
        if cache_key in self._cache:
            ts, data = self._cache[cache_key]
            if time.time() - ts < self._cache_ttl:
                return data

        for attempt in range(3):
            try:
                response = await self.client.request(method, path, **kwargs)
                response.raise_for_status()
                data = response.json()

                self.breaker.record_success()
                self._cache[cache_key] = (time.time(), data)
                return data

            except httpx.HTTPStatusError as e:
                if e.response.status_code == 429:  # Rate limited
                    wait = float(e.response.headers.get("Retry-After", 2 ** attempt))
                    await self._async_sleep(wait)
                    continue
                self.breaker.record_failure()
                raise
            except httpx.RequestError:
                self.breaker.record_failure()
                if attempt == 2:
                    raise
                await self._async_sleep(2 ** attempt)

    async def _async_sleep(self, seconds: float):
        import asyncio
        await asyncio.sleep(seconds)

    async def close(self):
        await self.client.aclose()
```

## Code Template: Parallel API Calls with Aggregation

```python
"""Call multiple APIs in parallel and merge results."""
import asyncio
import httpx

async def fetch_user_context(user_id: str) -> dict:
    """Gather user data from multiple services in parallel."""
    async with httpx.AsyncClient() as client:
        # Fire all requests simultaneously
        profile, orders, preferences, recommendations = await asyncio.gather(
            client.get(f"https://api.internal/users/{user_id}"),
            client.get(f"https://api.internal/orders?user_id={user_id}"),
            client.get(f"https://api.internal/preferences/{user_id}"),
            client.get(f"https://api.ml/recommendations/{user_id}"),
            return_exceptions=True,  # Don't fail on one error
        )

    result = {}

    # Handle each response, gracefully handling failures
    for name, response in [
        ("profile", profile),
        ("orders", orders),
        ("preferences", preferences),
        ("recommendations", recommendations),
    ]:
        if isinstance(response, Exception):
            result[name] = {"error": str(response)}
        elif response.status_code == 200:
            result[name] = response.json()
        else:
            result[name] = {"error": f"HTTP {response.status_code}"}

    return result
```

## Code Template: API Chain with Fallbacks

```python
"""Chain APIs with automatic fallback to alternative providers."""
from dataclasses import dataclass
from typing import Callable, Any

@dataclass
class APIProvider:
    name: str
    call: Callable[..., Any]
    priority: int  # Lower = tried first

class APIChain:
    def __init__(self):
        self.providers: list[APIProvider] = []

    def register(self, name: str, call: Callable, priority: int = 10):
        self.providers.append(APIProvider(name=name, call=call, priority=priority))
        self.providers.sort(key=lambda p: p.priority)

    async def execute(self, *args, **kwargs) -> tuple[str, Any]:
        """Try providers in priority order. Returns (provider_name, result)."""
        errors = []
        for provider in self.providers:
            try:
                result = await provider.call(*args, **kwargs)
                return provider.name, result
            except Exception as e:
                errors.append(f"{provider.name}: {e}")
                continue

        raise RuntimeError(f"All providers failed:\n" + "\n".join(errors))

# Usage
chain = APIChain()
chain.register("openai", call_openai_embedding, priority=1)
chain.register("cohere", call_cohere_embedding, priority=2)
chain.register("local", call_local_embedding, priority=3)

provider, result = await chain.execute(text="Hello world")
```

## Code Template: Rate-Limited API Gateway

```python
"""Shared rate limiter for multiple API consumers."""
import asyncio
import time
from collections import defaultdict

class RateLimiter:
    """Token bucket rate limiter per API endpoint."""

    def __init__(self):
        self.buckets: dict[str, dict] = defaultdict(lambda: {
            "tokens": 0,
            "max_tokens": 10,
            "refill_rate": 1.0,  # tokens per second
            "last_refill": time.time(),
        })

    def configure(self, endpoint: str, max_tokens: int, refill_rate: float):
        self.buckets[endpoint]["max_tokens"] = max_tokens
        self.buckets[endpoint]["refill_rate"] = refill_rate
        self.buckets[endpoint]["tokens"] = max_tokens

    async def acquire(self, endpoint: str):
        """Wait until a token is available."""
        bucket = self.buckets[endpoint]
        while True:
            now = time.time()
            elapsed = now - bucket["last_refill"]
            bucket["tokens"] = min(
                bucket["max_tokens"],
                bucket["tokens"] + elapsed * bucket["refill_rate"],
            )
            bucket["last_refill"] = now

            if bucket["tokens"] >= 1:
                bucket["tokens"] -= 1
                return

            # Wait for refill
            wait = (1 - bucket["tokens"]) / bucket["refill_rate"]
            await asyncio.sleep(wait)

# Usage
limiter = RateLimiter()
limiter.configure("openai", max_tokens=60, refill_rate=1.0)
limiter.configure("google_search", max_tokens=100, refill_rate=10.0)

async def call_with_rate_limit(endpoint: str, fn, *args):
    await limiter.acquire(endpoint)
    return await fn(*args)
```

## Patterns

### Pattern: Request/Response Middleware
```python
class Middleware:
    async def before_request(self, request: dict) -> dict:
        """Transform request before sending."""
        return request

    async def after_response(self, response: dict) -> dict:
        """Transform response after receiving."""
        return response

class LoggingMiddleware(Middleware):
    async def before_request(self, request):
        print(f"→ {request['method']} {request['url']}")
        return request

    async def after_response(self, response):
        print(f"← {response.status_code}")
        return response
```

### Pattern: API Response Normalization
```python
def normalize_weather(provider: str, raw: dict) -> dict:
    """Normalize different weather APIs to a common format."""
    if provider == "openweathermap":
        return {
            "temp_c": raw["main"]["temp"],
            "condition": raw["weather"][0]["description"],
            "humidity": raw["main"]["humidity"],
        }
    elif provider == "weatherapi":
        return {
            "temp_c": raw["current"]["temp_c"],
            "condition": raw["current"]["condition"]["text"],
            "humidity": raw["current"]["humidity"],
        }
```

### Pattern: Event-Driven API Updates
```python
"""Subscribe to API webhooks instead of polling."""
from fastapi import FastAPI, Request

app = FastAPI()
agent_state = {}

@app.post("/webhook/stripe")
async def stripe_webhook(request: Request):
    event = await request.json()
    if event["type"] == "payment_intent.succeeded":
        agent_state["last_payment"] = event["data"]["object"]
    return {"status": "ok"}
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| No timeout | API hangs indefinitely | Always set connection + read timeouts |
| Sequential calls | Slow when APIs are independent | Use `asyncio.gather` for parallel calls |
| No retry logic | Transient failures break agent | Exponential backoff with jitter |
| Cache invalidation | Stale data served from cache | Use TTL-based cache + manual invalidation |
| API key exposure | Keys leaked in logs or errors | Mask sensitive headers in logs |
| No circuit breaker | Failing API drags down whole agent | Open circuit after N failures |

## Proven Patterns

1. **Use `httpx` over `requests`** — async support, connection pooling, HTTP/2
2. **Set timeouts on every request** — both connection and read timeouts
3. **Implement circuit breakers** — prevent cascading failures
4. **Cache aggressively** — API calls are expensive; cache responses with TTL
5. **Normalize responses** — different APIs return different shapes; unify them early
6. **Log request/response metadata** — URL, status code, latency; never log API keys

## See Also

- [Function Calling](./function-calling.md) — LLM tool use protocol
- [MCP Integration](./mcp-integration.md) — Standardized tool servers
- [Custom Agents](../Agent框架/custom-agents.md) — Agent loop fundamentals
- [Task Decomposition](../多Agent协作/task-decomposition.md) — Breaking tasks into API calls

---

## 中文版本

### 使用场景

- Agent 需要按顺序调用多个 API
- 合并来自不同服务的结果
- 为 agent 构建数据管道
- 跨 API 边界的错误恢复

> 单个 API 调用直接使用 requests 即可，无需编排。

### 核心步骤

1. **弹性 API 客户端** — 实现重试、缓存（TTL）、熔断器（CircuitBreaker）的生产级客户端
2. **并行 API 调用** — 使用 `asyncio.gather` 同时调用多个独立 API，`return_exceptions=True` 优雅处理失败
3. **API 链 + 降级** — 注册多个 provider 按优先级尝试，自动降级到备选方案
4. **速率限制** — 实现 Token Bucket 速率限制器，按 API endpoint 独立限流
5. **响应标准化** — 不个 API 返回格式不同，统一转换为通用格式

### 模板说明

- ResilientAPIClient — 带重试、缓存、熔断器的 httpx 异步客户端
- 并行 API 调用 — asyncio.gather 同时获取用户资料、订单、偏好、推荐
- API Chain — 多 provider 降级链（OpenAI → Cohere → 本地模型）
- RateLimiter — Token Bucket 速率限制器实现

### 常见陷阱

1. **无超时** — API 无限挂起，始终设置连接超时和读取超时
2. **顺序调用** — 独立 API 串行调用浪费时间，使用 asyncio.gather 并行
3. **无重试逻辑** — 瞬态故障导致 agent 失败，使用指数退避 + 抖动重试
4. **缓存失效** — 缓存返回过期数据，使用 TTL 缓存 + 手动失效机制
5. **API key 泄露** — 密钥泄露到日志或错误信息中，日志中遮蔽敏感 header
