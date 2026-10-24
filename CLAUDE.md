# VoiceGrid Claude Operating Guide

This file governs how Claude Code and Claude Chat should operate within the VoiceGrid project.

It is not optional background reading. It is a set of operating constraints that Claude must follow in every session. Read this file before reading any other file. If you have not read this file, do not begin work.

---

## Section 1: Mandatory Pre-Session Protocol

At the start of every session, Claude must read the following files in this order before doing anything else:

### Step 1: Read PROJECT_CHARTER.md
- Confirm the mission statement
- Note the non-goals list
- Note the scope-change approval policy

### Step 2: Read PROJECT_STATUS.md
- Identify the current stage
- Identify the current hackathon (if any)
- Identify the current GitHub issue (if any)
- Identify the last completed task
- Identify blockers

### Step 3: Read ROADMAP.md
- Identify the current stage's acceptance criteria
- Confirm prerequisites for the current stage are met

### Step 4: Read DECISIONS.md
- Understand which technologies are approved
- Do not suggest alternatives to accepted decisions without a clear engineering reason

### Step 5: Read SESSION_HANDOFF.md
- Identify the last verified state
- Identify unfinished work from the previous session
- Identify the next immediate action

### Step 6: Inspect relevant source code
- Read the source files relevant to the current task
- Do not assume file contents — read them

### Step 7: Identify the current GitHub issue (if available)
- If a GitHub issue number is referenced in PROJECT_STATUS.md, review the issue's acceptance criteria before beginning work

### After completing all steps:
State your understanding of:
1. What stage you are in
2. What the current task is
3. What the last verified state is
4. What the next immediate action is
5. Any ambiguity that requires clarification before proceeding

Do not begin implementation until the project owner confirms your understanding is correct, unless the task is unambiguous and continuation is clearly safe.

---

## Section 2: Project Identity Rules

### Mission Statement (Mandatory)

VoiceGrid is a GPU-optimized, distributed inference infrastructure platform designed to improve the latency, throughput, scalability, and reliability of real-time Voice AI workloads.

You must use this exact statement when describing VoiceGrid. You may not paraphrase it, generalize it, or replace it with a different description without explicit approval from the project owner.

### What VoiceGrid IS
- A distributed task scheduling engine for inference workloads
- A GPU inference optimization platform for Voice AI (ASR, LLM, TTS)
- A learning vehicle for C++, CUDA, Redis, distributed systems, and GPU inference
- An infrastructure layer, not a user-facing product

### What VoiceGrid IS NOT (Non-Goals)
You must not implement, suggest as critical-path features, or architect VoiceGrid to become any of the following:

| Non-Goal | Why it is excluded |
|----------|--------------------|
| A general-purpose chatbot | VoiceGrid is infrastructure |
| An algorithmic trading platform | Unrelated to the mission |
| A generic RAG framework | RAG is a component of Stage 6, not the system's purpose |
| An autonomous AI agent framework | Out of scope |
| A blockchain or Web3 application | Optional only (D008) |
| A WebRTC or SIP platform | Audio transport is out of scope |
| A consumer voice product | Infrastructure layer only |

### Scope Change Policy
If the project owner asks Claude to implement something that falls outside the current stage's acceptance criteria or changes the mission:

1. State clearly that this is out of scope for the current stage
2. Explain why (reference the relevant section of PROJECT_CHARTER.md)
3. Offer to log a new decision in DECISIONS.md if the change is warranted
4. Do not implement the change until the project owner explicitly approves it

---

## Section 3: AI Behavior Rules

### 3.1 Continue from Latest Verified State
- Read SESSION_HANDOFF.md before beginning any work
- Do not restart a task that was already completed in a previous session
- Do not redo work that is already done — build on it

### 3.2 Do Not Redesign Unnecessarily
- If an architectural decision exists in DECISIONS.md, do not propose replacing it without a clear, specific engineering reason
- Preference: improve the existing design. Redesign only when the existing design has a documented, specific failure

### 3.3 Do Not Introduce Unvetted Technologies
- Do not add a new technology to the critical path without a DECISIONS.md entry
- If you believe a new technology would improve the system, propose it as a decision (explain context, rationale, alternatives) and wait for approval before implementing
- "I think X would be better than Y" is not sufficient justification. Reference specific requirements, benchmarks, or design constraints

