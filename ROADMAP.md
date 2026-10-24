# VoiceGrid Roadmap

Version: 1.0
Date: 2026-10-24
Total Estimated Effort: 550 hours across 10 stages (Stage 0 through Stage 9)

---

## Roadmap Overview

| Stage | Name | Hours | Status |
|-------|------|-------|--------|
| 0 | Project Setup and Baseline | 10h | **Complete** |
| 1 | Task Scheduling Engine (C++, DSA) | 60h | Not Started |
| 2 | Distributed Task Execution (Redis, C++) | 70h | Not Started |
| 3 | Concurrent Worker Execution (C++, Multithreading) | 60h | Not Started |
| 4 | Working Voice AI Inference Pipeline (Python) | 80h | Not Started |
| 5 | GPU Inference Optimization (CUDA) | 90h | Not Started |
| 6 | Conversational Orchestration and RAG (LangGraph) | 70h | Not Started |
| 7 | Performance Benchmarking | 50h | Not Started |
| 8 | Cloud Deployment (GCP, Kubernetes) | 40h | Not Started |
| 9 | Open-Source Contributions | 20h | Not Started |
| | **Total** | **550h** | |

---

## Checkpoints

| Checkpoint | Cumulative Hours | Stages Complete | Milestone Description |
|-----------|-----------------|-----------------|----------------------|
| Checkpoint 1 | ~150h | Stages 0–2 (140h) | C++ scheduler built and benchmarked; Redis distributed task queue operational |
| Checkpoint 2 | ~300h | Stages 3–4 (280h cumulative) | Concurrent worker pool running; first working end-to-end Voice AI pipeline |
| Checkpoint 3 | ~550h | Stages 5–9 | GPU-optimized, cloud-deployed, benchmark-documented portfolio platform |

> **Note:** Checkpoint hours are planning targets across the whole project budget. Stage groupings are approximate — e.g., Stage 5 (90h) may overlap with or push Checkpoint 2 past 300h. Track actual hours in PROJECT_STATUS.md.

---

## Effort Budget

| Category | Estimated Hours | Percentage |
|----------|----------------|------------|
| C++ systems programming | ~190h | 35% |
| Python and pipeline work | ~100h | 18% |
| CUDA and GPU work | ~90h | 16% |
| Distributed systems (Redis) | ~70h | 13% |
| Benchmarking and profiling | ~50h | 9% |
| Cloud deployment | ~40h | 7% |
| Open-source | ~20h | 4% |
| Setup and documentation | ~10h | 2% |

---

---

## Stage 0: Project Setup and Baseline

**Status:** Complete
**Estimated Hours:** 10h

### Objective
Establish a complete, well-organized project foundation before writing any implementation code. This includes repository structure, documentation, AI assistant operating guidelines, and a clear definition of done for all subsequent stages. A solid Stage 0 prevents wasted effort from unclear requirements or missing context later.

### Prerequisites
- Git installed and configured
- GitHub account available
- Development environment (Linux or macOS) set up
- Python 3.11+ installed

### Engineering Deliverables
- [ ] Repository directory structure created
- [ ] Git repository initialized
- [ ] README.md with mission statement and architecture overview
- [ ] PROJECT_CHARTER.md with full scope definition
- [ ] ROADMAP.md with acceptance criteria for all 9 stages
- [ ] PROJECT_STATUS.md initialized
- [ ] DECISIONS.md with initial architecture decisions (D001-D008)
- [ ] SESSION_HANDOFF.md created
- [ ] CLAUDE.md and AGENTS.md created
- [ ] Learning journals initialized for all 10 technology areas
- [ ] GitHub remote repository created
- [ ] Initial commit pushed to GitHub
- [ ] GitHub Issues created for Stage 1 acceptance criteria
- [ ] GitHub Issue templates configured
- [ ] docs/SESSION_START_PROMPT.md and SESSION_END_PROMPT.md created
- [ ] hackathons/ directory structure created with H01-H04 directories

### Learning Objectives
- Understand the full VoiceGrid architecture before writing code
- Learn how to structure a multi-stage engineering project
- Practice writing engineering documentation

### DSA Concepts
- None (setup stage)

### System Design Concepts
- Repository structure patterns for large projects
- Documentation-driven development

### Estimated Hours
10h

