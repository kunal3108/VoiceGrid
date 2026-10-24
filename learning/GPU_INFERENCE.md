# VoiceGrid Learning Journal: GPU Inference

Technology: GPU Inference Optimization (vLLM, SGLang, Whisper, TTS)
Relevance to VoiceGrid: Stages 4, 5 — inference pipeline implementation and optimization
Last updated: 2026-10-24

---

## Why GPU Inference Matters to VoiceGrid

VoiceGrid's primary performance targets are latency and throughput: < 200ms end-to-end for a voice pipeline that includes ASR, LLM, and TTS inference — all on GPU. Meeting these targets requires understanding how modern LLM inference engines work, what limits their performance, and how to optimize them. This is the most specialized technical knowledge area in VoiceGrid and the one most directly tied to the mission.

Key applications:
- **Stage 4:** Implement working inference pipeline using Whisper (ASR), vLLM or local LLM (text generation), and TTS
- **Stage 5:** Optimize GPU inference using CUDA-level techniques: quantization, continuous batching, KV-cache
- **Stage 7:** Benchmark inference performance with full methodology

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| LLM Inference fundamentals | Not Started | — | — | No | — |
| KV-Cache | Not Started | — | — | No | — |
| Continuous Batching | Not Started | — | — | No | — |
| Dynamic Batching | Not Started | — | — | No | — |
| Speculative Decoding | Not Started | — | — | No | — |
| Quantization (INT8/INT4/FP8) | Not Started | — | — | No | — |
| Tensor Parallelism | Not Started | — | — | No | — |
| Pipeline Parallelism | Not Started | — | — | No | — |
| vLLM architecture | Not Started | — | — | No | — |
| SGLang architecture | Not Started | — | — | No | — |
| ASR (Whisper) inference optimization | Not Started | — | — | No | — |
| TTS inference optimization | Not Started | — | — | No | — |
| GPU Memory Management | Not Started | — | — | No | — |
| Inference Benchmarking (latency/throughput) | Not Started | — | — | No | — |
| TTFT (Time to First Token) | Not Started | — | — | No | — |
| Voice pipeline end-to-end latency | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order:

1. **LLM Inference fundamentals** — Understand the token generation loop before optimizing it
2. **TTFT (Time to First Token)** — Primary latency metric; understand before any optimization
3. **KV-Cache** — The most important optimization in LLM inference; required for Stage 4
4. **Dynamic Batching** — Understand before continuous batching (simpler model)
5. **Continuous Batching** — The core of vLLM's performance advantage; required for Stage 5
6. **vLLM architecture** — How vLLM implements continuous batching and PagedAttention
7. **GPU Memory Management** — Required for understanding KV-Cache and model loading
8. **Quantization (INT8/INT4/FP8)** — Stage 5 optimization; understand trade-offs before implementing
9. **ASR (Whisper) inference optimization** — Stage 4 and 5; understand how to speed up Whisper
10. **TTS inference optimization** — Stage 4 and 5; understand TTS model inference characteristics
11. **Inference Benchmarking** — Learn benchmarking methodology before Stage 7
12. **Voice pipeline end-to-end latency** — Understand how individual stage latencies combine
13. **SGLang architecture** — Compare with vLLM; understand where SGLang excels
14. **Speculative Decoding** — Advanced optimization; learn after continuous batching
15. **Tensor Parallelism** — Multi-GPU optimization; learn before Stage 8 if multi-GPU available
16. **Pipeline Parallelism** — Multi-GPU optimization; understand alongside tensor parallelism

---

## Topic Notes

### LLM Inference Fundamentals
- **Token generation loop:** LLMs generate one token at a time (autoregressive). Each token requires a forward pass through the model.
- **Prefill phase:** Process the entire input prompt in one forward pass. Compute-bound. Produces KV-Cache for all input tokens.
- **Decode phase:** Generate output tokens one at a time. Memory-bandwidth-bound (KV-Cache reads dominate).
- **Latency components:**
  - TTFT (Time to First Token) = prefill latency
  - ITL (Inter-Token Latency) = latency per generated token during decode
  - Total generation latency = TTFT + (output_tokens × ITL)
- **Notes:** (empty)

### KV-Cache
- **What:** During the decode phase, key and value tensors from previous tokens are cached to avoid recomputation
- **Without KV-Cache:** Each decode step recomputes attention over all previous tokens → O(n²) complexity
- **With KV-Cache:** Each decode step only computes attention for the new token → O(n) complexity per step
- **Memory cost:** KV-Cache size = 2 × num_layers × num_heads × head_dim × sequence_length × batch_size × dtype_bytes
- **Problem:** KV-Cache grows with sequence length. For long conversations, it can consume most GPU memory.
- **PagedAttention (vLLM):** Stores KV-Cache in non-contiguous pages, like OS virtual memory. Reduces fragmentation, enables larger effective batch sizes.
- **Notes:** (empty)