### 3.4 Do Not Claim Tests Passed Unless Executed
- Never write "tests pass" without having actually run the tests and seen passing output
- If you cannot run tests (e.g., missing hardware, environment not set up), say so explicitly
- If tests fail, report the failure honestly and help diagnose it

### 3.5 Do Not Mark Tasks Complete Without Evidence
- A task is complete only when its acceptance criteria are met and verified
- Do not mark a GitHub Issue as closed without the project owner confirming acceptance criteria
- Do not update PROJECT_STATUS.md to show a stage as complete without evidence

### 3.6 Do Not Invent Hours or Benchmark Results
- Actual hours are recorded only when the project owner provides them
- Benchmark results are recorded only from actual benchmark runs
- If no measurement exists, the correct answer is "not yet measured" — never estimate or extrapolate

### 3.7 Do Not Auto-Implement Everything — Teach First
- VoiceGrid is a learning project, not just a delivery project
- When a concept is a learning objective (listed in the current stage), follow the Teaching Protocol in Section 4 before implementing
- The goal is for the project owner to understand the concept, not just to have working code

### 3.8 Be Explicit About Uncertainty
- If you are unsure whether something is correct, say so
- If you are writing code that you have not tested, say so
- If an architectural decision has trade-offs, name them

---

## Section 4: Teaching Protocol

When a concept is listed as a learning objective for the current stage, follow this protocol:

### Step 1: Explain the Concept
Provide a clear, precise explanation of the concept. Include:
- What it is
- Why it exists (what problem it solves)
- How it works at a conceptual level
- Where it appears in VoiceGrid specifically

### Step 2: Give the Project Owner the Opportunity to Implement
After explaining, present the project owner with the task:
- "Here is the problem to implement: [specific task]"
- "Try to implement it. Here are the constraints: [constraints]"
- Do not provide the implementation yet

### Step 3: Review the Implementation
When the project owner provides their implementation:
- Review it for correctness
- Point out bugs, edge cases, and style issues
- Do not rewrite it wholesale — give targeted feedback

### Step 4: Explain Time and Space Complexity
After the implementation works:
- Analyze the time complexity of each operation (with Big O notation)
- Analyze the space complexity
- Explain whether the implementation is optimal and why

### Step 5: Discuss System Design Trade-offs
Connect the concept to VoiceGrid's design:
- How does this data structure/algorithm affect the system's latency?
- When would this choice be wrong? What would you use instead?
- What is the real-world impact at scale?

### Step 6: Test the Implementation
- Write or review unit tests for the implementation
- Run the tests and confirm they pass
- Identify edge cases that should be tested

### Step 7: Record Learning Outcome
Update the relevant learning journal (e.g., learning/DSA.md):
- Mark the topic as "Completed"
- Record the implementation reference (file path, hackathon)
- Update confidence level honestly
- Note any remaining gaps

---

## Section 5: End-of-Session Protocol

At the end of every session, Claude must update the following before the session ends:

### Required Updates

**1. PROJECT_STATUS.md**
- Update "Last updated" date
- Update "Completed Work" with all work completed this session (only verified work)
- Update "In Progress" to reflect current state
- Update "Blockers" (add new blockers, remove resolved ones)
- Update "Latest Verified Tests" with test results
- Update "Next Three Actions" to reflect the current state
- Add actual hours if the project owner provides them

**2. SESSION_HANDOFF.md**
- Update "Date" and "AI assistant used"
- Update "Work Completed" section
- Update "Files Created or Modified" section
- Update "Tests Executed and Results"
- Update "Concepts Learned"
- Update "Current Problem" if any blocker exists
- Update "Next Immediate Action" to the specific next task
- Update "Latest Commit" with actual commit hash if a commit was made

**3. Learning Journals**
- Update any learning journal topic that was studied or implemented this session
- Change status from "Not Started" → "In Progress" → "Completed"
- Update confidence level
- Update "Last Practiced" date

**4. GitHub Issues (when accessible)**
- Comment on the active GitHub Issue with progress summary
- Close issues that meet their acceptance criteria
- Create new issues for the next task if they do not exist

### End-of-Session Summary Format

When ending a session, output a summary in this format:

```
## Session Summary — [Date]

### Work Completed (Verified)
- [List only work that is done and tested]

### Files Modified
- [List files with one-line description of change]

### Tests Run
- [List tests run and results]

### Actual Hours This Session
[Number, or "Not recorded" if owner did not provide]

### Blockers
- [Any blockers preventing the next task]

### Next Immediate Action
[Specific, actionable next task]
```

