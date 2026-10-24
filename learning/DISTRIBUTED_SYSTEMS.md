# VoiceGrid Learning Journal: Distributed Systems

Technology: Distributed Systems Concepts and Patterns
Relevance to VoiceGrid: Stages 2, 3, 4, 7, 8 — coordination, reliability, scalability
Last updated: 2026-10-24

---

## Why Distributed Systems Matter to VoiceGrid

VoiceGrid is a distributed system: multiple worker nodes, a shared task queue (Redis), and coordinated inference pipelines. Every design decision in Stages 2-8 involves distributed systems trade-offs. Without understanding these concepts, it is easy to build systems that fail silently, lose tasks, deadlock under load, or violate SLAs in ways that are hard to debug.

Key applications:
- **Stage 2:** At-least-once delivery, idempotency, distributed task coordination
- **Stage 3:** Work stealing, concurrent worker coordination
- **Stage 4:** Pipeline reliability, graceful degradation, retry patterns
- **Stage 8:** Kubernetes deployment, horizontal scaling, health checks, observability

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| Concurrency vs Parallelism | Not Started | — | — | No | — |
| Distributed Coordination | Not Started | — | — | No | — |
| Idempotency | Not Started | — | — | No | — |
| At-least-once vs At-most-once vs Exactly-once | Not Started | — | — | No | — |
| Retry Patterns with Backoff | Not Started | — | — | No | — |
| Leases and Heartbeats | Not Started | — | — | No | — |
| Consistency Models (eventual, strong) | Not Started | — | — | No | — |
| CAP Theorem | Not Started | — | — | No | — |
| Backpressure | Not Started | — | — | No | — |
| Work Stealing | Not Started | — | — | No | — |
| Priority Scheduling in Distributed Context | Not Started | — | — | No | — |
| Circuit Breakers | Not Started | — | — | No | — |
| Fault Tolerance Patterns | Not Started | — | — | No | — |
| Health Checks | Not Started | — | — | No | — |
| Observability (metrics/traces/logs) | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order:

1. **Concurrency vs Parallelism** — Clarify the terminology before everything else
2. **At-least-once vs At-most-once vs Exactly-once** — Foundational delivery semantics; required before Stage 2
3. **Idempotency** — Required corollary to at-least-once delivery; learn together
4. **Retry Patterns with Backoff** — Applied immediately in Stage 2 for failed task handling
5. **Distributed Coordination** — Understand the coordination problem before implementing a solution
6. **Leases and Heartbeats** — Required for worker health monitoring in Stage 3
7. **Health Checks** — Applied in Stage 8 (Kubernetes liveness/readiness probes)
8. **Backpressure** — Required for Stage 2 queue design
9. **Work Stealing** — Required for Stage 3 worker pool
10. **Priority Scheduling in Distributed Context** — Required for Stage 2 (Redis priority queue design)
11. **CAP Theorem** — Understand consistency vs. availability trade-offs before Stage 8
12. **Consistency Models** — Required for understanding Redis durability guarantees
13. **Circuit Breakers** — Applied in Stage 4 pipeline reliability
14. **Fault Tolerance Patterns** — Broader view; covers bulkheads, timeouts, fallbacks
15. **Observability** — Required for Stage 7 (benchmarking) and Stage 8 (production monitoring)

---

## Topic Notes

### Concurrency vs Parallelism
- **Concurrency:** Multiple tasks are in progress at the same time; they may share CPU via interleaving
- **Parallelism:** Multiple tasks run at the exact same instant; requires multiple CPU cores
- **VoiceGrid distinction:**
  - C++ worker pool (Stage 3): True parallelism — threads run on separate cores simultaneously
  - Python async pipeline (Stage 4): Concurrency — asyncio interleaves I/O-bound operations on one thread
- **Notes:** (empty)

### At-least-once vs At-most-once vs Exactly-once
| Semantic | Meaning | Risk | VoiceGrid Use |
|----------|---------|------|---------------|
| At-most-once | Message delivered 0 or 1 times | Task may be lost | Not acceptable for inference tasks |
| At-least-once | Message delivered 1 or more times | Task may be duplicated | Default for Redis consumer groups |
| Exactly-once | Message delivered exactly 1 time | Complex to implement | Achieved via idempotency + at-least-once |

