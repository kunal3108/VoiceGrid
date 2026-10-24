# VoiceGrid Learning Journal: System Design

Technology: System Design Patterns and Architecture
Relevance to VoiceGrid: Every stage — architectural decisions, scaling, reliability, observability
Last updated: 2026-10-24

---

## Why System Design Matters to VoiceGrid

VoiceGrid is not just a collection of components — it is a distributed system with specific latency, throughput, and reliability requirements. Every architectural decision (API gateway design, queue depth limits, worker pool sizing, caching strategy) is a system design choice. Understanding the patterns and trade-offs behind these choices is required to make good decisions and to explain them clearly in portfolio presentations and technical interviews.

Key applications:
- **Stage 2:** Message queue design, backpressure, reliability patterns
- **Stage 3:** Worker pool design, horizontal scaling
- **Stage 4:** API gateway, request routing, pipeline architecture
- **Stage 8:** Cloud deployment, auto-scaling, SLA/SLO design

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| Load Balancing | Not Started | — | — | No | — |
| API Gateway patterns | Not Started | — | — | No | — |
| Message Queue design | Not Started | — | — | No | — |
| Rate Limiting | Not Started | — | — | No | — |
| Circuit Breaker | Not Started | — | — | No | — |
| Bulkhead pattern | Not Started | — | — | No | — |
| Timeout and Retry design | Not Started | — | — | No | — |
| Horizontal vs Vertical Scaling | Not Started | — | — | No | — |
| Stateless vs Stateful services | Not Started | — | — | No | — |
| Data consistency patterns | Not Started | — | — | No | — |
| Caching strategies (write-through, write-back, aside) | Not Started | — | — | No | — |
| Service Mesh concepts | Not Started | — | — | No | — |
| Observability (metrics, logs, traces) | Not Started | — | — | No | — |
| Capacity planning | Not Started | — | — | No | — |
| SLA/SLO/SLI design | Not Started | — | — | No | — |
| Disaster Recovery | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order to track VoiceGrid stage requirements:

1. **API Gateway patterns** — Required for Stage 4 gateway design
2. **Rate Limiting** — Required for Stage 4 (protect inference capacity)
3. **Message Queue design** — Required for Stage 2 (Redis Streams queue design)
4. **Timeout and Retry design** — Required for Stage 2 and 4 (inference reliability)
5. **Circuit Breaker** — Required for Stage 4 pipeline reliability
6. **Bulkhead pattern** — Required for isolating worker failures in Stage 3
7. **Horizontal vs Vertical Scaling** — Required for Stage 3 and Stage 8
8. **Stateless vs Stateful services** — Required for Stage 8 Kubernetes deployment
9. **Caching strategies** — Required for Stage 6 (RAG caching) and Stage 8
10. **Load Balancing** — Required for Stage 8 Kubernetes service configuration
11. **Observability** — Required for Stage 7 (benchmarking) and Stage 8 (production)
12. **SLA/SLO/SLI design** — Required for Stage 7 (defining and measuring success)
13. **Data consistency patterns** — Required for understanding Redis durability trade-offs
14. **Capacity planning** — Required for Stage 8 (sizing GKE cluster)
15. **Service Mesh concepts** — Background knowledge for Stage 8 production deployment
16. **Disaster Recovery** — Required for Stage 8 (GKE cluster recovery planning)

---

## Topic Notes

### API Gateway Patterns
- **What:** A single entry point for all client requests. Handles authentication, rate limiting, request routing, and response aggregation.
- **VoiceGrid gateway responsibilities:**
  - Accept audio upload (multipart form or binary body)
  - Validate request (format, size, auth token)
  - Assign priority based on client tier or request type
  - Submit task to Redis queue
  - Return task_id for async polling
  - Serve task results when ready
- **Pattern: Gateway offloads concerns from upstream services:** auth, rate limiting, request logging, SSL termination all happen at the gateway, not in inference workers
- **Notes:** (empty)

### Rate Limiting
- **Purpose:** Prevent individual clients from overwhelming the system; ensure fair access
- **Algorithms:**
  1. **Token Bucket:** Each client has a bucket of tokens; each request consumes a token; bucket refills at a fixed rate. Allows burst traffic.
  2. **Leaky Bucket:** Requests queue up; processed at fixed rate. Smooths traffic but adds latency.
  3. **Fixed Window Counter:** Count requests in fixed time windows. Simple but allows bursting at window boundary.
  4. **Sliding Window:** More accurate than fixed window; tracks rolling count.
- **VoiceGrid implementation:** Token bucket per client tier, stored in Redis (INCR + TTL pattern)
- **Response:** HTTP 429 Too Many Requests with Retry-After header
- **Notes:** (empty)

### Message Queue Design
- **Key design decisions:**
  1. **Ordering guarantees:** FIFO only? Priority? Partial ordering by key?
  2. **Delivery semantics:** At-most-once, at-least-once, exactly-once?
  3. **Consumer model:** Competing consumers (one message → one consumer) or fan-out (one message → many)?
  4. **Durability:** Survive process restart? Node failure? Disk failure?
  5. **Backpressure:** What happens when producers outpace consumers?
- **VoiceGrid design:**
  - Multiple priority streams (one per priority level, or sorted set with score = priority)
  - At-least-once delivery via Redis consumer groups
  - Competing consumers (each task processed by exactly one worker)
  - Persistent with AOF for durability
  - Backpressure via XLEN check + HTTP 429
- **Notes:** (empty)