### Acceptance Criteria
- [ ] All documentation files created with complete, non-placeholder content
- [ ] GitHub remote repository exists and is accessible
- [ ] Initial commit pushed to GitHub
- [ ] GitHub Issues configured with at least Stage 1 acceptance criteria as issues
- [ ] AI assistant can read all documentation and correctly state the current project stage and active task

### Dependencies
- None (first stage)

---

## Stage 1: Task Scheduling Engine (C++, DSA)

**Status:** Not Started
**Estimated Hours:** 60h
**Hackathon:** H01-priority-scheduler

### Objective
Build a production-quality task scheduler in C++ that forms the core scheduling engine of VoiceGrid. This stage implements the fundamental data structures (priority queues, FIFO queues) and scheduling algorithms that will later coordinate real inference tasks. The goal is to understand scheduling trade-offs empirically, not just theoretically.

### Prerequisites
- Stage 0 complete (all acceptance criteria met)
- GitHub remote repository exists
- C++17 compiler installed (g++ 9+ or clang++ 10+)
- CMake 3.14+ installed
- Basic C++ syntax familiarity

### Engineering Deliverables
- [ ] C++ project scaffold with CMake build system (hackathons/H01-priority-scheduler/)
- [ ] Task struct: id, priority, type (ASR/LLM/TTS), payload, arrival_time, deadline
- [ ] Abstract IScheduler interface (pure virtual base class)
- [ ] FIFOScheduler implementation (std::queue)
- [ ] PriorityScheduler implementation (std::priority_queue / custom max-heap)
- [ ] Deadline-Aware Scheduler implementation (secondary sort on deadline)
- [ ] Unit tests for all three schedulers (Google Test or custom test harness)
- [ ] Benchmark harness: measure enqueue/dequeue throughput at 1k, 10k, 100k tasks
- [ ] Results documented in docs/benchmarks/ and learning/DSA.md
- [ ] Integration documented: how this connects to Stage 2 (Redis) and Stage 3 (workers)

### Learning Objectives
- Implement a max-heap from scratch before using std::priority_queue
- Understand the difference between FIFO scheduling and priority scheduling in terms of both correctness and performance
- Learn CMake build system for C++ projects
- Understand abstract base classes and polymorphism in C++

### DSA Concepts
- Max-heap and min-heap: structure, insert (O log n), extract-max (O log n), heapify
- Priority queue as an abstract data type vs. heap as a concrete implementation
- FIFO queue: O(1) enqueue and dequeue
- Complexity analysis: when does constant factor matter over Big O?
- Stable vs. unstable priority ordering (tie-breaking)

### System Design Concepts
- Scheduling algorithms: FIFO, Priority, Round-Robin, Earliest Deadline First (EDF)
- Task prioritization in real systems: how do real-time voice systems assign priority?
- Backpressure: what happens when the queue grows faster than it is drained?
- Scheduler interface design: why abstract base class instead of concrete class?

### Estimated Hours
60h (includes learning, implementation, testing, and benchmarking)

### Acceptance Criteria
- [ ] C++ project compiles cleanly with `-Wall -Wextra` (no warnings)
- [ ] FIFOScheduler correctly returns tasks in FIFO order under all test cases
- [ ] PriorityScheduler correctly returns highest-priority task first under all test cases
- [ ] Deadline-Aware Scheduler correctly breaks priority ties by earliest deadline
- [ ] Unit tests pass: minimum 20 test cases covering normal operation, empty queue, single task, equal priorities, and concurrent enqueue (if implemented)
- [ ] Benchmark run completed and results documented: enqueue/dequeue at 1k, 10k, 100k tasks
- [ ] Time complexity analysis written in learning/DSA.md with O-notation for all operations
- [ ] GitHub Issue for Stage 1 closed with all criteria verified

### Dependencies
- Stage 0 acceptance criteria complete
- C++17 environment configured

---

## Stage 2: Distributed Task Execution (Redis, C++)

**Status:** Not Started
**Estimated Hours:** 70h
**Hackathon:** H02-redis-task-queue

### Objective
Extend the Stage 1 scheduler to operate in a distributed context using Redis Streams as the coordination layer. Tasks enqueued by the C++ scheduler are published to Redis and consumed by worker processes, enabling the first distributed version of the VoiceGrid task pipeline. This stage makes the scheduler real: multiple processes can now coordinate task execution across a network boundary.

