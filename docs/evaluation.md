# DentalAI Nexus — public evaluation notes

The private project contains more detailed logs and artifacts than can be published. This page records the types of evidence that are suitable for a public overview and gives examples already documented in aggregate form.

## Evaluation principles

- Separate compatibility, throughput, latency, quality, and clinical review.
- Prefer held-out sets and deterministic checks where possible.
- Track provenance for benchmark inputs and model artifacts.
- Report sample counts and workload shape.
- Distinguish live-proven behavior from planned or experimental behavior.
- Never publish patient audio, transcripts, clinical notes, prompts, completions, or private source text.

## Aggregate results suitable for disclosure

### Local LLM runtime benchmark

A documented Prism/llama.cpp-derived benchmark used a quantized 27B-class model on an NVIDIA RTX 5070 Ti. In the validated four-slot comparison:

| Total context | Median latency | p95 latency | Aggregate throughput | HTTP successes |
| ---: | ---: | ---: | ---: | ---: |
| 8,192 | 3.845 s | 3.918 s | 1,895 tok/s | 20/20 |
| 16,384 | 3.411 s | 3.537 s | 2,136 tok/s | 20/20 |
| 32,768 | 3.541 s | 3.617 s | 2,058 tok/s | 20/20 |

The workload used identical synthetic token-count inputs, a warm-up, five measured repetitions, four requests per repetition, and a maximum generation budget. The result is a runtime/throughput measurement, not a dental quality result. The 32,768-context configuration also accepted a roughly 7,011-token prompt plus a 256-token generation budget without truncation or overflow in the documented test.

### Qualcomm mobile ASR compatibility

Android/QNN tests on a Qualcomm SM8550-class device demonstrated successful execution of several ASR models using the QNN backend. Same-input observations included:

| Model family | Approximate RTF | Public interpretation |
| --- | ---: | --- |
| SenseVoice Small INT8 | 0.006 | Fast fixed-window compatibility run |
| Distil-Whisper Medium | 0.132 | Mobile execution and timing observation |
| Nemotron 3.5 ASR 0.6B streaming | 0.20 | Streaming compatibility and timing observation |

These figures are not WER, CER, or production-quality claims. Source audio, normalized references, decoded text, and device-specific artifacts remain private. Additional bounded shape and endurance tests were also recorded, but public release should continue to use aggregate timing only.

### Dual-V100/NVLink status

The private system records dual-V100/NVLink runtime and Registry integration, including a live-served 27B-class model path. A bounded synthetic benchmark was subsequently run against that live tensor-parallel endpoint:

| Test | Result |
| --- | --- |
| Model/runtime | Quantized 27B-class model through the existing vLLM tensor-parallel endpoint |
| Workload | 5 rounds × 4 concurrent requests; 24 prompt tokens and 10 generated tokens per request |
| Responses | 20/20 HTTP 200 responses; exact synthetic marker returned |
| Median request completion time | 0.547 s |
| p95 request completion time | 0.616 s |
| Observed request range | 0.519–0.616 s |

This is a small runtime smoke benchmark, not a scaling study, a model-quality evaluation, or a clinical result. It measures end-to-end HTTP completion time for a tiny synthetic workload and should not be compared directly with larger-context or different-runtime benchmarks. A more useful dual-V100 study would vary prompt length, generation length, concurrency, model configuration, and tensor-parallel behavior.

A first bounded scaling follow-up used the same tiny synthetic workload:

| Concurrent requests | Median round wall time | Requests per round |
| ---: | ---: | ---: |
| 1 | 0.254 s | 1 |
| 2 | 0.261 s | 2 |
| 4 | 0.491 s | 4 |
| 8 | 0.530 s | 8 |
| 16 | 0.802 s | 16 |
| 32 | 1.158 s | 32 |
| 64 | 1.803 s | 64 |
| 128 | 4.219 s | 128 |

