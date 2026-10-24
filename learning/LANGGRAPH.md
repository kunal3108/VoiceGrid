# VoiceGrid Learning Journal: LangGraph

Technology: LangGraph + LangChain
Relevance to VoiceGrid: Conversational orchestration and RAG (Stage 6)
Last updated: 2026-10-24

---

## Why LangGraph Matters to VoiceGrid

Stage 6 adds multi-turn conversational intelligence to the VoiceGrid pipeline. LangGraph provides the state machine framework for managing conversation flow: tracking context across turns, routing to different pipeline paths based on intent, integrating retrieval-augmented generation, and supporting human-in-the-loop interventions. Without LangGraph (or equivalent), VoiceGrid can only do single-turn interactions.

Key applications:
- **Stage 6:** Conversation state graph, RAG retrieval node, tool-calling node, conditional routing
- **State management:** Typed conversation state persisted across turns
- **Checkpointing:** Save/restore conversation state (required for session resume)
- **Redis integration:** Use Redis as LangGraph's checkpointing backend

Architecture decision: D005 documents why LangGraph was chosen over plain Python state machines and AutoGen.

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| LangGraph Concepts (nodes/edges/state) | Not Started | — | — | No | — |
| StateGraph construction | Not Started | — | — | No | — |
| Conditional Edges | Not Started | — | — | No | — |
| Checkpointing | Not Started | — | — | No | — |
| Human-in-the-loop | Not Started | — | — | No | — |
| Streaming | Not Started | — | — | No | — |
| LangChain integration | Not Started | — | — | No | — |
| Tool Calling | Not Started | — | — | No | — |
| Memory in LangGraph | Not Started | — | — | No | — |
| Multi-agent architectures | Not Started | — | — | No | — |
| Interrupts and Resumption | Not Started | — | — | No | — |
| LangGraph with Redis | Not Started | — | — | No | — |
| Production deployment considerations | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order:

1. **LangGraph Concepts** — Understand nodes, edges, and state before any code
2. **StateGraph construction** — Build the simplest possible graph (two nodes, one edge)
3. **LangChain integration** — Understand how LangChain models and chains are used within nodes
4. **Conditional Edges** — Route conversation to different nodes based on state values
5. **Tool Calling** — Add an external tool node; understand how LLM decides when to call tools
6. **Memory in LangGraph** — Understand short-term (in-state) vs. long-term (persisted) memory
7. **Checkpointing** — Implement state persistence (SQLite first, then Redis)
8. **Streaming** — Stream LLM tokens back to the caller as they are generated
9. **Interrupts and Resumption** — Pause graph execution mid-node; resume after external input
10. **Human-in-the-loop** — Use interrupts to inject human approval/override steps
11. **LangGraph with Redis** — Replace default checkpointer with Redis-backed checkpointer
12. **Multi-agent architectures** — Understand how multiple LangGraph graphs can coordinate
13. **Production deployment considerations** — Thread safety, async execution, error handling at scale

---

## Topic Notes

### LangGraph Core Concepts

**Graph:** A collection of nodes (processing steps) connected by edges (transitions). The graph defines all possible paths a conversation can take.

**Node:** A Python function or object that takes the current state as input and returns a state update. Nodes are where logic lives: LLM calls, retrieval, tool execution.

**Edge:** A directed connection between nodes. Can be:
- **Regular edge:** Always transitions from node A to node B
- **Conditional edge:** Inspects state and routes to one of several target nodes

**State:** A TypedDict that carries all conversation data. Every node reads from and writes to the state. State is the shared memory of the graph.

**Example state:**
```python
from typing import TypedDict, List, Annotated
import operator

class VoiceGridState(TypedDict):
    audio_input: bytes
    transcript: str
    intent: str
    retrieved_context: List[str]
    llm_response: str
    audio_output: bytes
    turn_count: int
    messages: Annotated[List[dict], operator.add]  # append-only
```

- **Notes:** (empty)

### StateGraph Construction
```python
from langgraph.graph import StateGraph, END

graph = StateGraph(VoiceGridState)

# Add nodes
graph.add_node("asr", asr_node)
graph.add_node("retrieve", retrieval_node)
graph.add_node("llm", llm_node)
graph.add_node("tts", tts_node)

# Add edges
graph.set_entry_point("asr")
graph.add_edge("asr", "retrieve")
graph.add_edge("retrieve", "llm")
graph.add_edge("llm", "tts")
graph.add_edge("tts", END)

compiled = graph.compile()
```
- **Notes:** (empty)

### Conditional Edges
```python
def route_by_intent(state: VoiceGridState) -> str:
    if state["intent"] == "question":
        return "retrieve"
    elif state["intent"] == "command":
        return "llm"
    else:
        return "clarify"

graph.add_conditional_edges(
    "asr",
    route_by_intent,
    {
        "retrieve": "retrieve",
        "llm": "llm",
        "clarify": "clarify"
    }
)
```
- **Notes:** (empty)

### Checkpointing
- **What:** Saves graph state after each node execution. Allows resuming from any point.
- **Default:** In-memory (lost on restart)
- **SQLite:** Persistent, good for development
- **Redis:** Persistent, scalable, required for production VoiceGrid (integrates with existing Redis)
- **Usage:**
```python
from langgraph.checkpoint.redis import RedisSaver
checkpointer = RedisSaver.from_conn_string("redis://localhost:6379")
compiled = graph.compile(checkpointer=checkpointer)
```
- **Notes:** (empty)

### Human-in-the-Loop
- **Pattern:** Add an interrupt before or after a node; execution pauses until human input is provided
- **Use case in VoiceGrid:** Allow human supervisor to review LLM response before TTS synthesis in sensitive calls
- **Implementation:** Use graph.compile(interrupt_before=["tts"]) and then graph.update_state() to resume
- **Notes:** (empty)

### LangGraph with Redis
- **Redis as checkpointer:** State is stored in Redis; every graph invocation with the same thread_id resumes from the last checkpoint
- **VoiceGrid advantage:** Already using Redis for task coordination; adding LangGraph checkpointing reuses existing infrastructure
- **Notes:** (empty)

---

## VoiceGrid Conversation Graph Design (Target — Stage 6)

```
Entry: Audio Input
    │
    ▼
[ASR Node] → Transcribes audio to text
    │
    ▼
[Intent Classification Node] → Classifies user intent
    │
    ├── (question) ──────────▶ [Retrieval Node] → Vector DB search
    │                               │
    │                               ▼
    │                          [LLM Node] → Generate response with context
    │
    ├── (command) ──────────▶ [Tool Calling Node] → Execute tool
    │                               │
    │                               ▼
    │                          [LLM Node] → Generate response with tool result
    │
    └── (unclear) ──────────▶ [Clarification Node] → Ask for clarification
                                    │
                                    ▼ (loop back to ASR)

    After LLM Node:
    │
    ▼
[TTS Node] → Synthesize audio from LLM response
    │
    ▼
Output: Audio Response
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