### Prerequisites
- Stage 1 acceptance criteria complete
- Redis 7.0+ installed locally (or Docker)
- Python 3.11+ with redis-py installed
- Familiarity with Redis data structures

### Engineering Deliverables
- [ ] Redis Streams task queue design documented (message format, consumer group design)
- [ ] C++ Redis publisher: publishes Task objects as Redis Stream entries
- [ ] Python Redis consumer: reads tasks from Redis Stream, simulates execution
- [ ] Consumer group configuration for at-least-once delivery semantics
- [ ] Distributed lock implementation using Redis SETNX for task claim
- [ ] Dead letter queue (DLQ) for failed task handling
- [ ] Retry logic with exponential backoff for failed tasks
- [ ] Unit tests for publisher, consumer, and DLQ logic
- [ ] Integration test: end-to-end task flow from C++ publisher to Python consumer
- [ ] Benchmark: task throughput at 100, 1k, 10k tasks/second with Redis

### Learning Objectives
- Understand Redis Streams vs. Pub/Sub vs. List-based queues
- Implement consumer groups and understand why they matter for distributed coordination
- Learn distributed locking patterns (SETNX, Redlock)
- Understand at-least-once delivery and how to make processing idempotent

### DSA Concepts
- Queues as the abstraction underlying distributed task systems
- Ordered streams vs. unordered sets in Redis
- Consistent hashing (conceptual: how Redis Cluster distributes keys)

### System Design Concepts
- Message queue design: ordering guarantees, delivery semantics, consumer groups
- Distributed coordination without a central lock server
- Dead letter queues and retry patterns
- Backpressure: XLEN check, consumer lag monitoring
- Idempotency: why workers must be safe to re-execute a task

### Estimated Hours
70h

### Acceptance Criteria
- [ ] Tasks published by C++ producer appear in Redis Stream within 5ms (local Redis)
- [ ] Python consumer processes all published tasks exactly once under normal conditions
- [ ] Consumer group correctly handles consumer restart (pending entry re-delivery)
- [ ] Dead letter queue captures tasks that fail after max retries
- [ ] Distributed lock prevents two consumers from claiming the same task simultaneously
- [ ] Integration test passes: 1000 tasks published and consumed end-to-end without loss
- [ ] Benchmark documented: throughput at 100, 1k, 10k tasks/second
- [ ] learning/REDIS.md updated with consumer groups and distributed lock topics marked as practiced

### Dependencies
- Stage 1 complete
- Redis available (local or Docker)

---

## Stage 3: Concurrent Worker Execution (C++, Multithreading)

**Status:** Not Started
**Estimated Hours:** 60h
**Hackathon:** H03-concurrent-workers

### Objective
Replace the single-threaded consumer with a concurrent worker pool implemented in C++. Workers pull tasks from the Redis queue (from Stage 2) and execute them concurrently using C++ threads. This stage implements the core parallelism that allows VoiceGrid to serve multiple inference requests simultaneously. The goal is to understand thread safety, synchronization, and the performance characteristics of concurrent execution.

### Prerequisites
- Stage 2 acceptance criteria complete
- Familiarity with C++ std::thread
- Understanding of mutex and condition variable patterns

### Engineering Deliverables
- [ ] WorkerPool class: manages N concurrent worker threads
- [ ] Task executor interface: pluggable execution backend (simulated and real)
- [ ] Thread-safe task queue: internal queue between Redis consumer and worker threads
- [ ] Worker thread: pulls from internal queue, executes task, reports result
- [ ] Graceful shutdown: workers drain queue and exit cleanly on signal
- [ ] Work stealing implementation: idle workers can steal tasks from overloaded workers
- [ ] Thread pool benchmarks: throughput vs. thread count (1, 2, 4, 8, 16 threads)
- [ ] Deadlock analysis: identify and document all potential deadlock scenarios
- [ ] Unit tests: concurrent access, shutdown, work stealing, error handling
- [ ] Integration test: WorkerPool + Redis queue end-to-end

### Learning Objectives
- Implement a thread pool from scratch before using a library
- Understand deadlock conditions and how to prevent them
- Learn condition variables and why they are preferred over busy-waiting
- Understand work stealing and when it improves performance

### DSA Concepts
- Work-stealing deque (ArrayDeque / lock-free deque conceptually)
- Thread pool as a producer-consumer pattern
- Lock-free data structures: why they matter and the trade-offs

