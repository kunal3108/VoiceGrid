# VoiceGrid Learning Journal: CUDA

Technology: CUDA (Compute Unified Device Architecture)
Relevance to VoiceGrid: GPU kernel optimization for ASR, LLM, and TTS inference (Stage 5)
Last updated: 2026-10-24

---

## Why CUDA Matters to VoiceGrid

VoiceGrid's latency and throughput targets require GPU inference to run as efficiently as possible. PyTorch and other frameworks abstract away GPU details, but they cannot always produce optimal code for every operation. CUDA allows writing custom kernels that directly control thread hierarchy, memory layout, and compute overlap — enabling optimizations not possible at the framework level.

Key applications:
- **Stage 5:** CUDA kernel profiling (Nsight), custom kernel implementation, memory optimization, CUDA Streams
- **KV-Cache management:** Custom memory allocation patterns for LLM KV-Cache
- **Quantization kernels:** INT8/INT4 dequantization and matrix multiply kernels
- **Pipeline parallelism:** CUDA Streams to overlap memory transfer and compute

---

## Progress Tracker

| Topic | Status | Hackathon | Confidence (1-5) | Independent Implementation | Last Practiced |
|-------|--------|-----------|-----------------|---------------------------|---------------|
| GPU Architecture Overview | Not Started | — | — | No | — |
| CUDA Thread Hierarchy (threads/blocks/grids) | Not Started | — | — | No | — |
| CUDA Memory Hierarchy (global/shared/local/constant) | Not Started | — | — | No | — |
| Writing Basic Kernels | Not Started | — | — | No | — |
| Memory Transfers (host↔device) | Not Started | — | — | No | — |
| Synchronization (__syncthreads) | Not Started | — | — | No | — |
| Parallel Reductions | Not Started | — | — | No | — |
| Warp Divergence | Not Started | — | — | No | — |
| Memory Coalescing | Not Started | — | — | No | — |
| Occupancy Optimization | Not Started | — | — | No | — |
| CUDA Streams | Not Started | — | — | No | — |
| cuBLAS basics | Not Started | — | — | No | — |
| NVIDIA Nsight Profiler | Not Started | — | — | No | — |
| nvprof | Not Started | — | — | No | — |
| Kernel Launch Overhead | Not Started | — | — | No | — |
| Error Handling | Not Started | — | — | No | — |

Confidence scale: 1 = Heard of it | 2 = Understand conceptually | 3 = Can implement with reference | 4 = Can implement independently | 5 = Can explain trade-offs and teach

---

## Learning Path

Learn in this order to build understanding correctly:

1. **GPU Architecture Overview** — Understand SM (Streaming Multiprocessor), warp, core hierarchy before programming
2. **CUDA Thread Hierarchy** — The fundamental programming model: threads → blocks → grids
3. **CUDA Memory Hierarchy** — The fundamental performance model: global (slow) → shared (fast) → registers (fastest)
4. **Writing Basic Kernels** — Write, compile (nvcc), run a simple vector addition kernel
5. **Memory Transfers** — cudaMalloc, cudaMemcpy, cudaFree; understand synchronization points
6. **Error Handling** — Wrap every CUDA call with error checking before going further
7. **Synchronization** — __syncthreads for intra-block sync; cudaDeviceSynchronize for host-device sync
8. **Parallel Reductions** — Classic CUDA pattern; understand naive vs. optimized reduction
9. **Warp Divergence** — Understand what happens when threads in a warp take different code paths
10. **Memory Coalescing** — The single most common performance issue: how to structure memory access for throughput
11. **Occupancy Optimization** — Balance between register usage, shared memory, and thread count
12. **CUDA Streams** — Pipeline compute and memory transfer; required for Stage 5 pipeline parallelism
13. **cuBLAS** — High-performance BLAS routines for GPU; understand before writing custom matrix multiply
14. **NVIDIA Nsight Profiler** — Profile before optimizing; required practice before Stage 5
15. **nvprof** — Legacy profiler; understand for older environments
16. **Kernel Launch Overhead** — Understand minimum kernel duration for profitability

---

## Topic Notes

