# VoiceGrid Architecture Decision Log

This file records all significant architectural decisions made during the VoiceGrid project. Every decision that affects the technology stack, architecture, methodology, or scope of the project must be documented here before implementation begins.

## Format

Each entry follows this structure:

```
### D[NNN]: [Short Decision Title]
- **ID:** D[NNN]
- **Date:** YYYY-MM-DD
- **Status:** Accepted | Superseded | Deprecated | Proposed
- **Context:** Why was this decision needed? What problem does it solve?
- **Decision:** The actual choice made.
- **Rationale:** Why this choice over the alternatives?
- **Consequences:** What does this mean going forward (positive and negative)?
- **Alternatives Considered:** What else was evaluated?
```

Statuses:
- **Accepted:** Active decision currently in effect
- **Superseded:** Replaced by a newer decision (reference the new decision)
- **Deprecated:** No longer relevant (component removed)
- **Proposed:** Under discussion, not yet finalized

---

## Decision Log

---

### D001: Redis for Initial Distributed Coordination

- **ID:** D001
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
VoiceGrid requires a distributed task coordination layer that allows multiple worker nodes to consume inference tasks from a shared queue. The system needs durability (tasks should survive worker restarts), ordering guarantees (priority must be preserved across the network boundary), and the ability to support multiple consumer groups. The choice of coordination technology determines the complexity of Stages 2 and 3 and affects the learning objectives of the project.

**Decision:**
Use Redis Streams as the primary distributed task coordination layer for VoiceGrid inference task queuing, with Redis also serving as a distributed lock provider and cache layer.

**Rationale:**
Redis Streams natively support consumer groups, at-least-once delivery semantics, and pending entry lists (for crash recovery), which map directly to VoiceGrid's requirements. Redis is operationally simpler than Apache Kafka for a project at this scale, requires less configuration, and has well-supported client libraries in both Python (redis-py) and C++ (redis-plus-plus). Redis also serves as a cache and distributed lock provider (via SETNX / Redlock), allowing it to serve multiple roles and reduce the number of infrastructure dependencies.

**Consequences:**
- Positive: Simple deployment, well-understood semantics, serves multiple roles
- Positive: Consumer groups provide the at-least-once delivery guarantee required for task recovery
- Positive: Redis is a core learning objective; using it for coordination maximizes learning value
- Negative: Redis Streams do not natively support priority ordering. Priority must be implemented at the application level (multiple streams, one per priority level, or sorted sets with scores as priorities)
- Negative: Redis is single-threaded for most operations; very high throughput scenarios (>1M msg/sec) may require Redis Cluster

**Alternatives Considered:**
- **Apache Kafka:** More powerful for high-throughput streaming, but operationally complex (ZooKeeper or KRaft required), heavier dependency, and does not add learning value not already covered by Redis
- **RabbitMQ:** Supports priority queues natively (AMQP priority queue), but adds another system dependency and has less relevance to modern AI infrastructure stacks
- **In-memory queue (std::queue in C++):** Simple but not distributed — cannot span multiple worker nodes; fails on worker restart
- **PostgreSQL LISTEN/NOTIFY:** Too slow for high-throughput task queuing; not designed for this use case

---

### D002: C++ for the Scheduling Engine

- **ID:** D002
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
VoiceGrid's scheduling engine must handle high-throughput task sorting, priority queue operations, and coordination with the Redis layer. The project owner has identified C++ systems programming as a primary learning objective, and the scheduling engine is the most appropriate place to apply it. The language choice for this component affects performance, learning value, and complexity.

**Decision:**
Implement the VoiceGrid scheduling engine in C++17, using the standard library (STL) for core data structures and CMake for the build system.

**Rationale:**
C++ provides deterministic, low-overhead execution with direct control over memory layout and allocation — properties that matter for a hot-path scheduler that must make sub-millisecond decisions. Python's Global Interpreter Lock (GIL) prevents true multi-threaded parallelism for CPU-bound scheduling logic. Beyond performance, C++ is a primary technical learning objective: implementing the scheduler in C++ ensures the project owner gains hands-on experience with STL data structures, RAII, templates, and multithreading in the context of a real system.

