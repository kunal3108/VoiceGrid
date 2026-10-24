# VoiceGrid Learning Journal: Python

Technology: Python 3.11+
Relevance to VoiceGrid: API gateway, inference pipeline, Redis coordination, testing, observability
Last updated: 2026-10-24

---

## Why Python Matters to VoiceGrid

Python is the primary language for VoiceGrid's API layer, inference pipeline (ASR, LLM, TTS), Redis consumers, and testing. The AI/ML ecosystem is predominantly Python-based, and VoiceGrid's inference components (Whisper, vLLM, TTS libraries) all have Python interfaces. Understanding Python's async model, performance characteristics, and testing patterns is required for Stages 4-8.

Key applications:
- **Stage 4:** FastAPI gateway, ASR/LLM/TTS pipeline, async request handling
- **Stage 2:** Redis consumer workers (redis-py with asyncio)
- **Stage 6:** LangGraph orchestration layer
- **Stages 7-8:** Benchmark scripts, integration tests, observability configuration

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| Async/Await and asyncio | Not Started | — | — | No | — |
| FastAPI | Not Started | — | — | No | — |
| Pydantic | Not Started | — | — | No | — |
| HTTP clients (httpx/aiohttp) | Not Started | — | — | No | — |
| WebSockets | Not Started | — | — | No | — |
| Redis with Python (redis-py) | Not Started | — | — | No | — |
| Process management | Not Started | — | — | No | — |
| Logging and observability | Not Started | — | — | No | — |
| Unit testing (pytest) | Not Started | — | — | No | — |
| Integration testing | Not Started | — | — | No | — |
| Performance profiling (cProfile, py-spy) | Not Started | — | — | No | — |
| Type hints | Not Started | — | — | No | — |
| Packaging | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order:

1. **Type hints** — Apply immediately; makes all subsequent code clearer and safer
2. **Async/Await and asyncio** — Required for FastAPI and Redis async client; learn before Stage 4
3. **Pydantic** — Required for FastAPI request/response models; learn alongside FastAPI
4. **FastAPI** — Required for Stage 4 gateway; learn after async and Pydantic
5. **HTTP clients (httpx)** — Required for testing FastAPI and inter-service calls
6. **Redis with Python (redis-py)** — Required for Stage 2 Redis consumer
7. **Unit testing (pytest)** — Required for all stages; learn early and apply throughout
8. **WebSockets** — Required for streaming audio to/from the gateway
9. **Logging and observability** — Required for Stage 7 benchmarking and Stage 8 production
10. **Integration testing** — Required for Stage 4 end-to-end test
11. **Performance profiling** — Required for Stage 5 Python-side bottleneck analysis
12. **Process management** — Required for Stage 8 container management and worker lifecycle
13. **Packaging** — Required for Stage 8 Docker image creation

---

## Topic Notes

### Async/Await and asyncio
- **What:** Python's cooperative multitasking model. A single thread switches between coroutines at await points.
- **When to use:** I/O-bound operations (HTTP, Redis, file I/O). Not for CPU-bound operations.
- **Event loop:** asyncio.get_event_loop() or asyncio.run() as the entry point
- **Key primitives:** async def, await, asyncio.gather() (concurrent), asyncio.Queue, asyncio.Lock
- **VoiceGrid use:** FastAPI gateway is async; Redis consumer is async; LLM streaming response is async
- **Common mistake:** Calling a synchronous (blocking) function inside async code — this blocks the event loop and defeats the purpose of async
- **Notes:** (empty)

### FastAPI
- **What:** Modern Python web framework built on Starlette + Pydantic with automatic OpenAPI docs
- **Key features:** Async-native, type-safe via Pydantic, automatic validation, dependency injection
- **VoiceGrid gateway routes:**
  - POST /inference/voice — Accept audio upload, return task_id
  - GET /inference/{task_id}/status — Check task status
  - GET /inference/{task_id}/result — Retrieve audio result
  - GET /health — Health check endpoint