- **Redis consumer groups:** Provide at-least-once. Exactly-once requires idempotent consumer logic.
- **Notes:** (empty)

### Idempotency
- **Definition:** An operation is idempotent if applying it multiple times has the same effect as applying it once
- **Why required:** With at-least-once delivery, a task may be executed more than once. If execution is not idempotent, the second execution may corrupt state or double-count results.
- **VoiceGrid approach:** Each task has a unique task_id. Worker checks "has this task_id already been processed?" before executing. If yes, acknowledge and skip.
- **Implementation:** Use Redis SET task_id "done" with TTL as the idempotency record.
- **Notes:** (empty)

### Retry Patterns with Backoff
- **Naive retry:** Retry immediately on failure → amplifies load on already-failing system
- **Fixed backoff:** Wait constant time between retries → better, but slow to recover
- **Exponential backoff:** Double the wait time after each failure → standard practice
- **Exponential backoff with jitter:** Add randomness to avoid all retriers firing simultaneously (thundering herd)
- **Formula:** wait = min(cap, base * 2^n) + random_jitter
- **VoiceGrid use:** Stage 2 worker retry logic for failed Redis operations and inference failures
- **Notes:** (empty)

### Leases and Heartbeats
- **Lease:** A time-bounded claim on a resource. If not renewed before expiry, claim is lost.
- **Heartbeat:** Periodic signal proving that a worker is still alive
- **In Redis:** Worker claims a task and sets a TTL (the lease). If worker crashes, TTL expires and another worker can reclaim the task (via XAUTOCLAIM).
- **VoiceGrid use:** Task lease TTL in Redis Streams (Stage 2); worker heartbeat in Stage 3
- **Notes:** (empty)

### CAP Theorem
- **C = Consistency:** Every read receives the most recent write
- **A = Availability:** Every request receives a (possibly stale) response
- **P = Partition Tolerance:** System operates despite network partitions
- **CAP says:** In the presence of a partition, you must choose between C and A
- **Redis default:** CP (prioritizes consistency when there is a partition; may refuse writes)
- **VoiceGrid implication:** Under network partition, Redis may reject task publications. VoiceGrid must handle this case (queue locally, retry).
- **Notes:** (empty)

### Backpressure
- **Problem:** Producers generate tasks faster than consumers can process them. Queue grows without bound. Memory exhaustion. SLA violations.
- **Solutions:**
  1. **Drop:** Reject new tasks when queue exceeds threshold (fast, but clients receive errors)
  2. **Block:** Block producer until queue drains (slow producer, prevents overload)
  3. **Spill:** Overflow to a secondary slower queue (complex, but graceful)
  4. **Shed load:** Drop lowest-priority tasks first when overloaded
- **VoiceGrid approach:** Drop with backpressure signal to client (HTTP 429 Too Many Requests), then shed lowest-priority tasks
- **Notes:** (empty)

### Circuit Breakers
- **States:** Closed (normal) → Open (failing, reject fast) → Half-Open (testing recovery)
- **When to use:** Protect against calling a failing dependency (e.g., LLM service is down)
- **Parameters:** Failure threshold (how many failures to open), reset timeout (how long until half-open)
- **VoiceGrid use:** Stage 4 pipeline — if LLM inference node is unresponsive, open circuit and return error fast rather than queuing indefinitely
- **Notes:** (empty)

### Observability
- **Three pillars:**
  1. **Metrics:** Numeric values over time (e.g., P99 latency, GPU utilization, queue depth)
  2. **Traces:** End-to-end path of a single request across services
  3. **Logs:** Discrete events with context (structured logs preferred)
- **VoiceGrid requirements:**
  - Per-request trace: ASR start/end, LLM start/end, TTS start/end, total
  - System metrics: GPU utilization, queue depth, consumer lag, error rate
  - Structured logs: task_id, priority, latency, outcome in every log entry
- **Tools:** Prometheus (metrics), OpenTelemetry (traces), structured JSON logging
- **Notes:** (empty)

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