### Circuit Breaker
- **States:**
  - **Closed:** Normal operation. Requests pass through. Failure counter tracked.
  - **Open:** Too many failures. Requests immediately rejected (fast fail). Recovery timer started.
  - **Half-Open:** Recovery timer expired. One test request allowed. If it succeeds → Closed; if fails → Open.
- **Parameters:** failure_threshold (e.g., 5 failures in 60s), recovery_timeout (e.g., 30s)
- **VoiceGrid use:** Wrap LLM inference call with circuit breaker. If vLLM is unresponsive, open circuit and return error immediately rather than piling up requests.
- **Notes:** (empty)

### Bulkhead Pattern
- **What:** Isolate different parts of the system so a failure in one doesn't cascade to others
- **Analogy:** Ship bulkheads prevent flooding the entire ship when one compartment is breached
- **VoiceGrid application:**
  - Separate worker pools for ASR, LLM, and TTS — a crashed TTS worker doesn't stop ASR processing
  - Separate Redis streams per priority level — a flood of low-priority tasks doesn't block real-time tasks
  - Thread pool isolation: high-priority workers cannot be starved by low-priority work
- **Notes:** (empty)

### Timeout and Retry Design
- **Timeout:** Maximum time to wait for a response. Required on every external call.
- **VoiceGrid timeout values (target, not yet measured):**
  - ASR: 100ms timeout
  - LLM: 300ms timeout (TTFT; then stream)
  - TTS: 100ms for first chunk
  - Redis operations: 50ms
- **Retry logic:** Only retry on transient failures (network error, temporary resource unavailability). Do not retry on 4xx client errors.
- **Exponential backoff + jitter:** Required to prevent retry storms
- **Maximum retry attempts:** 3 attempts for task queue operations; 1 attempt for real-time inference (retry adds latency beyond SLA)
- **Notes:** (empty)

### Horizontal vs Vertical Scaling
- **Vertical scaling:** Add more resources to a single node (more RAM, faster GPU). Limited by hardware ceiling.
- **Horizontal scaling:** Add more nodes. Requires stateless services or explicit state distribution.
- **VoiceGrid scaling model:**
  - Gateway: Stateless → Horizontal (add more gateway pods)
  - Workers: Stateless → Horizontal (add more worker pods; Redis queue distributes work)
  - Redis: Vertical first, then Redis Cluster for horizontal
  - LLM inference: Horizontal via vLLM (multiple replicas); Vertical via tensor parallelism (multi-GPU per node)
- **Key insight:** Horizontal scaling works only if services are stateless or state is shared via external store (Redis, PostgreSQL)
- **Notes:** (empty)

### SLA/SLO/SLI Design
- **SLI (Service Level Indicator):** A metric that measures service quality (e.g., P99 latency, error rate)
- **SLO (Service Level Objective):** A target for an SLI (e.g., P99 latency < 500ms for 99.5% of requests)
- **SLA (Service Level Agreement):** A contract that includes SLOs and consequences for missing them
- **VoiceGrid SLOs (target, not yet measured):**
  - P50 end-to-end latency < 200ms
  - P99 end-to-end latency < 500ms
  - Error rate < 0.1% (excluding client errors)
  - GPU utilization > 80% under sustained load
- **Notes:** (empty)

### Caching Strategies
| Strategy | Description | Write Performance | Read Performance | Consistency |
|----------|-------------|------------------|-----------------|-------------|
| Cache-aside | App reads cache; on miss, loads from DB and writes cache | No extra write overhead | Fast on hit | Stale on cache miss |
| Write-through | Write to cache and DB simultaneously | Slower writes | Fast reads | Strong |
| Write-back | Write to cache only; async flush to DB | Fast writes | Fast reads | Eventual |

- **VoiceGrid use:** Cache-aside for inference results (cache completed task results in Redis); write-through for session metadata
- **Notes:** (empty)

### Observability
- **Metrics to expose (Prometheus):**
  - `voicegrid_request_latency_seconds` (histogram with labels: stage, priority)
  - `voicegrid_queue_depth` (gauge with labels: priority)
  - `voicegrid_gpu_utilization` (gauge with labels: device)
  - `voicegrid_error_total` (counter with labels: stage, error_type)
- **Tracing:** OpenTelemetry spans for each pipeline stage; trace propagated via request headers
- **Logging:** Structured JSON; every log includes trace_id, task_id, stage, outcome
- **Notes:** (empty)

---

## VoiceGrid System Design Summary

```
Internet
    │
    ▼
[Load Balancer (GKE Ingress)]
    │
    ▼
[API Gateway (FastAPI)] ← Rate Limiter, Auth, Circuit Breaker
    │
    ▼ (task submission)
[Redis Streams] ← Backpressure check (XLEN)
    │
    ├──▶ [High-Priority Worker Pool] ← Bulkhead
    │
    ├──▶ [Normal-Priority Worker Pool] ← Bulkhead
    │
    └──▶ [Low-Priority Worker Pool] ← Bulkhead
           │
           ▼
    [Inference Pipeline (ASR → LLM → TTS)]
           │
           ▼
    [Result stored in Redis / PostgreSQL]
           │
           ▼ (async polling or webhook)
    [Client]
```

---

## Resources

- (to be added as topics are studied)

---

## Implementation References

| Topic | File | Stage | Date | Notes |
|-------|------|-------|------|-------|
| (empty) | — | — | — | — |

---

## Open Questions

- (empty — add questions as they arise)