### System Design Concepts
- Thread pool design: fixed size vs. elastic pool
- Backpressure between Redis consumer and worker threads
- Graceful shutdown patterns for long-running services
- Worker health monitoring and heartbeat patterns
- Error isolation: one failing worker must not crash the pool

### Estimated Hours
60h

### Acceptance Criteria
- [ ] WorkerPool correctly runs N tasks concurrently where N = thread count
- [ ] All unit tests pass with ThreadSanitizer (tsan) enabled — no data races detected
- [ ] Graceful shutdown completes within 5 seconds: all in-flight tasks finish, no tasks lost
- [ ] Work stealing demonstrates measurable throughput improvement over no-stealing baseline (documented benchmark)
- [ ] Benchmark completed: throughput vs. thread count documented in docs/benchmarks/
- [ ] learning/CPP.md topics: std::thread, mutexes, condition variables, atomic operations marked as practiced

### Dependencies
- Stage 2 complete
- C++ ThreadSanitizer available (clang++ with -fsanitize=thread)

---

## Stage 4: Working Voice AI Inference Pipeline (Python)

**Status:** Not Started
**Estimated Hours:** 80h

### Objective
Build a complete, working end-to-end Voice AI inference pipeline in Python. This stage delivers the first functional version of VoiceGrid: audio in, transcription via Whisper (ASR), response generation via a local LLM, and text-to-speech output. The pipeline integrates with the C++ worker pool from Stage 3. By the end of this stage, VoiceGrid processes real voice input and produces real audio output — even if latency targets are not yet met.

### Prerequisites
- Stage 3 acceptance criteria complete
- Python 3.11+ with torch, transformers, openai-whisper installed
- At minimum one GPU available (can use CPU fallback for development)
- TTS library available (Coqui TTS, pyttsx3, or similar)

### Engineering Deliverables
- [ ] FastAPI gateway: accepts audio upload (WAV/MP3), returns audio response
- [ ] ASR stage: Whisper integration (small or medium model), returns transcript
- [ ] LLM stage: local model inference via transformers or vLLM (e.g., Mistral-7B or similar)
- [ ] TTS stage: text-to-speech synthesis, returns WAV audio
- [ ] Pipeline orchestrator: chains ASR → LLM → TTS with per-stage timing
- [ ] Integration with Stage 3 WorkerPool: tasks dispatched through C++ worker pool
- [ ] End-to-end latency measurement: per stage and total (logged for every request)
- [ ] Baseline benchmark: latency at 1, 5, 10 concurrent sessions (GPU and CPU)
- [ ] Unit tests for each pipeline stage
- [ ] Integration test: full audio-in to audio-out test with known input/output pair

### Learning Objectives
- Understand ASR model architecture and inference characteristics (Whisper)
- Understand LLM inference basics: tokenization, generation, sampling
- Understand TTS synthesis pipeline
- Learn FastAPI async patterns for handling concurrent requests
- Learn how to measure per-stage latency in a pipeline

### DSA Concepts
- Pipeline as a sequence of data transformations
- Buffering and streaming: when to buffer audio vs. process as stream

### System Design Concepts
- API gateway design for inference workloads
- Pipeline stage interfaces: what data flows between stages?
- Latency budget allocation: how to split 200ms across 3 stages?
- Baseline-first development: measure before optimizing
- Graceful error handling in pipelines: what happens if LLM fails?

### Estimated Hours
80h

### Acceptance Criteria
- [ ] End-to-end pipeline accepts real audio input and produces real audio output
- [ ] ASR stage correctly transcribes a set of test utterances (> 90% word accuracy on clean audio)
- [ ] LLM stage produces coherent responses to transcribed prompts
- [ ] TTS stage produces intelligible audio from LLM output
- [ ] Per-stage and total latency logged for every request
- [ ] Baseline benchmark documented: P50 and P99 latency at 1 and 5 concurrent sessions
- [ ] Integration test passes: known audio input produces expected category of response
- [ ] learning/GPU_INFERENCE.md topics: LLM inference fundamentals, ASR optimization, TTS optimization marked as studied

### Dependencies
- Stage 3 complete
- At least one GPU available or CPU fallback confirmed working

---

## Stage 5: GPU Inference Optimization (CUDA)

**Status:** Not Started
**Estimated Hours:** 90h

