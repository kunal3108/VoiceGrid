# H01: Priority Scheduler

**Hackathon ID:** H01
**Stage:** Stage 1 — Task Scheduling Engine
**Status:** Not Started
**Technologies:** C++17, STL (priority_queue, queue), CMake, Google Test
**Estimated Effort:** 60 hours
**GitHub Issue:** (to be created after remote repository is configured)

---

## Problem Statement

VoiceGrid must schedule inference tasks with different priority levels. A real-time voice call requires an ASR task to be scheduled before a background transcription job — even if the background job arrived in the queue first. Without a priority-aware scheduler, VoiceGrid cannot meet its latency SLAs for high-priority workloads under concurrent load.

This hackathon builds the priority-based task scheduler in C++ that forms the core of VoiceGrid's scheduling engine. You will implement two scheduling strategies (FIFO and Priority), benchmark them at realistic task volumes, and understand empirically — not just theoretically — why priority scheduling matters for Voice AI infrastructure.

---

## Engineering Objective

Implement a production-quality C++ task scheduler with FIFO and priority queue variants. The scheduler must:
- Correctly order tasks by priority (higher priority tasks always scheduled first)
- Handle priority tie-breaking correctly (by deadline, then by arrival time)
- Be benchmarked at 1k, 10k, and 100k task volumes
- Have the trade-offs between scheduling strategies documented in the learning journal

---

## DSA Concepts

| Concept | Why It Matters Here |
|---------|-------------------|
| **Max-Heap** | The data structure underlying PriorityScheduler. Must implement from scratch before using std::priority_queue. |
| **FIFO Queue** | The data structure underlying FIFOScheduler. Baseline for benchmarks. |
| **Complexity Analysis** | O(log n) for heap insert/extract vs O(1) for queue enqueue/dequeue. What does this mean at 100k tasks? |
| **Abstract Data Types** | Priority Queue as an ADT vs Heap as an implementation. Why the distinction matters. |
| **Stable Ordering** | How to break ties correctly (priority → deadline → arrival time). |

---

## System Design Concepts

| Concept | Why It Matters Here |
|---------|-------------------|
| **Scheduling Algorithms** | FIFO vs. Priority vs. EDF (Earliest Deadline First). When is each appropriate? |
| **Task Queuing** | How tasks are represented, stored, and consumed. What metadata does a task need? |
| **Backpressure Basics** | What happens when tasks are enqueued faster than they are dequeued? |
| **Scheduler Interface Design** | Why use an abstract base class? What does the interface need to expose? |
| **Integration Design** | How this scheduler connects to Stage 2 (Redis) and Stage 3 (workers) |

---

## Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| C++17 | GCC 9+ or Clang 10+ | Implementation language |
| STL: priority_queue, queue | C++17 STL | Standard scheduler backends |
| CMake | 3.14+ | Build system |
| Google Test | 1.12+ | Unit test framework |

---

## Prerequisites

Before starting H01, you should have:
- [ ] C++ syntax basics: types, classes, functions, pointers
- [ ] Conceptual understanding of heaps and priority queues (learn/DSA.md — Heaps section)
- [ ] CMake basics: know what CMakeLists.txt does (learn/CPP.md — CMake section)
- [ ] Stage 0 acceptance criteria complete
- [ ] GitHub remote repository created

---

## Implementation Tasks

Work through these in order. Check off each task only when the implementation is complete and tested.

### Phase 1: Project Setup
- [ ] Create `hackathons/H01-priority-scheduler/` directory structure:
  ```
  H01-priority-scheduler/
  ├── CMakeLists.txt
  ├── include/
  │   ├── task.hpp           # Task struct definition
  │   ├── scheduler.hpp      # IScheduler abstract base class
  │   ├── fifo_scheduler.hpp # FIFOScheduler class
  │   └── priority_scheduler.hpp # PriorityScheduler class
  ├── src/
  │   ├── fifo_scheduler.cpp
  │   └── priority_scheduler.cpp
  ├── tests/
  │   ├── CMakeLists.txt
  │   ├── test_fifo_scheduler.cpp
  │   └── test_priority_scheduler.cpp
  └── benchmarks/
      ├── CMakeLists.txt
      └── bench_schedulers.cpp
  ```