**Consequences:**
- Positive: Demonstrates C++ proficiency in a portfolio-ready codebase
- Positive: Provides real-world context for DSA concepts (implementing heap vs. using one)
- Positive: C++ scheduler can be called from Python via pybind11 or subprocess if needed
- Negative: Higher development overhead than Python for the same feature
- Negative: Debugging C++ is harder than debugging Python (no debugger built into typical workflow)
- Negative: Integration with Python inference layer requires a well-defined interface (C ABI, protobuf, or REST)

**Alternatives Considered:**
- **Python:** Simpler, faster iteration, but GIL limits parallelism and does not serve the C++ learning objective
- **Rust:** Excellent performance and safety, but the project owner does not have Rust as a learning objective; adding Rust would dilute focus
- **Go:** Good concurrency model, simpler than C++, but less relevant to GPU inference systems and does not cover memory management learning objectives

---

### D003: CUDA for GPU Optimization

- **ID:** D003
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
GPU inference optimization is a primary technical objective of VoiceGrid. Stage 5 requires going below the PyTorch abstraction level to directly optimize GPU memory usage, kernel execution, and compute utilization. The choice of GPU programming framework determines what can be optimized and what is left to library defaults.

**Decision:**
Use CUDA (C++ for CUDA) as the GPU programming framework for VoiceGrid's optimization layer. Target CUDA Toolkit 12.0+, NVIDIA compute capability 7.0+ (Volta architecture or newer).

**Rationale:**
CUDA is the dominant GPU programming framework for NVIDIA hardware, which is the standard for AI inference workloads. CUDA provides direct access to GPU memory management (cudaMalloc, unified memory), thread hierarchy (threads, blocks, grids), and profiling tools (Nsight Systems, Nsight Compute). Custom CUDA kernels allow optimizations not possible through PyTorch's autograd layer — particularly fusion of multiple operations, custom memory layouts, and pipeline parallelism with CUDA Streams. CUDA expertise is itself a marketable skill in the AI infrastructure domain.

**Consequences:**
- Positive: Enables fine-grained GPU optimization not possible at the PyTorch level
- Positive: Nsight profiler provides the deepest insight into GPU bottlenecks
- Positive: CUDA expertise is directly applicable to roles at inference infrastructure companies
- Negative: CUDA is NVIDIA-specific; AMD GPUs are not supported (ROCm would be required for AMD)
- Negative: CUDA programming has a steep learning curve; kernel bugs are hard to debug
- Negative: Custom kernels require validation testing before performance claims are valid

**Alternatives Considered:**
- **Triton (OpenAI):** Python-based GPU programming, easier to learn, but less control than raw CUDA and fewer learning objectives for systems-level GPU programming
- **OpenCL:** Cross-platform, but significantly less tooling, fewer resources, and rarely used in modern AI infrastructure
- **ROCm (AMD):** Relevant only if AMD GPU hardware is available; NVIDIA is the standard for inference workloads

---

### D004: Real-Time Voice AI as the Application Domain

- **ID:** D004
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
VoiceGrid requires a specific application domain to provide concrete requirements for scheduling priority, latency targets, GPU utilization goals, and pipeline design. The domain must be technically demanding (to justify infrastructure complexity), relevant to the current AI market, and provide clear, measurable success criteria.

**Decision:**
Use real-time Voice AI (ASR → LLM → TTS pipeline) as the primary application domain driving VoiceGrid's design requirements.

**Rationale:**
Real-time Voice AI imposes the most demanding latency requirements in applied AI (sub-200ms end-to-end), which forces infrastructure decisions that would not be necessary in more latency-tolerant domains. The three-stage pipeline (ASR, LLM, TTS) provides natural boundaries for scheduling decisions, GPU optimization opportunities, and distributed coordination. Voice AI is a rapidly growing market (call centers, voice assistants, real-time translation) with substantial engineering job opportunities. The pipeline is complex enough to be portfolio-worthy but well-defined enough to be completable.

**Consequences:**
- Positive: Clear, measurable latency targets (< 200ms) drive all optimization work
- Positive: Well-established component models (Whisper, vLLM, TTS) with strong open-source support
- Positive: Directly relevant to current AI infrastructure job market
- Negative: Audio processing adds complexity to testing (need test audio files)
- Negative: Voice AI requires real GPU hardware to test latency targets (CPU-only development is limited)