---

## Section 6: Technology Guidance

### C++ Guidelines
- Always compile with `-Wall -Wextra` and fix all warnings
- Use RAII for all resource management — no naked `new` without corresponding `delete` in destructor
- Prefer `std::unique_ptr` over raw pointers for heap-allocated objects
- Use `const` and `constexpr` where possible
- Do not use undefined behavior — run with AddressSanitizer and UndefinedBehaviorSanitizer during development
- Benchmark before declaring a C++ implementation "fast"

### CUDA Guidelines
- Always check CUDA error return codes — never ignore them
- Profile with Nsight before writing custom kernels — confirm the bottleneck is where you think it is
- Validate kernel output for correctness before measuring performance
- Document kernel grid and block dimensions with the rationale
- Use CUDA Streams for overlapping compute and memory transfer — do not leave the GPU idle during transfers

### Redis Guidelines
- Always use consumer groups for task queuing — do not use basic LPOP/RPUSH for distributed task queues
- Implement idempotent consumer logic — assume any task may be delivered more than once
- Monitor consumer lag — a growing lag is a backpressure signal, not just a performance metric
- Use Redis distributed locks (SETNX) sparingly — they add latency and can cause issues under failure scenarios
- Always set TTLs on keys that should not persist indefinitely

### Python Guidelines
- Use type hints for all function signatures
- Use async/await (asyncio) for I/O-bound operations, not threads
- Use `httpx` (not `requests`) for async HTTP clients
- Use `pydantic` for data validation at API boundaries
- Write tests before or alongside code, not after
- Use `pytest` with explicit assertion messages

### LangGraph Guidelines
- Define the conversation state schema explicitly as a typed TypedDict
- Name nodes and edges descriptively — graph structure should be self-documenting
- Use checkpointing from day one — retrofitting checkpointing is harder than including it initially
- Test each node in isolation before testing the full graph
- Profile RAG latency separately from LLM latency — they have different optimization paths

---

## Section 7: Performance Claims Policy

### Never Invent Benchmarks
Claude must not produce fabricated latency numbers, throughput figures, or GPU utilization percentages. If no measurement exists, say "not yet measured."

### Establish Baseline First
Before any optimization, document the baseline:
- What workload was used
- What hardware was used
- What the measured value was (P50, P95, P99)
- What software versions were running

### Document Methodology
Every benchmark result reported in VoiceGrid documentation must include:
- Hardware specification (CPU, GPU model, RAM)
- Software versions (CUDA version, Python version, library versions)
- Workload description (number of requests, payload size, concurrency)
- Measurement tool and how it was used
- Number of runs and statistical approach (min/max/mean/percentile)

### What to Do When Asked for Performance Estimates
If the project owner asks for a performance estimate before measurements exist:
1. Clearly label it as an estimate, not a measurement
2. Explain the reasoning behind the estimate
3. Note that the actual measurement must be run to confirm
4. Do not record the estimate in PROJECT_STATUS.md or benchmark documentation as if it were a measurement

---

## Section 8: File Modification Policy

### Files to Update Every Session (End of Session)
- SESSION_HANDOFF.md — always
- PROJECT_STATUS.md — always

### Files to Update When Work is Completed
- ROADMAP.md — when a stage's acceptance criteria are met (change status)
- DECISIONS.md — when a new architecture decision is made
- Relevant learning/*.md — when a topic is studied or implemented

### Files That Require Project Owner Approval to Modify
- PROJECT_CHARTER.md — mission statement, scope, non-goals
- ROADMAP.md — stage definitions, acceptance criteria, estimated hours (informational updates are fine; structural changes require approval)
- DECISIONS.md — adding a new decision requires approval; updating status of existing decisions does not

### Files Claude Must Never Modify
- This file (CLAUDE.md) — modifying it would change the operating rules without the owner's knowledge
- AGENTS.md — same reason

### Files Claude Should Read But Not Modify Without Instruction
- Source code files — read to understand context; modify only when implementing a specific task

### Creating New Files
- New files in src/, tests/, benchmarks/, hackathons/ are expected as part of implementation
- New files in docs/, learning/, .github/ require explanation of purpose
- Do not create files outside the project directory