- [ ] CMakeLists.txt compiles with `-Wall -Wextra -std=c++17` with no warnings

### Phase 2: Task Definition
- [ ] Implement `Task` struct in `include/task.hpp`:
  ```cpp
  struct Task {
      std::string id;         // Unique task identifier
      int priority;           // 0 = background, 5 = real-time
      std::string type;       // "ASR", "LLM", "TTS"
      std::string payload;    // Task data (JSON string for now)
      uint64_t arrival_time;  // Unix timestamp in milliseconds
      uint64_t deadline;      // Expected completion deadline in milliseconds
  };
  ```
- [ ] Task struct is copyable and movable
- [ ] Write unit test: create Task, verify all fields accessible

### Phase 3: Scheduler Interface
- [ ] Implement `IScheduler` abstract base class in `include/scheduler.hpp`:
  ```cpp
  class IScheduler {
  public:
      virtual ~IScheduler() = default;
      virtual void enqueue(Task task) = 0;
      virtual Task dequeue() = 0;
      virtual bool empty() const = 0;
      virtual size_t size() const = 0;
  };
  ```
- [ ] `dequeue()` must throw `std::runtime_error` (or return std::optional<Task>) when queue is empty — decide and document the choice

### Phase 4: FIFO Scheduler
- [ ] Implement `FIFOScheduler` in `include/fifo_scheduler.hpp` and `src/fifo_scheduler.cpp`
  - Use `std::queue<Task>` internally
  - `enqueue()`: O(1) push to back
  - `dequeue()`: O(1) pop from front
  - `empty()`: O(1)
  - `size()`: O(1)
- [ ] Write unit tests in `tests/test_fifo_scheduler.cpp`:
  - [ ] Empty queue returns empty
  - [ ] Single task enqueued and dequeued correctly
  - [ ] Tasks returned in FIFO order (10 tasks with different priorities — FIFO should return in arrival order, not priority order)
  - [ ] Size() correct after enqueue and dequeue
  - [ ] Dequeue on empty queue throws or returns empty optional

### Phase 5: Priority Scheduler (Core Implementation)
- [ ] Implement a max-heap from scratch (in a separate header or source file) BEFORE using std::priority_queue
  - This is a learning requirement: understand the data structure before using the STL version
  - Implement: `heapify_up`, `heapify_down`, `insert`, `extract_max`
  - Write tests for the raw heap implementation
- [ ] Implement `PriorityScheduler` in `include/priority_scheduler.hpp` and `src/priority_scheduler.cpp`
  - Use `std::priority_queue<Task, std::vector<Task>, TaskComparator>` internally
  - `TaskComparator`: sort by priority (descending), then by deadline (ascending), then by arrival_time (ascending)
  - `enqueue()`: O(log n)
  - `dequeue()`: O(log n)
  - `empty()`: O(1)
  - `size()`: O(1)
- [ ] Write unit tests in `tests/test_priority_scheduler.cpp`:
  - [ ] Empty queue returns empty
  - [ ] Single task enqueued and dequeued correctly
  - [ ] Two tasks with different priorities: higher priority returned first
  - [ ] Two tasks with same priority, different deadlines: earlier deadline returned first
  - [ ] Two tasks with same priority and deadline, different arrival times: earlier arrival returned first
  - [ ] 100 tasks with random priorities: all returned in correct priority order
  - [ ] Interleaved enqueue/dequeue: 50 enqueue, 10 dequeue, 50 more enqueue, drain — verify ordering throughout

### Phase 6: Deadline-Aware Scheduler (Optional Extension)
- [ ] Implement a deadline-aware variant that uses Earliest Deadline First (EDF) ordering within same-priority tasks
  - This may be the same as PriorityScheduler with a different comparator — document the design
  - Write tests demonstrating correct EDF behavior

### Phase 7: Benchmarks
- [ ] Implement benchmark harness in `benchmarks/bench_schedulers.cpp`
  - Measure: time to enqueue N tasks + time to dequeue all N tasks
  - Measure at N = 1,000 / 10,000 / 100,000 tasks
  - Run each benchmark 5 times; report min/max/mean
  - Compare FIFOScheduler vs. PriorityScheduler
