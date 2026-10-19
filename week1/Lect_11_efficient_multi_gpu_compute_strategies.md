# Lecture Notes: Efficient Multi-GPU Compute Strategies
**Week1 1:** Generative AI use cases, project lifecycle, and model pre-training

---

## 1. Why Scale Beyond a Single GPU

- At some point you will likely need to scale model training beyond a single GPU.
- Multi-GPU strategies are **required** when the model becomes too big to fit on a single GPU.
- Even when the model **does** fit on one GPU, using multiple GPUs can **speed up training**.
- It is therefore useful to know how to distribute compute across GPUs, even for small models.

---

## 2. Case 1: Model Fits on a Single GPU — Distributed Data Parallel (DDP)

### Core idea
- The first step in scaling is to **distribute large datasets across multiple GPUs** and process batches of data **in parallel**.
- This is a **model replication** technique.
- A popular implementation is **PyTorch's Distributed Data Parallel (DDP)**.

### How DDP works
1. DDP **copies the model onto each GPU**.
2. **Batches of data are sent to each GPU in parallel**.
3. Each batch is processed in parallel on its GPU.
4. A **synchronization step** combines the results of each GPU.
5. The combined results **update the model on each GPU**, so the model remains **identical across chips**.

### Result
- Parallel computation across all GPUs leads to **faster training**.

### Key limitation
- DDP requires that the **model weights plus all additional training parameters, gradients, and optimizer states fit onto a single GPU**.
- If the model is too big for this, use **model sharding**.

---

## 3. Case 2: Model Too Big for One GPU — Model Sharding with FSDP

- A popular implementation of model sharding is **PyTorch's Fully Sharded Data Parallel (FSDP)**.
- FSDP is motivated by a **2019 paper from Microsoft researchers** that proposed a technique called **ZeRO**.

---

## 4. ZeRO (Zero Redundancy Optimizer)

### Goal
- **Optimize memory** by distributing (sharding) model states across GPUs with **zero data overlap**.
- Enables scaling training across GPUs when the model doesn't fit in the memory of a single chip.

### Memory components recap (from earlier in the week)
- The **largest memory requirement is the optimizer states**, which take up **twice as much space as the weights**.
- Followed by the **weights** themselves and the **gradients**.
- Visual representation used in the lecture:
  - Parameters = **blue**
  - Gradients = **yellow**
  - Optimizer states = **green**

### Problem with model replication (DDP)
- A **full model copy must be kept on each GPU**, causing **redundant memory consumption** — the same numbers are stored on every GPU.

### How ZeRO solves it
- ZeRO **eliminates redundancy** by **sharding** the model parameters, gradients, and optimizer states across GPUs instead of replicating them.
- **Communication overhead** for syncing model states stays **close to that of DDP**.

### Three ZeRO optimization stages

| Stage | What is sharded | Memory reduction |
|-------|-----------------|------------------|
| **Stage 1** | Optimizer states only | Up to **4×** |
| **Stage 2** | Gradients (in addition to stage 1) | Up to **8×** (when combined with stage 1) |
| **Stage 3** | All components, including model parameters (in addition to stages 1 and 2) | **Linear with number of GPUs** |

- Example: sharding across **64 GPUs** could reduce memory by a factor of **64**.

---

## 5. How FSDP Works

### Data and model distribution
- Like DDP, FSDP **distributes data** across multiple GPUs.
- Unlike DDP, FSDP also **shards the model parameters, gradients, and optimizer states** across GPU nodes, using one of the strategies specified in the ZeRO paper.
- This allows working with models **too big to fit on a single chip**.

### DDP vs. FSDP data availability
- **DDP:** Each GPU has all model states needed to process each batch **available locally**.
- **FSDP:** You must **collect this data from all GPUs before the forward and backward pass**.

### FSDP operational flow
1. Each GPU **requests data from other GPUs on demand** to **materialize sharded data into unsharded data** for the duration of the operation.
2. After the operation, the **unsharded non-local data is released** back to the other GPUs as original sharded data.
3. Optionally, you can **keep the unsharded data for future operations** (for example, during the backward pass) — but this **requires more GPU RAM**.
   - This is a typical **performance vs. memory trade-off** decision.
4. After the **backward pass**, FSDP **synchronizes gradients across GPUs** in the same way as in DDP.

### Memory-saving options
- Model sharding with FSDP **reduces overall GPU memory utilization**.
- Optionally, FSDP can **offload part of the training computation to CPUs** to reduce GPU memory use further.

---

## 6. The Sharding Factor

The **sharding factor** configures the level of sharding to manage the performance vs. memory-utilization trade-off.

| Sharding factor | Behavior |
|-----------------|----------|
| **1** | No sharding — replicates the full model, similar to DDP |
| **Maximum number of available GPUs** | **Full sharding** — most memory savings, but **increases communication volume** between GPUs |
| **Anything in between** | **Hybrid sharding** |

---

## 7. Performance Comparison: FSDP vs. DDP

### Test setup
- Up to **512 NVIDIA A100 GPUs**, each with **80 GB** memory.
- Metric: **teraflops per GPU** (1 teraflop = 1 trillion floating point operations per second).

### Figure 1: FSDP performance across T5 model sizes
- Curves shown: FSDP **full sharding** (blue), **hybrid sharding** (orange), **full replication** (green); **DDP** (red) for reference.
- **T5 with 611 million and 2.28 billion parameters:** FSDP and DDP performance are **similar**.
- **T5 with 11.3 billion parameters (beyond 2.28B):**
  - **DDP runs into an out-of-memory error.**
  - **FSDP handles the model easily** and achieves **much higher teraflops** when the model's precision is lowered to **16 bits**.

### Figure 2: Scaling number of GPUs (11B T5 model)
- Increasing GPUs from **8 to 512** gives a **7% decrease in per-GPU teraflops**.
- Plotted with batch size **16** (orange) and batch size **8** (blue).
- Explanation: as the model grows and is distributed across more GPUs, **increased communication volume between chips** starts to hurt performance, **slowing computation**.

### Summary
- FSDP can be used for **both small and large models** and can **seamlessly scale training across multiple GPUs**.

---

## 8. Key Message of the Lecture

- You **don't need to remember all the technical details**.
- The most important thing is to have a **sense of how data, model parameters, and training computations are shared across processes** when training LLMs.

---

## 9. Looking Ahead

- Because training across GPUs is **expensive and technically complex**, some researchers are exploring ways to achieve **better performance with smaller models**.
- The next video covers research into **compute-optimal models**.

---

## Quick Summary Cheat Sheet

| Technique | Use when | Key idea |
|-----------|----------|----------|
| **DDP** | Model (weights, gradients, optimizer states) fits on one GPU | Replicate model on each GPU, process data batches in parallel, synchronize results |
| **FSDP** | Model too big for one GPU | Shard parameters, gradients, optimizer states across GPUs (based on ZeRO); gather on demand |
| **ZeRO Stage 1** | — | Shard optimizer states (up to 4× memory reduction) |
| **ZeRO Stage 2** | — | Also shard gradients (up to 8× with stage 1) |
| **ZeRO Stage 3** | — | Also shard parameters (linear reduction with GPU count) |
| **Sharding factor** | Tuning performance vs. memory | 1 = DDP-like; max GPUs = full sharding; in between = hybrid |