### Objective
Optimize the Stage 4 inference pipeline using CUDA-level GPU programming techniques. This stage moves beyond using PyTorch as a black box and instead applies direct CUDA optimization: custom kernels for pipeline bottlenecks, memory optimization, quantization, and profiling-guided improvements. Every optimization must be measured against the Stage 4 baseline. This stage is the most technically demanding in the project.

### Prerequisites
- Stage 4 complete with documented baseline benchmark
- CUDA Toolkit 12.0+ installed
- NVIDIA Nsight Systems and Nsight Compute available
- Understanding of CUDA thread hierarchy and memory hierarchy (learning/CUDA.md basics)

### Engineering Deliverables
- [ ] CUDA profiling baseline: Nsight profile of Stage 4 pipeline, identifying top bottlenecks
- [ ] Custom CUDA kernel: at least one bottleneck replaced with optimized custom kernel
- [ ] Memory optimization: eliminate unnecessary host↔device transfers
- [ ] Quantization implementation: INT8 quantization of LLM, before/after benchmark
- [ ] KV-Cache optimization: implement or configure KV-cache for LLM serving
- [ ] Continuous batching: replace static batching with dynamic batching
- [ ] CUDA Streams: overlap computation and memory transfer with CUDA streams
- [ ] GPU utilization improvement: measured increase over Stage 4 baseline
- [ ] Updated benchmark: P50/P95/P99 latency comparison Stage 4 vs. Stage 5
- [ ] CUDA kernel unit tests: validate kernel output correctness before measuring performance

### Learning Objectives
- Learn to use Nsight Systems and Nsight Compute to identify GPU bottlenecks
- Implement a non-trivial CUDA kernel (reduction, matrix operation, or similar)
- Understand memory coalescing and how to optimize global memory access patterns
- Understand quantization trade-offs: precision vs. throughput vs. model quality
- Learn continuous batching and its impact on GPU utilization

### DSA Concepts
- Parallel reduction: from O(n) sequential to O(log n) parallel
- Matrix multiplication parallelism: tiling, shared memory
- Warp-level primitives: __shfl_down_sync for efficient reductions

### System Design Concepts
- Profiling-driven optimization: identify bottleneck before writing code
- Latency vs. throughput trade-offs in GPU inference
- Memory bandwidth as a primary constraint (not compute) for many inference operations
- Quantization as a system-level decision: calibration, accuracy validation, deployment

### Estimated Hours
90h (CUDA is the most technically challenging component)

### Acceptance Criteria
- [ ] Nsight profile analysis documented identifying top 3 bottlenecks in Stage 4 pipeline
- [ ] At least one custom CUDA kernel implemented and validated for correctness
- [ ] Quantization reduces LLM inference latency by at least 20% with less than 5% degradation in response quality (measured on test set)
- [ ] GPU utilization increased from Stage 4 baseline (specific number documented)
- [ ] P50 end-to-end latency improved from Stage 4 baseline (specific number documented)
- [ ] All CUDA kernel unit tests pass
- [ ] Optimization methodology documented in docs/benchmarks/ (profile first, measure after)
- [ ] learning/CUDA.md: all core topics marked as practiced with confidence level updated

### Dependencies
- Stage 4 complete with documented latency baseline
- CUDA Toolkit available
- NVIDIA GPU with compute capability 7.0+ (Volta or newer)

---

## Stage 6: Conversational Orchestration and RAG (LangGraph, LangChain)

**Status:** Not Started
**Estimated Hours:** 70h

### Objective
Add multi-turn conversational intelligence to VoiceGrid using LangGraph for orchestration and a vector database for Retrieval-Augmented Generation (RAG). This stage transforms VoiceGrid from a single-turn voice pipeline into a stateful conversational system that can maintain context across turns, retrieve relevant information, and use external tools. LangGraph's graph-based state machine manages conversation flow.

### Prerequisites
- Stage 4 complete (working voice pipeline)
- LangGraph, LangChain installed
- Vector database available (Pinecone, Weaviate, or local Chroma/FAISS)
- Understanding of embeddings and semantic similarity