- [ ] Compile with `-O2` for benchmark runs (not `-O0`)
- [ ] Document results in `docs/benchmarks/h01_scheduler_benchmark.md` with:
  - Hardware specification
  - Compiler and flags
  - Raw results (all 5 runs)
  - Summary table (mean values)
  - Analysis: what does the O(log n) vs O(1) difference actually look like in practice?

### Phase 8: Documentation and Integration
- [ ] Update `learning/DSA.md`: mark Heaps, Priority Queues, and Complexity Analysis as practiced
- [ ] Update `learning/CPP.md`: mark Classes/OOP, STL Containers, CMake basics as practiced
- [ ] Write integration note: how does this scheduler feed into H02 (Redis Task Queue)?
  - The Task struct from this hackathon will be serialized and published to Redis in H02
  - The IScheduler interface may be adapted to wrap Redis publishing in H02

---

## Acceptance Criteria

All of the following must be true for H01 to be considered complete:

- [ ] C++ project compiles cleanly with `-Wall -Wextra -std=c++17` (zero warnings, zero errors)
- [ ] `FIFOScheduler` correctly returns tasks in FIFO order under all test cases
- [ ] `PriorityScheduler` correctly returns highest-priority task first under all test cases
- [ ] Priority tie-breaking by deadline is correct
- [ ] Priority and deadline tie-breaking by arrival time is correct
- [ ] All unit tests pass with actual execution (not "I believe they pass")
- [ ] Minimum 20 test cases implemented and passing
- [ ] Benchmark run completed at 1k, 10k, 100k tasks
- [ ] Benchmark results documented in `docs/benchmarks/h01_scheduler_benchmark.md` with full methodology
- [ ] Time complexity analysis written in `learning/DSA.md` with O-notation for all operations
- [ ] Max-heap implemented from scratch (before using std::priority_queue) as a learning exercise
- [ ] GitHub Issue for H01 closed with all criteria verified

---

## Tests Required

| Test File | Test Cases |
|-----------|-----------|
| test_fifo_scheduler.cpp | Empty queue, single task, FIFO ordering, size tracking, empty dequeue behavior |
| test_priority_scheduler.cpp | Empty queue, single task, priority ordering, deadline tie-breaking, arrival time tie-breaking, interleaved operations, large queue (100 tasks) |
| (heap unit test) | Insert, extract-max, heapify correctness, size tracking |

---

## Expected Benchmark Metrics

> Note: These are expectations, not results. Actual measurements will differ based on hardware.

| N Tasks | FIFOScheduler | PriorityScheduler | Expected Ratio |
|---------|---------------|-------------------|----------------|
| 1,000 | ~1ms | ~2ms | ~2x |
| 10,000 | ~5ms | ~25ms | ~5x |
| 100,000 | ~50ms | ~400ms | ~8x |

The growing ratio reflects O(1) vs O(log n): at N=100k, log₂(100,000) ≈ 17, so O(log n) operations take approximately 17x longer per operation than O(1). The actual ratio will depend on constant factors and cache behavior.

Record actual results in `docs/benchmarks/h01_scheduler_benchmark.md`. Do not copy these estimates as results.

---

## Lessons Learned

(Empty — fill in after hackathon is complete)

- What surprised you about the implementation?
- What was harder than expected?
- What would you do differently?
- What questions do you still have?

---

## Integration: Connection to Stage 2 (Redis) and Stage 3 (Workers)

**H01 → H02 (Redis Task Queue):**
The `Task` struct defined in H01 will be serialized (JSON or Protocol Buffers) and published to a Redis Stream in H02. The `IScheduler` interface may be wrapped or extended so that `enqueue()` publishes directly to Redis. The priority ordering established by `PriorityScheduler` informs the Redis stream design: VoiceGrid will use multiple Redis streams (one per priority level) to maintain priority ordering across process boundaries.

**H01 → H03 (Concurrent Workers):**
The Stage 3 worker pool will use an internal queue (similar to `FIFOScheduler`) to buffer tasks after they are pulled from Redis. Understanding the trade-offs between FIFO and priority queuing (from H01) informs the internal queue design for the worker pool.

**H01 → Full System:**
The scheduling logic in H01 is the conceptual backbone of VoiceGrid's entire coordination layer. Every subsequent stage builds on the assumption that high-priority tasks are scheduled and executed before lower-priority tasks. Getting this right in H01 — with tests and benchmarks — is the foundation for everything that follows.