**Alternatives Considered:**
- **Real-time code generation:** Similar LLM workload but no audio I/O, less differentiated
- **Image generation pipeline (Stable Diffusion):** GPU-intensive but different scheduling characteristics; TTS/ASR not applicable
- **General LLM API serving:** More generic, less differentiated from existing tools like vLLM itself

---

### D005: LangGraph for Conversational Orchestration

- **ID:** D005
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
Stage 6 requires adding multi-turn conversational intelligence to VoiceGrid. The orchestration layer must manage conversation state across turns, support conditional routing (different conversation paths based on intent), enable retrieval-augmented generation, and integrate with the existing Python pipeline. The choice of orchestration framework affects maintainability, debuggability, and the learning value of Stage 6.

**Decision:**
Use LangGraph for conversational flow management, with LangChain as the underlying abstraction for LLM calls, tool calling, and retrieval.

**Rationale:**
LangGraph models conversation as an explicit state graph with typed nodes and edges, which provides better debuggability and control than sequential chain-based approaches. Checkpointing is a built-in feature, enabling conversation state to be saved and restored across sessions (required for VoiceGrid's session continuity use case). LangGraph's conditional edge support maps naturally to intent-based conversation routing. It is an active, well-maintained project from the LangChain team with growing industry adoption.

**Consequences:**
- Positive: Explicit state graph makes conversation flow debuggable and testable
- Positive: Built-in checkpointing simplifies session state management
- Positive: Human-in-the-loop support allows hybrid AI/human conversation management
- Negative: LangGraph adds framework complexity; simple pipelines are harder to read than plain Python
- Negative: LangGraph abstractions can obscure what is happening at the LLM call level
- Negative: Framework updates may introduce breaking changes in a fast-moving library

**Alternatives Considered:**
- **Plain Python state machine:** Full control, no framework overhead, but requires implementing graph traversal, checkpointing, and streaming from scratch
- **LangChain LCEL (LangChain Expression Language):** Linear chains, less suitable for complex multi-path conversation routing
- **AutoGen (Microsoft):** Multi-agent framework, but heavier than needed for VoiceGrid's conversational use case

---

### D006: GitHub as the Source of Truth

- **ID:** D006
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
VoiceGrid is developed over an extended period using AI assistants. Session context is lost between sessions. The project needs a reliable, persistent, versioned source of truth that both human and AI can access to resume work accurately. The choice of project management approach affects how effectively AI assistants can continue work across sessions.

**Decision:**
Use GitHub as the authoritative source of truth for all VoiceGrid work. All code, documentation, architecture decisions, progress tracking, and task management live in the GitHub repository. PROJECT_STATUS.md and SESSION_HANDOFF.md are updated at the end of every session. GitHub Issues track all tasks and acceptance criteria.

**Rationale:**
GitHub provides version control (git), issue tracking (GitHub Issues), project boards (GitHub Projects), and a persistent, accessible record of all changes. AI assistants can read files from the repository, and the project owner can point AI assistants to specific files or issues to resume context quickly. Centralizing all information in one location (vs. Notion, JIRA, or other tools) reduces the risk of the source of truth becoming fragmented or stale. The project management approach must not add overhead that slows down engineering work.

**Consequences:**
- Positive: Version control provides audit trail for all changes
- Positive: AI assistants can read repository files directly
- Positive: GitHub Issues provide a structured task management system with completion criteria
- Negative: GitHub Issues require a remote repository (not available until Stage 0 remote creation)
- Negative: GitHub Projects requires manual configuration and ongoing maintenance
- Negative: If the project becomes private, sharing benchmarks requires additional configuration

**Alternatives Considered:**
- **Notion:** Rich documentation, but not version-controlled; AI access to Notion requires API setup
- **JIRA:** Enterprise-grade but heavyweight for a solo project
- **Linear:** Modern, but adds another tool dependency and less accessible to AI assistants
- **Plain markdown files only (no GitHub Issues):** Simpler but loses task tracking and completion verification

---

### D007: Benchmark-Driven Development Methodology

- **ID:** D007
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
VoiceGrid's primary engineering value is improving performance (latency, throughput, GPU utilization) of Voice AI inference. Without rigorous measurement, it is impossible to know whether optimizations actually improve performance or merely appear to. The AI infrastructure domain is full of performance claims that are not reproducible. VoiceGrid must set a higher standard.

**Decision:**
Adopt a benchmark-driven development methodology: no performance claim may be made without a documented, reproducible benchmark. Every optimization stage must document a baseline before implementing the optimization, measure the result after, and document the methodology, hardware, and raw results.

**Rationale:**
Benchmark-driven development prevents the most common failure modes in performance engineering: optimizing without measuring (which may have no effect or make things worse), and claiming improvements that cannot be reproduced. For a portfolio project, reproducible benchmarks are more valuable than unsubstantiated claims. AI assistants must not invent benchmark numbers — the methodology must produce real measurements from real runs.

**Consequences:**
- Positive: All performance claims are defensible and reproducible
- Positive: Establishes a professional standard that applies to future work
- Positive: Baseline measurement often reveals unexpected bottlenecks
- Negative: Benchmarking adds overhead to each stage (Stage 7 is dedicated to benchmarking)
- Negative: Running benchmarks requires real hardware (GPU, sufficient RAM), which may add cost

**Alternatives Considered:**
- **Claim-based development:** State performance targets and implement to meet them without intermediate measurement — too easy to fool yourself
- **Profiler-only approach:** Profile without establishing baselines — cannot quantify improvement magnitude

---

### D008: Blockchain Remains Optional (No Critical-Path Requirement)

- **ID:** D008
- **Date:** 2026-10-24
- **Status:** Accepted

**Context:**
The project owner has a background that includes blockchain technologies, and there is potential for blockchain-adjacent use cases in VoiceGrid (e.g., verifiable compute receipts, immutable audit trails for inference results, tokenized access control). However, adding blockchain as a required component without a clear, justified use case would expand scope significantly and potentially contradict VoiceGrid's core infrastructure mission.

**Decision:**
Blockchain technology is explicitly NOT on the critical path for any VoiceGrid stage. It remains optional and may be explored as a supplementary project or Stage 9+ extension if a specific, justified use case is identified. No VoiceGrid stage completion criteria require blockchain components. No AI assistant may add blockchain components to the critical path without explicit project owner approval.

**Rationale:**
VoiceGrid's mission is GPU-optimized, distributed inference infrastructure for Voice AI. Blockchain adds significant complexity (smart contract development, chain selection, gas costs, on-chain latency) that does not improve latency, throughput, or GPU utilization. A blockchain integration that adds latency to a sub-200ms pipeline would contradict the core mission. If a specific use case with clear value is identified (e.g., verifiable compute attestation for regulated industries), it may be evaluated at that time with a new DECISIONS.md entry.

**Consequences:**
- Positive: Maintains project focus on infrastructure performance objectives
- Positive: Avoids scope creep into a complex, unrelated engineering domain
- Positive: learning/BLOCKCHAIN.md exists for optional study without blocking progress
- Negative: Potential blockchain use cases (audit trails, verifiable compute) are deferred indefinitely
- Negative: If a blockchain component is later needed, it will require retrofitting

**Alternatives Considered:**
- **Make blockchain a Stage 10 extension:** Would add to the roadmap without being required; deferred instead to avoid implicit scope pressure
- **Require blockchain for audit trail use case in Stage 8 (cloud deployment):** No clear requirement for this at present; deferred until a specific customer or deployment scenario demands it

---

## How to Add a New Decision

When a significant architectural decision is made, add a new entry to this file immediately — before implementing the decision. Follow these steps:

1. **Assign the next available ID** (e.g., if D008 exists, the next is D009)

2. **Copy the template:**
```
### D[NNN]: [Short Decision Title]
- **ID:** D[NNN]
- **Date:** YYYY-MM-DD
- **Status:** Accepted
**Context:**
[2-4 sentences explaining why this decision was needed]
**Decision:**
[The actual choice made, in one sentence]
**Rationale:**
[Why this choice was made]
**Consequences:**
[Positive and negative consequences]
**Alternatives Considered:**
[What else was evaluated and why it was not chosen]
```

3. **Fill in all sections.** Do not leave any section blank. If alternatives were not formally considered, document why.

4. **Commit the DECISIONS.md update** before implementing the decision.

5. **If a decision supersedes an existing decision:** Update the old entry's Status to "Superseded by D[NNN]" and reference the new decision.

6. **If an AI assistant suggests a new technology or approach:** Do not implement it until a DECISIONS.md entry exists and the project owner has approved it.
