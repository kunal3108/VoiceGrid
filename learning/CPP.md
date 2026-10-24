# VoiceGrid Learning Journal: C++

Technology: C++17
Relevance to VoiceGrid: Scheduling engine, worker pool, high-performance coordination layer
Last updated: 2026-10-24

---

## Why C++ Matters to VoiceGrid

The VoiceGrid scheduling engine is implemented in C++17. This is a deliberate choice: C++ provides deterministic performance, direct memory management, true multithreading (no GIL), and the ability to express low-level optimizations that Python cannot. Understanding C++ at a systems level is one of the primary technical learning objectives of VoiceGrid.

Key applications:
- **Scheduling Engine (Stage 1):** Priority queue, FIFO queue, scheduler interface
- **Worker Pool (Stage 3):** Thread pool, work stealing, condition variables
- **Redis Integration (Stage 2):** C++ Redis client (redis-plus-plus) for publishing tasks
- **CUDA Integration (Stage 5):** Custom CUDA kernels written in CUDA C++

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| Syntax and Types | Not Started | — | — | No | — |
| STL Containers (vector, map, set, queue, deque) | Not Started | — | — | No | — |
| Iterators | Not Started | — | — | No | — |
| Pointers and References | Not Started | — | — | No | — |
| RAII and Smart Pointers | Not Started | — | — | No | — |
| Memory Management (heap/stack) | Not Started | — | — | No | — |
| Classes and OOP | Not Started | — | — | No | — |
| Templates | Not Started | — | — | No | — |
| Lambda Functions | Not Started | — | — | No | — |
| std::thread | Not Started | — | — | No | — |
| Mutexes and Locks | Not Started | — | — | No | — |
| Condition Variables | Not Started | — | — | No | — |
| Atomic Operations | Not Started | — | — | No | — |
| std::async and Futures | Not Started | — | — | No | — |
| Performance Profiling (gprof, perf) | Not Started | — | — | No | — |
| Compile Flags and Optimization | Not Started | — | — | No | — |
| CMake basics | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order to maximize VoiceGrid applicability:

1. **Syntax and Types** — Before anything else. Understand int, char, bool, float, double, const, auto.
2. **Pointers and References** — The most important C++-specific concept. Must understand before memory management.
3. **Memory Management (heap/stack)** — Required before RAII. Understand stack lifetime vs. heap lifetime.
4. **RAII and Smart Pointers** — Modern C++ memory management. Required before implementing any class.
5. **Classes and OOP** — Required for IScheduler interface and FIFOScheduler/PriorityScheduler classes.
6. **STL Containers** — Required for using std::priority_queue, std::queue, std::vector in the scheduler.
7. **Iterators** — Supporting concept; useful with STL containers.
8. **Templates** — Required for generic scheduler interface.
9. **Lambda Functions** — Required for threading callbacks and comparators.
10. **CMake basics** — Required to build Stage 1. Learn before writing any C++ code for H01.
11. **std::thread** — Required for Stage 3 (worker pool). Learn after completing H01.
12. **Mutexes and Locks** — Required for thread-safe task queue in Stage 3.
13. **Condition Variables** — Required for efficient worker thread sleep/wake pattern.
14. **Atomic Operations** — Required for lock-free counters and flags.
15. **std::async and Futures** — Alternative to std::thread for task-based parallelism.
16. **Performance Profiling** — Required for Stage 5 and Stage 7. Learn when profiling becomes relevant.
17. **Compile Flags and Optimization** — Required for benchmark runs. Understand -O0, -O1, -O2, -O3, -g.

---

## Topic Notes

### Syntax and Types
- **VoiceGrid relevance:** Every C++ file in VoiceGrid
- **Key types to know:** int, uint32_t, uint64_t, size_t, float, double, const, constexpr, auto
- **Notes:** (empty)

### Pointers and References
- **VoiceGrid relevance:** Task struct passed by reference to scheduler; scheduler stores pointers to tasks
- **Key concepts:** raw pointer, reference, pointer-to-pointer, nullptr, address-of (&), dereference (*)
- **Pitfalls:** Dangling pointers, null pointer dereference, pointer arithmetic
- **Notes:** (empty)

### RAII and Smart Pointers
- **VoiceGrid relevance:** Task objects managed with unique_ptr; worker pool uses shared_ptr for shared resources
- **Key types:** std::unique_ptr (sole ownership), std::shared_ptr (shared ownership), std::weak_ptr (non-owning)
- **RAII principle:** Resource acquisition is initialization — constructor acquires, destructor releases
- **Notes:** (empty)

### Memory Management
- **VoiceGrid relevance:** Scheduler allocates Task objects; worker pool allocates thread objects
- **Stack vs Heap:** Stack = automatic, fast, limited size. Heap = manual/smart-ptr, slow, large.
- **Key operations:** new/delete (avoid in modern C++), malloc/free (C-style, avoid), smart ptrs (use these)
- **Notes:** (empty)

### Classes and OOP
- **VoiceGrid relevance:** IScheduler abstract base class; FIFOScheduler, PriorityScheduler as derived classes
- **Key concepts:** Constructors, destructors, virtual functions, pure virtual (= 0), override keyword
- **Notes:** (empty)

### STL Containers
- **VoiceGrid relevance:**
  - std::priority_queue — PriorityScheduler
  - std::queue — FIFOScheduler
  - std::vector — task lists, benchmark result arrays
  - std::unordered_map — task lookup by ID
  - std::deque — work-stealing double-ended queue in Stage 3
- **Notes:** (empty)

### std::thread
- **VoiceGrid relevance:** Worker threads in Stage 3 WorkerPool
- **Key functions:** std::thread constructor, join(), detach(), hardware_concurrency()
- **Pitfall:** Forgetting to join threads before destruction causes std::terminate
- **Notes:** (empty)

### Mutexes and Locks
- **VoiceGrid relevance:** Thread-safe task queue in Stage 3; shared scheduler state
- **Key types:** std::mutex, std::lock_guard (RAII lock), std::unique_lock (movable lock for condition vars)
- **Pitfalls:** Deadlock (two threads each holding a lock the other needs), lock contention bottleneck
- **Notes:** (empty)

### Condition Variables
- **VoiceGrid relevance:** Worker threads wait for tasks without busy-looping (efficient sleep/wake)
- **Pattern:** producer notifies, consumer waits (with predicate to handle spurious wake-ups)
- **Why not busy-loop:** 100% CPU usage while doing nothing; condition variables allow CPU to yield
- **Notes:** (empty)

### CMake Basics
- **VoiceGrid relevance:** Build system for all C++ components
- **Key concepts:** CMakeLists.txt, add_executable, add_library, target_link_libraries, find_package
- **Target for H01:** CMakeLists.txt that builds scheduler binary and test binary
- **Notes:** (empty)

---

## Compiler Flags Reference

| Flag | Purpose | When to Use |
|------|---------|-------------|
| -std=c++17 | Enable C++17 | Always |
| -Wall -Wextra | Enable all warnings | Always |
| -g | Include debug symbols | Debug builds |
| -O0 | No optimization | Debug builds |
| -O2 | Standard optimization | Production builds |
| -O3 | Aggressive optimization | Benchmark runs |
| -fsanitize=address | AddressSanitizer | Finding memory bugs |
| -fsanitize=thread | ThreadSanitizer | Finding data races |
| -fsanitize=undefined | UndefinedBehaviorSanitizer | Finding UB |

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