### GPU Architecture Overview
- **Key concepts:** GPU = many simple cores; CPU = few complex cores; GPU excels at data parallelism
- **Streaming Multiprocessor (SM):** Executes thread blocks. Each SM has: CUDA cores, shared memory, registers, warp schedulers
- **Warp:** 32 threads that execute the same instruction simultaneously (SIMT: Single Instruction Multiple Threads)
- **VoiceGrid relevance:** Understanding SM limits helps explain why occupancy matters and why shared memory is precious
- **Notes:** (empty)

### CUDA Thread Hierarchy
- **Hierarchy:** Thread → Block → Grid
  - Thread: single execution unit
  - Block: group of threads that can cooperate via shared memory and __syncthreads
  - Grid: collection of blocks; blocks cannot communicate directly
- **Built-in variables:** threadIdx (x,y,z), blockIdx (x,y,z), blockDim (x,y,z), gridDim (x,y,z)
- **VoiceGrid relevance:** Choosing grid/block dimensions is the first performance decision for every kernel
- **Notes:** (empty)

### CUDA Memory Hierarchy
| Memory Type | Scope | Speed | Size | Notes |
|------------|-------|-------|------|-------|
| Registers | Per thread | Fastest | ~32 per thread | Compiler-managed |
| Shared Memory | Per block | Very fast | 48-96KB per SM | Must be managed explicitly |
| L1/L2 Cache | Per SM / Global | Fast | KB / MB | Automatic |
| Global Memory | All threads | Slowest | GB | Main GPU RAM; cudaMalloc |
| Constant Memory | All threads (read-only) | Fast (cached) | 64KB | Best for broadcast reads |

- **Notes:** (empty)

### Parallel Reductions
- **Pattern:** Reduce N elements to 1 (sum, max, min) using N/2 threads per step
- **Naive reduction:** O(n) work, O(log n) steps, but poor memory access pattern
- **Optimized reduction:** Interleaved addressing, sequential addressing, warp-unrolling
- **VoiceGrid relevance:** Reduction is a common primitive in inference kernels (softmax, layer norm)
- **Notes:** (empty)

### Memory Coalescing
- **Rule:** Consecutive threads should access consecutive memory addresses
- **Why:** GPU memory controller coalesces aligned accesses from consecutive threads into fewer transactions
- **Uncoalesced access:** 32 threads accessing 32 non-consecutive addresses = 32 memory transactions
- **Coalesced access:** 32 threads accessing 32 consecutive addresses = 1 memory transaction
- **VoiceGrid relevance:** This is often the first thing Nsight Compute flags in unoptimized kernels
- **Notes:** (empty)

### CUDA Streams
- **What:** A sequence of CUDA operations that execute in order on the GPU
- **Why:** Multiple streams can overlap: compute in stream 1 while copying memory in stream 2
- **Default stream:** All operations that do not specify a stream use the default stream (synchronous)
- **Usage pattern:** Create stream with cudaStreamCreate, pass stream to kernel launch and memcpy, destroy with cudaStreamDestroy
- **VoiceGrid relevance:** Stage 5 uses streams to overlap ASR memory transfer with LLM compute
- **Notes:** (empty)

### Error Handling
- **Rule:** Every CUDA API call returns cudaError_t; every kernel launch can be checked with cudaGetLastError()
- **Pattern:**
```cpp
#define CUDA_CHECK(call) \
    { \
        cudaError_t err = (call); \
        if (err != cudaSuccess) { \
            fprintf(stderr, "CUDA error at %s:%d: %s\n", __FILE__, __LINE__, cudaGetErrorString(err)); \
            exit(EXIT_FAILURE); \
        } \
    }
```
- **VoiceGrid rule:** Always use CUDA_CHECK in VoiceGrid code. Never ignore CUDA errors.
- **Notes:** (empty)

---

## CUDA Quick Reference

### Kernel Launch Syntax
```cpp
// Launch kernel with gridDim blocks of blockDim threads each
my_kernel<<<gridDim, blockDim, sharedMemBytes, stream>>>(args...);
```

### Common Dimension Patterns
| Use Case | Block Size | Grid Size |
|----------|-----------|-----------|
| 1D array, simple ops | 256 | (N + 255) / 256 |
| 1D array, memory-bound | 512 | (N + 511) / 512 |
| 2D matrix | dim3(16,16) | dim3((W+15)/16, (H+15)/16) |

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
