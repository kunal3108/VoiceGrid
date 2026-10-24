# VoiceGrid Hackathons

This directory contains all VoiceGrid hackathon projects. Each hackathon is a focused engineering sprint that contributes a concrete, tested component to VoiceGrid.

---

## What Is a VoiceGrid Hackathon?

In this project, a "hackathon" is not a 24-hour competition. It is a structured, focused engineering sprint with:

- A clear **problem statement** and **objective**
- Explicit **DSA and system design learning objectives**
- A detailed **implementation task list**
- Measurable **acceptance criteria**
- Required **tests and benchmarks**
- A clear **integration point** showing how the component fits into the larger VoiceGrid system

Hackathons are the primary vehicle for implementing VoiceGrid's core components. Each hackathon corresponds to a VoiceGrid roadmap stage.

---

## Completion Criteria

A hackathon is complete when ALL of the following are true:

1. **All acceptance criteria are met** — Every item in the hackathon's acceptance criteria checklist is satisfied
2. **Code is committed** — All implementation files are committed to the repository (no uncommitted changes)
3. **Tests pass** — All unit tests and integration tests for the hackathon pass with actual test runs
4. **Benchmarks documented** — Any benchmark results are written to docs/benchmarks/ with full methodology
5. **Documentation updated** — The hackathon README reflects the completed state
6. **Learning recorded** — Relevant learning journal entries updated with implemented topics, confidence levels, and last practiced date
7. **PROJECT_STATUS.md updated** — The project status file reflects completion
8. **GitHub Issue closed** — The corresponding GitHub Issue is closed with all criteria verified

A hackathon is NOT complete if:
- Tests were written but not run
- Benchmarks show numbers but methodology is not documented
- Learning journals were not updated
- Code is incomplete but "close enough"

---

## Hackathon Directory

| ID | Name | Stage | Technologies | Status | Estimated Hours |
|----|------|-------|-------------|--------|----------------|
| H01 | Priority Scheduler | Stage 1 | C++17, STL, CMake | Not Started | 60h |
| H02 | Redis Task Queue | Stage 2 | Redis Streams, C++, Python | Not Started | 70h |
| H03 | Concurrent Workers | Stage 3 | C++17, std::thread, mutexes | Not Started | 60h |
| H04 | GPU Inference Pipeline | Stage 4 | Python, Whisper, vLLM, TTS | Not Started | 80h |

Additional hackathons may be added as later stages are planned in detail.

---

## Hackathon Descriptions

### H01: Priority Scheduler
**Stage:** 1
**Technologies:** C++17, STL priority_queue, CMake, Google Test
**Problem:** VoiceGrid needs a scheduling engine that can rank inference tasks by priority. This hackathon builds the core priority queue-based scheduler in C++, establishing both the DSA foundation (heap data structure) and the C++ implementation pattern for the entire scheduling engine.
**Output:** A C++ library with FIFO and priority-based schedulers, unit tests, and documented benchmarks comparing the two approaches.

### H02: Redis Task Queue
**Stage:** 2
**Technologies:** Redis Streams, C++ redis-plus-plus client, Python redis-py, Consumer Groups
**Problem:** The Stage 1 scheduler runs in a single process. Stage 2 extends it to work across processes using Redis Streams. This hackathon builds the distributed task queue: C++ publishes tasks, Python workers consume them via Redis consumer groups.
**Output:** A distributed task queue with at-least-once delivery, dead letter queue, and documented throughput benchmark.

### H03: Concurrent Workers
**Stage:** 3
**Technologies:** C++17, std::thread, std::mutex, condition_variable, work stealing
**Problem:** A single-threaded worker cannot saturate a GPU. This hackathon builds a concurrent worker pool in C++ that processes multiple tasks in parallel, with work stealing for load balancing.
**Output:** A thread pool with work stealing, unit tests (ThreadSanitizer clean), and benchmark showing throughput vs. thread count.

### H04: GPU Inference Pipeline
**Stage:** 4
**Technologies:** Python, Whisper, vLLM or local LLM, TTS library, FastAPI
**Problem:** VoiceGrid needs to process real audio end-to-end. This hackathon assembles the complete ASR → LLM → TTS pipeline, integrates it with the Stage 3 worker pool, and establishes the baseline latency measurement for all subsequent optimization work.
**Output:** Working voice pipeline that accepts audio and returns audio, with per-stage latency logging and baseline benchmark.

---

## How to Start a New Hackathon

1. **Confirm the previous stage is complete** — Check PROJECT_STATUS.md and verify all acceptance criteria are met for the preceding stage.

2. **Create or update the hackathon directory** — Each hackathon already has a directory (H01-H04 are pre-created). The README.md in the directory is the authoritative spec.

3. **Create a GitHub Issue** — Create a GitHub Issue using the Engineering Task template. Reference the hackathon README in the issue. Link to all acceptance criteria as checklist items in the issue.

4. **Update PROJECT_STATUS.md** — Set "Current hackathon" to the new hackathon ID (e.g., H01).

5. **Update SESSION_HANDOFF.md** — Record the active hackathon and its GitHub Issue number.

6. **Begin implementation** — Follow the implementation task list in the hackathon README. Check off items as they are completed (with tests).

7. **End each session with documentation updates** — Follow the end-of-session protocol in CLAUDE.md or AGENTS.md.

---

## How Hackathons Connect to VoiceGrid

Each hackathon produces a component that integrates with later components:

```
H01 (Scheduler)
    │
    ▼ provides task ordering and priority logic
H02 (Redis Queue)
    │
    ▼ provides distributed coordination; H01 scheduler feeds the Redis queue
H03 (Worker Pool)
    │
    ▼ provides concurrent execution; workers consume from H02 Redis queue
H04 (Inference Pipeline)
    │
    ▼ provides the actual computation; pipeline runs inside H03 workers

Final system: H01 prioritizes → H02 distributes → H03 executes → H04 infers
```

This sequential dependency is why hackathons must be completed in order and why a hackathon's acceptance criteria must all be met before the next begins.

---

## Notes on AI-Assisted Hackathon Work

When working on a hackathon with an AI assistant:

- The AI should follow the Teaching Protocol from CLAUDE.md / AGENTS.md: explain concepts first, give the project owner the opportunity to implement, then review
- The AI should not auto-implement the entire hackathon — the learning objective is equally important as the deliverable
- The AI should help write tests and run them (not just write test stubs)
- The AI should update PROJECT_STATUS.md and SESSION_HANDOFF.md at the end of every session
- The AI should not mark a hackathon complete unless all acceptance criteria have been verified
