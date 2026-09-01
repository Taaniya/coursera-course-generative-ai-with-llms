# Computational Challenges of Training LLMs — Study Notes

## 1. Core Problem: GPU Memory (Out-of-Memory Errors)

- Training or even just **loading** large language models on Nvidia GPUs frequently triggers **out-of-memory (OOM)** errors.
- **CUDA** (Compute Unified Device Architecture): a collection of Nvidia-developed libraries/tools used by frameworks like **PyTorch** and **TensorFlow** to accelerate operations such as matrix multiplication, which are central to deep learning.
- Root cause: LLMs have an enormous number of parameters, and storing + training all of them requires huge amounts of memory.

## 2. Quantifying the Memory Problem

### 2.1 Storing Model Weights Alone
- A single parameter is typically stored as a **32-bit float (FP32)** = **4 bytes**.
- Memory to store weights = `4 bytes × number of parameters`.
- Example: **1 billion parameters → 4 GB of GPU RAM** just to store the weights (at full FP32 precision).

### 2.2 Additional Memory Needed for Training (Not Just Storage)
Training requires more than just holding the weights. Extra components consuming GPU memory:
1. **Adam optimizer states** [(two per parameter)](https://huggingface.co/blog/Isayoften/optimization-rush#11-model-states-optimizer-states-gradients-and-parameters)
2. **Gradients**
3. **Activations**
4. **Temporary variables** used by functions during computation

- These overheads can add up to roughly **~20 extra bytes per parameter**.
- **Rule of thumb:** total training memory ≈ **~6× the memory needed to just store the model weights**.

### 2.3 Example Calculation
- 1 billion parameter model, FP32 precision:
 - Weights alone: 4 GB
 - Full training requirement: **~24 GB of GPU RAM**
- This is:
 - Too large for most **consumer hardware**
 - Challenging even for **single-processor data center hardware**

## 3. Solution #1: Quantization

### 3.1 Core Idea
- Reduce memory footprint by **lowering the numerical precision** used to represent model weights (and other parameters), rather than reducing the number of parameters.
- Moves values from **32-bit floating point (FP32)** → **16-bit (FP16 / BFLOAT16)** → or **8-bit integers (INT8)**.

### 3.2 How It Works
- Quantization **statistically projects** the original FP32 values into a lower-precision space.
- Uses **scaling factors** calculated from the range of the original FP32 values.
- Modern frameworks support **quantization-aware training**, which learns the scaling factors *during* training (details considered out of scope for this lecture).

### 3.3 Anatomy of Floating Point Representations

| Format | Total Bits | Sign | Exponent | Fraction/Mantissa | Approx. Range | Bytes/Value |
|---|---|---|---|---|---|---|
| **FP32** (full precision) | 32 | 1 | 8 | 23 | ≈ −3×10³⁸ to 3×10³⁸ | 4 |
| **FP16** (half precision) | 16 | 1 | 5 | 10 | −65,504 to +65,504 | 2 |
| **BFLOAT16 (BF16)** | 16 | 1 | 8 | 7 | Same dynamic range as FP32 (truncated fraction) | 2 |
| **INT8** | 8 | 1 | — | 7 (integer range) | −128 to +127 | 1 |

- **Sign bit:** 0 = positive, 1 = negative.
- **Exponent bits:** determine the range of representable numbers.
- **Fraction/mantissa bits:** determine precision (how many significant digits are captured).

### 3.4 Worked Example — Representing Pi (π)
- **FP32:** stores π with high precision (loses only a small amount of precision vs. the true 19-decimal value).
- **FP16:** π ≈ **3.140625** — noticeably less precise (~6 decimal places), but memory halved (4 bytes → 2 bytes).
- **INT8:** π ≈ **3** — dramatic precision loss, but memory cut to 1 byte (from original 4 bytes).

### 3.5 BFLOAT16 (BF16) — Special Attention
- Developed by **Google Brain**; described as a **"truncated 32-bit float."**
- Hybrid between FP16 and FP32:
 - Uses **full 8 exponent bits** (like FP32) → preserves FP32's **dynamic range**.
 - Truncates the fraction to just **7 bits** (vs. FP32's 23) → sacrifices precision, not range.
- Benefits:
 - Significantly improves **training stability**.
 - Supported by newer GPUs (e.g., **NVIDIA A100**).
 - Speeds up calculations, improving performance.
- Drawback: not well-suited for **integer calculations** — but these are rare in deep learning anyway.
- Many LLMs are pre-trained using BF16, including **FLAN-T5**.

### 3.6 Memory Savings from Quantization (1B parameter model example)
| Precision | Bytes/Param | Memory to Store Weights |
|---|---|---|
| FP32 (full) | 4 | 4 GB |
| FP16 / BF16 (half) | 2 | 2 GB (50% savings) |
| INT8 | 1 | 1 GB (further 50% savings, 75% total) |

- **Important caveat:** Quantization changes *precision*, not parameter count — the model still has the same 1 billion parameters in every case. It only reduces the memory needed to represent each parameter.
- These savings apply similarly to memory needed **during training**, not just storage/inference.

## 4. The Scaling Problem Beyond Quantization

- Quantization alone is not enough for very large models:
 - Models today often have **50–100+ billion parameters**.
 - This can require **up to ~500× more memory** than the 1B-parameter example — i.e., **tens of thousands of gigabytes**.
- Beyond a few billion parameters, it becomes **impossible to train on a single GPU**, regardless of quantization.

## 5. Solution #2: Distributed Computing

- Large-scale training requires **distributing training across multiple GPUs** (potentially **hundreds of GPUs**).
- This is expensive — a major reason why practitioners typically **do not pre-train models from scratch**, and instead reuse existing pre-trained models.
- **Fine-tuning** (covered in a later lecture) also requires storing training parameters in memory, so these memory/scaling considerations remain relevant even when not pre-training from scratch.

## 6. Key Takeaways Summary

1. Training LLMs is memory-intensive because you must store not just weights, but optimizer states, gradients, activations, and temp variables (~6× the weight-only memory).
2. **Quantization** reduces memory by lowering numeric precision (FP32 → FP16/BF16 → INT8), trading off precision for memory savings.
3. **BFLOAT16** is a popular middle-ground: keeps FP32's dynamic range while halving memory, improving stability and speed on modern GPUs (e.g., A100).
4. Quantization gives you memory savings, but does **not** change the number of parameters — for genuinely massive models (50B–100B+ params), even full quantization isn't enough.
5. Beyond a few billion parameters, **distributed training across multiple GPUs** becomes necessary — a costly requirement that motivates using pre-trained models and fine-tuning rather than training from scratch.

## 7. Terms Glossary (Quick Reference)

- **CUDA** – Nvidia's library/tool suite for GPU-accelerated computing.
- **FP32** – 32-bit full-precision floating point (default storage format for model weights).
- **FP16** – 16-bit half-precision floating point.
- **BFLOAT16 (BF16)** – 16-bit float with FP32's exponent range but truncated (7-bit) mantissa.
- **INT8** – 8-bit integer representation.
- **Quantization** – Technique to reduce memory by lowering the numerical precision of model parameters.
- **Quantization-aware training** – Training approach where quantization scaling factors are learned during training itself.
- **Adam optimizer states** – Additional per-parameter memory used by the Adam optimization algorithm during training.
- **Distributed computing (multi-GPU training)** – Splitting model training across multiple GPUs to handle models too large for a single device.