### Engineering Deliverables
- [ ] LangGraph conversation state graph: nodes for ASR, retrieval, LLM, TTS, error handling
- [ ] Multi-turn conversation memory: maintain context across at least 10 turns
- [ ] Vector database integration: embed and index a test corpus
- [ ] RAG retrieval stage: retrieve relevant context before LLM generation
- [ ] Tool calling: LLM can invoke at least one external tool (e.g., get current time, query database)
- [ ] Conditional edges: different conversation paths based on intent classification
- [ ] Checkpointing: save and restore conversation state across sessions
- [ ] Human-in-the-loop: support for injecting human messages mid-conversation
- [ ] Integration with GPU-optimized pipeline from Stage 5
- [ ] End-to-end test: multi-turn voice conversation with RAG retrieval

### Learning Objectives
- Understand LangGraph's state machine model vs. sequential chains
- Learn vector database fundamentals: embedding, indexing, similarity search
- Understand RAG pipeline design and retrieval quality measurement
- Learn checkpointing patterns for stateful AI systems

### DSA Concepts
- Graph traversal in the context of conversation state machines
- Approximate nearest neighbor (ANN) search in vector databases (HNSW, IVF)
- Embedding space geometry: cosine similarity, dot product

### System Design Concepts
- State machine design for conversational AI
- Retrieval-Augmented Generation architecture decisions
- Conversation context window management
- Multi-agent patterns with LangGraph
- Latency impact of retrieval: measuring RAG overhead

### Estimated Hours
70h

### Acceptance Criteria
- [ ] Multi-turn voice conversation maintains correct context across 10+ turns
- [ ] RAG retrieval returns relevant documents for test queries (measured by recall@5)
- [ ] Tool calling executes correctly and LLM uses tool results in response
- [ ] Conversation state is saved and restored correctly after simulated session interruption
- [ ] Conditional conversation routing works correctly for at least 3 distinct intents
- [ ] End-to-end latency with RAG documented (overhead vs. no-RAG baseline)
- [ ] learning/LANGGRAPH.md: all topics marked as practiced

### Dependencies
- Stage 4 complete
- Vector database accessible

---

## Stage 7: Performance Benchmarking

**Status:** Not Started
**Estimated Hours:** 50h

### Objective
Conduct a comprehensive, rigorous benchmarking exercise across the entire VoiceGrid stack. This stage is not about writing new features — it is about measuring what exists, documenting results with full methodology, and identifying the remaining gaps between current performance and design targets. The output is a complete benchmark report suitable for a portfolio.

### Prerequisites
- Stages 5 and 6 complete
- All pipeline components working end-to-end
- Benchmark hardware documented

### Engineering Deliverables
- [ ] Benchmark methodology document: workload description, hardware, software versions, measurement tools
- [ ] End-to-end pipeline latency benchmark: P50, P95, P99 at 1, 5, 10, 25, 50 concurrent sessions
- [ ] GPU utilization benchmark: utilization under varying concurrent loads
- [ ] Scheduler benchmark: task throughput and priority correctness under load
- [ ] Redis coordination benchmark: task queue throughput and consumer lag under load
- [ ] Memory utilization benchmark: GPU memory and host memory under peak load
- [ ] Comparison tables: Stage 4 baseline vs. Stage 5 optimized vs. Stage 6 with RAG
- [ ] Bottleneck analysis: top 3 remaining performance constraints identified
- [ ] Recommendations document: what further work would close the gap to targets

### Learning Objectives
- Learn how to design a fair, reproducible benchmark
- Understand the difference between microbenchmarks and system benchmarks
- Learn to read and interpret profiler output
- Understand how to present performance results honestly

### DSA Concepts
- Statistical significance: why single-run measurements are unreliable
- Percentile calculations: P50 vs. P99 and what they reveal

### System Design Concepts
- Benchmarking methodology: avoiding common pitfalls (warm-up, noise, outliers)
- Load generation design: how to generate realistic voice workload
- Performance regression testing: how to detect regressions in CI

### Estimated Hours
50h

### Acceptance Criteria
- [ ] Full benchmark report written to docs/benchmarks/benchmark_report.md
- [ ] All performance claims backed by documented methodology and raw results
- [ ] P50 and P99 latency measured at minimum 5 concurrency levels
- [ ] GPU utilization curve plotted over increasing load
- [ ] Bottleneck analysis identifies at least 3 specific, actionable constraints
- [ ] Results are reproducible: another engineer can run the benchmark with the provided instructions

### Dependencies
- Stages 5 and 6 complete
- Stable pipeline (no known critical bugs)

---

## Stage 8: Cloud Deployment (GCP, Kubernetes)

