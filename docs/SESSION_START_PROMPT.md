# VoiceGrid Session Start Prompt

This file contains prompt templates for starting a new AI assistant session on the VoiceGrid project. Copy the appropriate template and paste it as your first message to the AI assistant.

---

## Full Session Start Prompt

Use this prompt when starting a fresh session or when the AI assistant has no prior context.

---

```
You are helping me build VoiceGrid, a GPU-optimized distributed inference infrastructure platform for real-time Voice AI.

Before doing anything, please read the following files in this order:

1. PROJECT_CHARTER.md — project mission, scope, non-goals, quality standards
2. PROJECT_STATUS.md — current stage, active task, completed work, blockers
3. ROADMAP.md — detailed acceptance criteria for all stages
4. DECISIONS.md — approved technology stack and architecture decisions
5. SESSION_HANDOFF.md — what was done last session and what comes next

After reading all five files, tell me:
1. What stage we are currently in
2. What the current active task is
3. What the last verified state is (last completed acceptance criterion or last test that passed)
4. What the next immediate action is
5. Any ambiguity in the current state that needs clarification before we proceed

Rules you must follow:
- VoiceGrid's mission statement is: "VoiceGrid is a GPU-optimized, distributed inference infrastructure platform designed to improve the latency, throughput, scalability, and reliability of real-time Voice AI workloads." — use this exact statement, do not paraphrase
- Do not redesign or change the architecture without my explicit approval
- Do not add new technologies to the stack without a DECISIONS.md entry and my approval
- Do not claim tests pass without actually running them
- Do not invent benchmark results
- VoiceGrid is NOT a chatbot, trading platform, generic RAG framework, or blockchain application
- When a concept is a learning objective, teach it to me before implementing it
- At the end of our session, update PROJECT_STATUS.md and SESSION_HANDOFF.md

If the current state is ambiguous (e.g., PROJECT_STATUS.md says a task is "in progress" but you cannot tell how far it got), ask me one clarifying question before proceeding.

Do not begin implementation work until I confirm your understanding of the current state is correct.
```

---

## Minimal Start Prompt

Use this for quick session resumption when you trust that context is clear from SESSION_HANDOFF.md.

---

```
Continue working on VoiceGrid. Read PROJECT_STATUS.md and SESSION_HANDOFF.md first, then tell me the current task and last verified state. Follow all rules in CLAUDE.md / AGENTS.md.
```

---

## Hackathon-Specific Start Prompt

Use this when starting a new hackathon sprint.

---

```
We are starting hackathon [H0X: NAME] for VoiceGrid.

Read these files:
1. PROJECT_CHARTER.md
2. PROJECT_STATUS.md
3. hackathons/[H0X-name]/README.md
4. SESSION_HANDOFF.md

Then:
1. Confirm the hackathon objective in one sentence
2. List the first 3 implementation tasks we need to complete
3. Identify any prerequisites that need to be verified before we begin coding
4. Ask me which learning objectives I want to tackle independently vs. with your help

Follow all rules in CLAUDE.md / AGENTS.md. Do not begin implementation until I confirm readiness.
```

---

## Stage Completion Verification Prompt

Use this to verify that all acceptance criteria for a stage are met before closing it.

---

```
We believe Stage [N] of VoiceGrid is complete. Please verify this by:

1. Reading ROADMAP.md — Stage [N] acceptance criteria
2. Reading PROJECT_STATUS.md — completed work section
3. Reading the hackathon README for the relevant hackathon
4. Checking the test output I will provide

For each acceptance criterion:
- Mark it VERIFIED if we have clear evidence it is met
- Mark it UNVERIFIED if we cannot confirm it without running tests
- Mark it NOT MET if there is evidence it is not met

After the review, tell me:
- How many criteria are verified / unverified / not met
- What we need to do to close out any unverified criteria
- Whether you believe Stage [N] can be officially closed

Do not mark Stage [N] complete unless every criterion is verified.
```

---

## Fallback Prompt (For AI Assistants That Cannot Read Files)

Use this when the AI assistant does not have direct file access (e.g., ChatGPT in chat mode without file attachment).

---

```
I am working on VoiceGrid, a GPU-optimized distributed inference infrastructure platform for real-time Voice AI.

VoiceGrid's mission: "VoiceGrid is a GPU-optimized, distributed inference infrastructure platform designed to improve the latency, throughput, scalability, and reliability of real-time Voice AI workloads."

I am going to paste the contents of several files so you have context. Please do not start working until I have pasted all of them and explicitly asked you to begin.

Files I will paste:
1. PROJECT_CHARTER.md contents
2. PROJECT_STATUS.md contents
3. SESSION_HANDOFF.md contents

[Paste file contents here]

After reading the above, tell me the current stage, active task, and next immediate action. Follow the rules in the AGENTS.md file (which I have already described above).
```

---

## Tips for Session Start

1. **Always read SESSION_HANDOFF.md** — this is the single most important context file for resuming work
2. **Confirm the current task before starting** — "What is the current task?" is a good first question
3. **Verify tests pass before adding new code** — if the previous session left tests failing, fix that first
4. **Check GitHub Issues** — if remote is configured, the active GitHub Issue has the most detailed task spec
5. **Don't skip the protocol** — it takes 2 minutes to read the context files and saves hours of confusion