- **Middleware requirements:** Rate limiting, request ID injection, latency logging
- **Notes:** (empty)

### Pydantic
- **What:** Data validation and serialization library using Python type annotations
- **VoiceGrid use:** Define request/response schemas; validate Redis message payloads; define task schemas
- **Key concepts:** BaseModel, Field validators, model_validator, serialization (model_dump), parsing (model_validate)
- **Notes:** (empty)

### Redis with Python (redis-py)
- **Key library:** `redis` package, or `redis.asyncio` for async usage
- **Connection:** `redis.asyncio.from_url("redis://localhost:6379")`
- **VoiceGrid patterns:**
  - Stream consumer: `await r.xreadgroup(...)`
  - Stream producer: `await r.xadd(...)`
  - Distributed lock: `await r.set("lock", worker_id, nx=True, px=5000)`
  - Idempotency check: `await r.get(f"processed:{task_id}")`
- **Notes:** (empty)

### Unit Testing (pytest)
- **VoiceGrid test structure:**
  - tests/unit/ — Unit tests (no external dependencies)
  - tests/integration/ — Integration tests (require Redis, model loaded)
  - tests/benchmarks/ — Performance benchmarks
- **Key patterns:**
  - `@pytest.fixture` for setup/teardown
  - `pytest.mark.asyncio` for testing async functions
  - `pytest.raises` for exception testing
  - `unittest.mock.patch` or `pytest-mock` for mocking dependencies
- **VoiceGrid rule:** Every module must have a corresponding test file. No feature complete without tests.
- **Notes:** (empty)

### Performance Profiling
- **cProfile:** Built-in Python profiler. Use for function-level profiling of CPU-bound code.
  - Run: `python -m cProfile -s cumtime my_script.py`
- **py-spy:** Sampling profiler, zero code change required, works on running processes
  - Run: `py-spy record -o profile.svg -- python my_script.py`
- **VoiceGrid use:** Profile inference pipeline in Stage 5 to identify Python-level bottlenecks before implementing CUDA optimizations
- **Notes:** (empty)

### Logging and Observability
- **Structured logging:** Use JSON format for all log messages; include task_id, priority, latency, outcome in every inference log
- **Library:** `structlog` or `python-json-logger` for structured logs
- **Key fields to log:**
  ```python
  log.info("inference_complete",
      task_id=task.id,
      priority=task.priority,
      asr_latency_ms=asr_ms,
      llm_latency_ms=llm_ms,
      tts_latency_ms=tts_ms,
      total_latency_ms=total_ms,
      outcome="success"
  )
  ```
- **Metrics:** Use `prometheus_client` to expose latency histograms and GPU utilization gauges
- **Notes:** (empty)

### Type Hints
- **Why:** Type hints enable static analysis (mypy), IDE autocompletion, and clearer interfaces
- **Key types:** `str`, `int`, `float`, `bool`, `list[T]`, `dict[K, V]`, `Optional[T]`, `Union[A, B]`, `TypedDict`
- **VoiceGrid rule:** All function signatures must have type hints. mypy must pass without errors.
- **Notes:** (empty)

---

## Python Anti-Patterns to Avoid in VoiceGrid

| Anti-Pattern | Problem | Preferred Alternative |
|-------------|---------|----------------------|
| `requests` in async code | Blocks event loop | `httpx.AsyncClient` |
| `time.sleep()` in async function | Blocks event loop | `asyncio.sleep()` |
| Global mutable state | Race conditions | Dependency injection, explicit state |
| `except Exception: pass` | Silently swallows errors | Log and re-raise or handle specifically |
| `print()` for logging | Not structured, hard to aggregate | `structlog` with JSON format |
| Synchronous database calls in async handler | Blocks event loop | Async DB driver (asyncpg, motor) |

---

## Resources

- (to be added as topics are studied)

---

## Implementation References

| Topic | File | Hackathon | Date | Notes |
|-------|------|-----------|------|-------|
| (empty) | — | — | — | — |

---

## Open Questions

- (empty — add questions as they arise)