### Continuous Batching
- **Static batching problem:** Must wait for the slowest request in the batch to finish before starting new requests. Long-running requests block short requests.
- **Continuous batching:** New requests are inserted into the batch as soon as a slot is free (iteration-level scheduling). Dramatically improves GPU utilization for mixed workloads.
- **VoiceGrid relevance:** Voice AI workloads have variable response lengths. A voice assistant answering a simple question finishes in 20 tokens; a complex answer takes 200 tokens. Without continuous batching, short requests wait behind long ones.
- **Notes:** (empty)

### Quantization
| Precision | Memory per parameter | Speed vs FP16 | Quality Loss |
|-----------|---------------------|---------------|-------------|
| FP32 | 4 bytes | 0.5x | None (baseline) |
| FP16/BF16 | 2 bytes | 1x (baseline) | Negligible |
| INT8 | 1 byte | 1.5-2x | Small (< 1% perplexity) |
| INT4 | 0.5 bytes | 2-3x | Moderate (depends on method) |
| FP8 | 1 byte | ~2x | Small (hardware-accelerated on H100) |

- **Methods:** GPTQ, AWQ, bitsandbytes, LLM.int8()
- **VoiceGrid approach (Stage 5):** INT8 quantization as baseline optimization; benchmark quality delta before declaring success
- **Notes:** (empty)

### vLLM Architecture
- **Core innovations:**
  1. **PagedAttention:** Non-contiguous KV-Cache storage, OS-style virtual memory for GPU
  2. **Continuous batching:** Iteration-level scheduling
  3. **Efficient CUDA kernels:** Fused attention, optimized sampling
- **VoiceGrid integration:** Use vLLM's OpenAI-compatible API server for LLM stage; connect via httpx
- **Key metrics to monitor:** GPU KV-Cache utilization (avoid eviction), queue length, P50/P99 TTFT
- **Notes:** (empty)

### ASR (Whisper) Inference Optimization
- **Whisper architecture:** Encoder-decoder transformer. Encoder processes audio; decoder generates transcript tokens.
- **Optimization levers:**
  1. **Model size:** tiny (39M) → base → small → medium → large-v3. Speed vs. accuracy trade-off.
  2. **FP16 inference:** Default on GPU; ensure not using FP32 accidentally
  3. **faster-whisper:** CTranslate2-based Whisper implementation; 4-8x faster than original
  4. **Batch processing:** If multiple short utterances arrive simultaneously, batch them
  5. **Voice Activity Detection (VAD):** Trim silence before feeding to Whisper; reduces effective audio length
- **VoiceGrid target:** < 50ms for utterances under 10 seconds using faster-whisper on GPU
- **Notes:** (empty)

### TTS Inference Optimization
- **Common TTS models:** Coqui TTS (VITS, YourTTS), Bark, Piper, Kokoro
- **Optimization levers:**
  1. **Model selection:** Smaller models (Piper, fast VITS) have much lower latency than large models (Bark)
  2. **GPU inference:** Ensure TTS model runs on GPU, not CPU
  3. **Streaming synthesis:** Generate audio in chunks rather than waiting for complete synthesis
  4. **FP16/INT8 quantization:** Applicable to neural TTS models
- **VoiceGrid target:** < 50ms for responses under 20 words using streaming synthesis
- **Notes:** (empty)

### Inference Benchmarking Methodology
Key metrics to measure for each inference stage:
- **TTFT:** Time from request arrival to first output token (LLM) or first audio chunk (TTS)
- **Throughput:** Tokens/second (LLM), seconds of audio/second (TTS), words/second (ASR)
- **P50/P95/P99 latency:** Measure over minimum 100 requests
- **GPU utilization:** nvidia-smi or nvml query during benchmark run
- **GPU memory:** Peak usage and KV-Cache occupancy

Benchmark tool: Use Python `time.perf_counter()` for high-resolution timing; avoid `time.time()` for latency measurements under 100ms.

### Voice Pipeline End-to-End Latency
**Latency budget (target: 200ms total):**
| Stage | Target | Notes |
|-------|--------|-------|
| Network in | 10ms | Client to gateway |
| ASR | 50ms | Audio transcription |
| LLM TTFT | 80ms | First token generation |
| TTS first chunk | 40ms | First audio chunk ready |
| Network out | 10ms | Gateway to client |
| **Total** | **190ms** | Under 200ms target |

**Key insight:** LLM and TTS can be partially pipelined — start synthesizing TTS as each sentence fragment arrives from LLM streaming, rather than waiting for complete LLM response.

---

## Resources

- (to be added as topics are studied)

---

## Implementation References

| Topic | File | Stage | Date | Notes |
|-------|------|-------|------|-------|
| (empty) | — | — | — | — |

---

## Open Questions

- (empty — add questions as they arise)
