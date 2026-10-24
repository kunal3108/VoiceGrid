# VoiceGrid Learning Journal: Redis

Technology: Redis 7.0+
Relevance to VoiceGrid: Distributed task coordination, work queue, cache, distributed locks (Stage 2)
Last updated: 2026-10-24

---

## Why Redis Matters to VoiceGrid

Redis is the distributed coordination layer for VoiceGrid. The C++ scheduling engine publishes tasks to Redis Streams, and Python worker processes consume them. Redis also provides distributed locks (to prevent two workers from claiming the same task) and caching (for session state and model outputs). Understanding Redis deeply — not just as a key-value store, but as a streams engine and coordination primitive — is a core learning objective.

Key applications:
- **Stage 2:** Redis Streams as the distributed task queue; consumer groups for worker coordination
- **Stage 3:** Distributed locks for worker claim logic
- **Stage 6:** Redis as a checkpointing backend for LangGraph conversation state
- **All stages:** Redis as cache for inference results and session metadata

Architecture decision: D001 documents why Redis was chosen over Kafka, RabbitMQ, and PostgreSQL.

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| Redis Data Structures (String) | Not Started | — | — | No | — |
| Redis Data Structures (Hash) | Not Started | — | — | No | — |
| Redis Data Structures (List) | Not Started | — | — | No | — |
| Redis Data Structures (Set) | Not Started | — | — | No | — |
| Redis Data Structures (Sorted Set) | Not Started | — | — | No | — |
| Redis Streams | Not Started | — | — | No | — |
| Consumer Groups | Not Started | — | — | No | — |
| Pub/Sub | Not Started | — | — | No | — |
| Redis as Task Queue | Not Started | — | — | No | — |
| Redis as Cache | Not Started | — | — | No | — |
| Redis as Session Store | Not Started | — | — | No | — |
| Distributed Locks (SETNX/Redlock) | Not Started | — | — | No | — |
| TTL and Expiration | Not Started | — | — | No | — |
| Redis Persistence (RDB/AOF) | Not Started | — | — | No | — |
| Redis Cluster basics | Not Started | — | — | No | — |
| Backpressure patterns | Not Started | — | — | No | — |
| Failure recovery patterns | Not Started | — | — | No | — |
| Python redis-py client | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order:

1. **Redis Data Structures (String, Hash, List)** — Foundational. Must know these before anything else.
2. **Redis Data Structures (Set, Sorted Set)** — Used for priority tracking and deduplication.
3. **TTL and Expiration** — Applied to almost every data structure. Learn early.
4. **Redis as Cache** — Common pattern; builds on strings and hashes.
5. **Redis as Session Store** — Builds on hashes and TTL.
6. **Redis as Task Queue** — Using Lists (LPUSH/BRPOP) before learning Streams.
7. **Redis Streams** — The primary data structure for Stage 2. Learn after List-based queues.
8. **Consumer Groups** — Required for at-least-once delivery. Learn immediately after Streams.
9. **Distributed Locks (SETNX)** — Required for worker claim logic in Stage 2.
10. **Backpressure patterns** — Required for Stage 2 worker coordination.
11. **Failure recovery patterns** — Required for Stage 2 (handling dead tasks).
12. **Redis Persistence (RDB/AOF)** — Required for understanding durability guarantees.
13. **Pub/Sub** — Understand as an alternative to Streams; know why Streams is better for task queues.
14. **Redis Cluster basics** — Required for understanding production deployment.
15. **Python redis-py client** — Applied implementation of all above using Python.

---

## Topic Notes

### Redis Data Structures Overview

| Structure | Primary Commands | VoiceGrid Use |
|-----------|-----------------|---------------|
| String | SET, GET, INCR, EXPIRE | Config values, counters, simple cache |
| Hash | HSET, HGET, HGETALL | Task metadata, session state |
| List | LPUSH, RPUSH, LRANGE, BRPOP | Simple FIFO queue (before Streams) |
| Set | SADD, SMEMBERS, SISMEMBER | Worker registry, deduplication |
| Sorted Set | ZADD, ZRANGE, ZRANGEBYSCORE | Priority queue implementation (alternative to Streams) |
| Stream | XADD, XREAD, XREADGROUP, XACK | Primary task queue (Stage 2) |

### Redis Streams
- **What:** A log-like data structure that records messages with auto-generated IDs. Supports consumer groups.
- **Why for VoiceGrid:** Unlike Lists, Streams support consumer groups (multiple consumers), pending entry lists (tasks in-flight), and message acknowledgement (XACK).
- **Key commands:**
  - `XADD stream_name * field1 value1 field2 value2` — Add message
  - `XREAD COUNT 10 STREAMS stream_name 0` — Read messages
  - `XREADGROUP GROUP group_name consumer_name COUNT 1 STREAMS stream_name >` — Consume as group
  - `XACK stream_name group_name message_id` — Acknowledge processing
- **Notes:** (empty)

### Consumer Groups
- **What:** Allows multiple consumers to read from the same stream without receiving the same message
- **Key concept:** Each message is delivered to exactly one consumer within a group
- **Pending Entry List (PEL):** Messages that have been delivered but not yet ACKed. On consumer crash, another consumer can reclaim them via XAUTOCLAIM or XCLAIM
- **Creating a group:** `XGROUP CREATE stream_name group_name $ MKSTREAM` ($ = only new messages)
- **VoiceGrid use:** One consumer group per priority level, or one group for all workers
- **Notes:** (empty)

### Distributed Locks (SETNX / Redlock)
- **Simple lock:** `SET lock_key worker_id NX PX 5000` — Atomic set if not exists with 5s TTL
- **Release lock:** Compare-and-delete using Lua script (ensure you only release your own lock)
- **Redlock:** Multi-Redis-node lock algorithm for high availability. Use when single-node SETNX is not sufficient.
- **VoiceGrid use:** Task claim lock to prevent two workers from processing the same task
- **Pitfall:** Always set a TTL. A lock without TTL will block forever if the holder crashes.
- **Notes:** (empty)

### Backpressure Patterns
- **Problem:** Producer publishes tasks faster than workers consume them. Queue grows unbounded.
- **Pattern 1:** Check XLEN before publishing; reject new tasks if queue exceeds threshold
- **Pattern 2:** Monitor consumer lag (delivery count vs. PEL size); alert or pause producers
- **Pattern 3:** Use Sorted Sets to track per-worker queue depth; load-balance routing accordingly
- **VoiceGrid relevance:** Backpressure is required for any system that makes latency guarantees under load
- **Notes:** (empty)

### Redis Persistence
- **RDB (Redis Database):** Point-in-time snapshots. Fast recovery, but may lose recent data after crash.
- **AOF (Append-Only File):** Log every write command. Slower writes, but near-zero data loss.
- **RDB + AOF:** Use both for production; RDB for fast restart, AOF for minimal data loss.
- **VoiceGrid relevance:** Task queue durability depends on persistence configuration. Without persistence, tasks in the queue are lost on Redis restart.
- **Notes:** (empty)

---

## Redis Command Quick Reference

### Streams (Stage 2 Key Commands)
```
# Add a task to stream
XADD voicegrid:tasks * task_id abc123 priority 5 type ASR payload "..."

# Read as consumer group
XREADGROUP GROUP workers worker-1 COUNT 1 BLOCK 5000 STREAMS voicegrid:tasks >

# Acknowledge task completion
XACK voicegrid:tasks workers <message-id>

# Check pending (un-acked) tasks
XPENDING voicegrid:tasks workers - + 10

# Re-claim abandoned tasks
XAUTOCLAIM voicegrid:tasks workers worker-2 60000 0-0
```

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