Each concurrency level was measured over three rounds, and all observed responses were successful. The 16-request case completed 48/48 requests successfully, with round times of 0.801–0.886 seconds. The 32-request case completed 96/96 requests successfully, with round times of 1.054–1.179 seconds. The 64-request case completed 192/192 requests successfully, with round times of 1.796–1.824 seconds. The 128-request case completed 384/384 requests successfully, with round times of 3.778–4.232 seconds. This means the client launched 128 parallel HTTP requests; server-side in-flight admission was not independently instrumented, so the result does not prove that vLLM actively executed all 128 requests simultaneously rather than queueing some behind scheduler limits. At the end of the 128-request test, both V100s reported approximately 31.36 GiB of 32 GiB memory in use, so this is a high-occupancy result rather than a comfortable operating point. A short endurance pass then completed 30 rounds of four concurrent requests—120 requests total—with 30/30 successful rounds over approximately 18 seconds. This is useful as a live stability check, but it is not long enough to establish thermal endurance, leak-free operation over hours, or production capacity.

The current evidence set does not include a directly comparable Blackwell-vs-V100 run from the same session, nor a quantitative RAG retrieval-quality or ASR WER/CER benchmark. Those remain future measurements rather than implied results.

### Reusable StackLab Node C benchmark run

A reusable StackLab suite was run against the same dual-V100 vLLM configuration
using the served Qwen3.6 27B AWQ artifact. The suite used synthetic prompts,
greedy decoding, and recorded aggregate timings and output digests without
retaining prompt or completion text:

| Workload | Result |
| --- | ---: |
| Short generation: 345 input / 256 output tokens | 47.8–48.1 output tok/s |
| Medium generation: 1,340 input tokens | 44.8–47.7 output tok/s |
| Long generation: 5,326 input tokens | 36.5–47.3 output tok/s |
| Eight-request short client burst | 158.4–158.7 aggregate output tok/s |
| Mixed workload, four requests | 4/4 successful; 14.707 s wall time |
| Mixed workload, eight requests | 8/8 successful; 15.787 s wall time |
| Same-prefix repeat, 10,640 input tokens | 6.539 s cold; 3.047 s repeat; output digest matched |
| Same-prefix repeat, 42,523 input tokens | 35.102 s cold; 3.505 s repeat; output digest matched |
| Five-minute endurance pass | 92/92 successful requests |

The eight-request and mixed-workload values represent parallel client
submissions. Server-side simultaneous admission was not independently
instrumented, so these results should not be interpreted as proof that all
requests were actively executing at the same instant rather than being queued
by the runtime scheduler. The prefix-repeat cases in this particular run did
not force GPU-cache eviction; they demonstrate repeat-prefix consistency, not a
new CPU-offload restoration result. The separate 4K–118K matrix below remains
the evidence for GPU eviction and host-RAM prefix restoration.

### Hybrid-cache and CPU-offload validation

Node C validation records cover the same serving setup used by the current
dual-V100 endpoint: same-engine prefix reuse with a paged GPU KV cache and
host-RAM offload. The tests used synthetic, deterministic
prefixes, forced GPU-cache eviction, then restored the same prompt from the
CPU-resident cache. Greedy output tokens matched between the cold-prefill and
restored requests at every tested size:

| Prefix size | Cold prefill | Restored request | External-cache hits |
| ---: | ---: | ---: | ---: |
| 4,031 tokens | 2.660 s | 0.423 s | 3,920 |
| 16,320 tokens | 10.989 s | 1.057 s | 15,680 |
| 32,704 tokens | 26.590 s | 1.339 s | 32,144 |
| 65,469 tokens | 73.015 s | 2.000 s | 65,072 |
| 120,768 tokens | 196.452 s | 2.206 s | 120,736 |

These are correctness and cache-behavior results for the same private runtime
configuration, not claims that arbitrary models or deployments support the same
behavior. The current concurrency and endurance measurements in this document
use that same setup; they test short-request serving behavior rather than
long-context cache restoration. The records also note that the offload cache was
same-engine and did not survive a server restart. A separate controlled tuning
report validated an incident-shaped mixed workload of four active decodes plus
four prefills, as well as concurrent asynchronous offload/restore with new
prefills, without an observed CUDA OOM under the tuned configuration. The public
project does not publish the prompts, generated text, raw logs, private launch
paths, or runtime patches.

### CPU and multimodal runtime evidence

The system has also exercised CPU-compatible document/OCR paths and GPU-backed layout/vision paths. Public documentation should describe those as runtime compatibility and integration evidence unless a complete, shareable quality benchmark exists.

## What is not currently claimed

The project does not currently claim a published clinical validation study, diagnostic accuracy, superiority over OralGPT/OralAgent, or a general dental benchmark leaderboard result. Those would require separately designed, licensed, and reviewable studies.