**Status:** Not Started
**Estimated Hours:** 40h

### Objective
Deploy VoiceGrid to Google Cloud Platform using Kubernetes (GKE) with GPU node pools. This stage moves VoiceGrid from a local development environment to a production-grade cloud deployment. The deployment must support auto-scaling, GPU node management, health checks, and monitoring. By the end, VoiceGrid runs in the cloud and accepts real traffic.

### Prerequisites
- Stage 7 complete (benchmarks documented)
- GCP account with billing enabled
- gcloud CLI and kubectl installed
- Docker installed
- VoiceGrid passes all tests locally

### Engineering Deliverables
- [ ] Dockerfiles for all VoiceGrid components (gateway, workers, inference nodes)
- [ ] Docker Compose for local multi-container testing
- [ ] Kubernetes manifests: Deployments, Services, ConfigMaps, Secrets
- [ ] GKE cluster configuration: CPU node pool + GPU node pool (T4 or L4)
- [ ] Horizontal Pod Autoscaler (HPA) for worker pods
- [ ] Health checks: liveness and readiness probes for all services
- [ ] Monitoring: Prometheus + Grafana dashboards for latency, throughput, GPU utilization
- [ ] Secret management: no credentials in code or manifests
- [ ] End-to-end smoke test: audio request processed in GKE cluster

### Learning Objectives
- Learn Kubernetes deployment patterns for stateful and stateless services
- Understand GPU node scheduling in Kubernetes
- Learn container image optimization for GPU workloads
- Understand Kubernetes resource requests and limits for GPU allocation

### DSA Concepts
- Consistent hashing for Kubernetes service routing (conceptual)

### System Design Concepts
- Container orchestration architecture
- GPU resource scheduling in Kubernetes
- Auto-scaling policies: when to scale up vs. out
- Secrets management in Kubernetes
- Observability stack: metrics, logs, traces in cloud deployment

### Estimated Hours
40h

### Acceptance Criteria
- [ ] All services running in GKE cluster without errors
- [ ] Audio request processed end-to-end in cloud deployment
- [ ] HPA scales worker pods up under load and down after load subsides
- [ ] Monitoring dashboard shows latency, throughput, and GPU utilization in real time
- [ ] No credentials committed to repository
- [ ] Cost estimate documented for running VoiceGrid in production at target load

### Dependencies
- Stage 7 complete
- GCP project with billing enabled and GPU quota approved
- Docker Hub or GCP Artifact Registry configured

---

## Stage 9: Open-Source Contributions

**Status:** Not Started
**Estimated Hours:** 20h

### Objective
Contribute improvements back to the open-source projects VoiceGrid depends on. This stage serves two purposes: improving the ecosystem that VoiceGrid uses, and demonstrating the ability to work within large, established codebases. Target projects include vLLM, SGLang, LangGraph, redis-py, or CUDA sample libraries.

### Prerequisites
- All stages 1-8 complete
- At least one identified improvement, bug fix, or documentation contribution
- Familiarity with the contribution guidelines of the target project

### Engineering Deliverables
- [ ] Identify at least one meaningful contribution opportunity (bug fix, performance improvement, documentation, or new feature)
- [ ] Fork target repository and set up development environment
- [ ] Implement the contribution following the target project's contribution guidelines
- [ ] Write tests for the contribution
- [ ] Submit pull request with clear description referencing the issue
- [ ] Address reviewer feedback until PR is merged or officially reviewed
- [ ] Document contribution in README.md and portfolio materials

### Learning Objectives
- Learn how to contribute to large open-source projects
- Understand code review dynamics and responding to feedback
- Practice writing clear, professional PR descriptions
- Understand the social and technical norms of open-source contribution

### DSA Concepts
- N/A — depends on the specific contribution

### System Design Concepts
- Open-source contribution workflow: fork, branch, PR, review
- How large projects manage backward compatibility

### Estimated Hours
20h

### Acceptance Criteria
- [ ] Pull request submitted to a recognized open-source project
- [ ] PR addresses a real issue or improvement (not trivial documentation typo)
- [ ] PR includes tests
- [ ] PR reviewed by project maintainers (merged or official review received)
- [ ] Contribution documented in portfolio materials

### Dependencies
- Stages 1-8 complete
- Contribution opportunity identified and confirmed (not all PRs get merged — this is acceptable)
