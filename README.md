# VoiceGrid

**GPU-Optimized Distributed Inference Infrastructure for Real-Time Voice AI**

[![Status](https://img.shields.io/badge/status-Stage%201%20In%20Progress-yellow)](PROJECT_STATUS.md)
[![Stage](https://img.shields.io/badge/stage-1%20of%209-blue)](ROADMAP.md)
[![License](https://img.shields.io/badge/license-MIT-green)](#license)

---

## Mission

VoiceGrid is a GPU-optimized, distributed inference infrastructure platform designed to improve the latency, throughput, scalability, and reliability of real-time Voice AI workloads.

---

## Architecture Overview

VoiceGrid implements a full end-to-end voice AI inference pipeline with production-grade infrastructure:

```
                         VoiceGrid Inference Pipeline
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   Audio Input                                                       │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────┐    ┌──────────────────────────────────────────────┐   │
│  │   ASR   │    │             Scheduling Engine (C++)           │   │
│  │(Whisper)│───▶│  Priority Queue │ FIFO │ Deadline Scheduling  │   │
│  └─────────┘    └──────────────────────┬───────────────────────┘   │
│                                        │                            │
│                                        ▼                            │
│                        ┌──────────────────────────┐                │
│                        │    Redis Task Coordinator │                │
│                        │  (Distributed Work Queue) │                │
│                        └──────────┬───────────────┘                │
│                                   │                                 │
│               ┌───────────────────┼───────────────────┐            │
│               ▼                   ▼                   ▼            │
│         ┌──────────┐       ┌──────────┐        ┌──────────┐        │
│         │  Worker  │       │  Worker  │        │  Worker  │        │
│         │  Node 1  │       │  Node 2  │        │  Node N  │        │
│         └────┬─────┘       └────┬─────┘        └────┬─────┘        │
│              │                  │                    │              │
│              └──────────────────┼────────────────────┘             │
│                                 ▼                                   │
│                     ┌───────────────────────┐                      │
│                     │    LLM Inference       │                      │
│                     │  (vLLM / SGLang)       │                      │
│                     │  GPU-Optimized CUDA    │                      │
│                     └───────────┬───────────┘                      │
│                                 │                                   │
│                     ┌───────────▼───────────┐                      │
│                     │ LangGraph Orchestrator │                      │
│                     │   (Conversational AI)  │                      │
│                     └───────────┬───────────┘                      │
│                                 │                                   │
│                     ┌───────────▼───────────┐                      │
│                     │      TTS Engine        │                      │
│                     │   (Audio Synthesis)    │                      │
│                     └───────────────────────┘                      │
│                                                                     │
│                         Audio Output                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Core Components

| Component | Description | Technology |
|-----------|-------------|------------|
| **API Gateway** | Request ingestion, rate limiting, auth | Python / FastAPI |
| **Scheduling Engine** | Priority-based task scheduling | C++17 |
| **Task Coordinator** | Distributed work queue and coordination | Redis Streams |
| **Worker Pool** | Concurrent inference workers | C++ / Python |
| **ASR Engine** | Automatic Speech Recognition | Whisper / CUDA |
| **LLM Inference** | Language model serving | vLLM / SGLang / CUDA |
| **TTS Engine** | Text-to-Speech synthesis | Python / CUDA |
| **Orchestrator** | Conversational flow management | LangGraph / LangChain |
| **Vector Store** | Semantic retrieval for RAG | Vector DB (Pinecone/Weaviate) |

---

## Technology Stack

| Category | Technology | Purpose |
|----------|------------|---------|
| **Systems** | C++17 | Scheduling engine, worker coordination |
| **GPU** | CUDA | Kernel optimization, inference acceleration |
| **Inference** | vLLM / SGLang | High-throughput LLM serving |
| **Language** | Python 3.11+ | API layer, pipeline orchestration |
| **Queue** | Redis Streams | Distributed task coordination |
| **Orchestration** | LangGraph / LangChain | Conversational AI flow |
| **Vector DB** | Pinecone / Weaviate | RAG semantic retrieval |
| **Database** | PostgreSQL | Persistent state, metadata |
| **Containers** | Docker / Kubernetes | Deployment, scaling |
| **Cloud** | GCP (GKE, GPU nodes) | Production infrastructure |
| **Build** | CMake | C++ build system |
| **Testing** | pytest / Google Test | Test frameworks |

---

## Repository Structure

```
VoiceGrid/
├── README.md                    ← This file
├── PROJECT_CHARTER.md           ← Project mission, objectives, scope
├── ROADMAP.md                   ← Detailed 9-stage development roadmap
├── PROJECT_STATUS.md            ← Live project status tracker
├── DECISIONS.md                 ← Architecture decision log
├── SESSION_HANDOFF.md           ← AI session continuity context
├── CLAUDE.md                    ← AI assistant operating guide (Claude)
├── AGENTS.md                    ← AI assistant operating guide (all AIs)
│
├── docs/
│   ├── architecture/            ← Architecture diagrams and specs
│   ├── system-design/           ← System design documents
│   ├── benchmarks/              ← Benchmark methodology and results
│   ├── research/                ← Research notes and references
│   ├── SESSION_START_PROMPT.md  ← Prompt to start a new AI session
│   └── SESSION_END_PROMPT.md    ← Prompt to end an AI session
│
├── learning/
│   ├── DSA.md                   ← Data structures and algorithms journal
│   ├── CPP.md                   ← C++ learning journal
│   ├── CUDA.md                  ← CUDA and GPU programming journal
│   ├── REDIS.md                 ← Redis learning journal
│   ├── DISTRIBUTED_SYSTEMS.md   ← Distributed systems concepts journal
│   ├── LANGGRAPH.md             ← LangGraph / LangChain journal
│   ├── BLOCKCHAIN.md            ← Blockchain (optional) journal
│   ├── PYTHON.md                ← Python async and infra journal
│   ├── GPU_INFERENCE.md         ← GPU inference optimization journal
│   └── SYSTEM_DESIGN.md         ← System design patterns journal
│
├── hackathons/
│   ├── README.md                ← Hackathon management guide
│   ├── H01-priority-scheduler/  ← Stage 1: C++ priority queue scheduler
│   ├── H02-redis-task-queue/    ← Stage 2: Redis distributed task queue
│   ├── H03-concurrent-workers/  ← Stage 3: Multithreaded worker pool
│   └── H04-gpu-inference/       ← Stage 4: GPU inference pipeline
│
├── src/
│   ├── gateway/                 ← API gateway (FastAPI)
│   ├── scheduler/               ← C++ scheduling engine
│   ├── workers/                 ← Worker pool implementation
│   ├── inference/               ← Inference pipeline (ASR, LLM, TTS)
│   └── orchestration/           ← LangGraph orchestrator
│
├── tests/                       ← Test suites
├── benchmarks/                  ← Benchmark scripts and results
│
└── .github/
    ├── ISSUE_TEMPLATE/          ← GitHub Issue templates
    └── pull_request_template.md ← PR template
```

---

## Quick Links

| Document | Purpose |
|----------|---------|
| [PROJECT_CHARTER.md](PROJECT_CHARTER.md) | Mission, objectives, scope, and success criteria |
| [ROADMAP.md](ROADMAP.md) | Detailed 9-stage roadmap with acceptance criteria |
| [PROJECT_STATUS.md](PROJECT_STATUS.md) | Current project state (updated every session) |
| [DECISIONS.md](DECISIONS.md) | Architecture decisions with rationale |
| [CLAUDE.md](CLAUDE.md) | How Claude should operate in this project |
| [AGENTS.md](AGENTS.md) | How any AI assistant should operate in this project |
| [docs/SESSION_START_PROMPT.md](docs/SESSION_START_PROMPT.md) | Prompt for starting a new AI session |
| [docs/SESSION_END_PROMPT.md](docs/SESSION_END_PROMPT.md) | Prompt for ending an AI session |

---

## Roadmap Summary

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

### Checkpoints

| Checkpoint | Hours | Milestone |
|-----------|-------|-----------|
| Checkpoint 1 | 150h | Functional distributed Voice AI MVP |
| Checkpoint 2 | 300h | GPU-backed inference with initial benchmarks |
| Checkpoint 3 | 550h | Advanced portfolio-ready platform |

---

## Using VoiceGrid with AI Assistants

VoiceGrid is designed for AI-assisted development. To start a new session with any AI assistant:

1. Open [docs/SESSION_START_PROMPT.md](docs/SESSION_START_PROMPT.md)
2. Copy the session start prompt
3. Paste it into your AI assistant as the first message
4. The AI will read the project context and continue from the last verified state

For session continuity guidelines, see [CLAUDE.md](CLAUDE.md) (Claude-specific) or [AGENTS.md](AGENTS.md) (all AI assistants).

---

## Performance Targets

> Note: These are design targets, not measured results. No benchmarks have been run yet.

| Metric | Target |
|--------|--------|
| End-to-end voice pipeline latency (P50) | < 200ms |
| End-to-end voice pipeline latency (P99) | < 500ms |
| Time to First Token (TTFT) | < 100ms |
| GPU utilization (inference) | > 80% |
| Throughput | > 100 concurrent voice sessions |

---

## License

MIT License — see LICENSE file for details. (License file not yet created.)
