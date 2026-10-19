# Lecture Notes: PEFT Techniques 1 – LoRA (Low-Rank Adaptation)

**Week 2:** Fine-tuning and evaluating large language models

---

## 1. What is LoRA?

- LoRA (Low-Rank Adaptation) is a **parameter-efficient fine-tuning (PEFT)** technique.
- It belongs to the **re-parameterization** category of PEFT methods.

---

## 2. Background: Where LoRA Fits in the Transformer

- The input prompt is turned into **tokens**, converted to **embedding vectors**, and passed into the encoder and/or decoder parts of the transformer.
- Both encoder and decoder contain two kinds of neural networks:
  - **Self-attention** networks
  - **Feed-forward** networks
- The weights of these networks are learned during **pre-training**.
- After embedding vectors are created, they are fed into the self-attention layers, where a series of weights is applied to calculate **attention scores**.
- In **full fine-tuning**, *every parameter* in these layers is updated.

---

## 3. How LoRA Works

1. **Freeze** all of the original model parameters.
2. **Inject a pair of rank decomposition matrices** alongside the original weights.
3. Set the dimensions of the smaller matrices so that **their product has the same dimensions as the weights they modify**.
4. **Train only the small matrices** using the same supervised learning process used in full fine-tuning (covered earlier in the week).

### Inference with LoRA

1. Multiply the two low-rank matrices together → produces a matrix with the **same dimensions as the frozen weights**.
2. **Add** this product to the original weights.
3. **Replace** the original weights in the model with these updated values.
4. The result is a **LoRA fine-tuned model** that can carry out the specific task.

### Impact on inference latency

- The updated model has the **same number of parameters as the original**.
- Therefore there is **little to no impact on inference latency**.

---

## 4. Where to Apply LoRA

- Researchers found that applying LoRA to **just the self-attention layers** is often enough to fine-tune for a task and achieve performance gains.
- In principle, LoRA can also be applied to other components, such as the **feed-forward layers**.
- Since **most of the parameters in LLMs are in the attention layers**, applying LoRA to the attention weight matrices gives the **biggest savings in trainable parameters**.

---

## 5. Worked Example: Parameter Savings

Based on the transformer in the *Attention Is All You Need* paper:

| Item | Dimensions | Trainable Parameters |
|---|---|---|
| Original weights matrix | 512 × 64 | **32,768** |
| LoRA matrix A (rank = 8) | 8 × 64 | 512 |
| LoRA matrix B (rank = 8) | 512 × 8 | 4,096 |
| **LoRA total (A + B)** | – | **4,608** |

- Training 4,608 parameters instead of 32,768 is an **86% reduction**.

---

## 6. Practical Benefits of LoRA

### 6.1 Reduced compute requirements
- Significantly fewer trainable parameters means LoRA fine-tuning can often be done on a **single GPU**.
- This avoids the need for a **distributed cluster of GPUs**.

### 6.2 Task switching with small, swappable matrices
- Because the rank decomposition matrices are small, you can **train a different set for each task** and **switch them at inference time** by updating the weights.
- Example workflow:
  - **Task A:** Train a pair of LoRA matrices → multiply them → add the result to the original frozen weights → replace the original weights in the model → run inference on Task A.
  - **Task B:** Take the LoRA matrices trained for Task B → calculate their product → add to the original weights → update the model again.
- The memory needed to store LoRA matrices is **very small**.
- In principle, you can train for **many tasks**, switch weights as needed, and **avoid storing multiple full-size copies of the LLM**.

---

## 7. Performance Evaluation (ROUGE Metric)

**Setup:**
- Model: **FLAN-T5** fine-tuned for **dialogue summarization**.
- The FLAN-T5 base model has already had an initial round of full fine-tuning on a large instruction dataset.
- Metric: **ROUGE** (higher = better). The lecture focuses on **ROUGE-1**, though other ROUGE scores could be used.

**Results comparison:**

| Model | ROUGE-1 Change vs. Base | Notes |
|---|---|---|
| FLAN-T5 base model | Baseline | Scores are fairly low |
| Full fine-tuning (dialogue summarization) | **+0.19** | Updates every weight; much higher ROUGE-1 |
| LoRA fine-tuning | **+0.17** | Slightly lower than full fine-tuning, but not by much |

**Key takeaways:**
- Even though FLAN-T5 is a capable model, it can still benefit from additional fine-tuning on specific tasks.
- LoRA gives a **big performance boost** over the baseline.
- LoRA trains a **much smaller number of parameters** and uses **significantly less compute** than full fine-tuning.
- The small trade-off in performance (0.19 vs. 0.17) **may well be worth it**.

---

## 8. Choosing the Rank of LoRA Matrices

- Choosing the rank is a good question and **still an active area of research**.
- In principle: **smaller rank → fewer trainable parameters → bigger compute savings**.
- However, there are **model performance issues** to consider.

### Findings from the original LoRA paper (Microsoft researchers)
- They explored how different ranks affect performance on **language generation tasks**.
- The results table includes: rank of LoRA matrices, final loss value, and scores for metrics including **BLEU** and **ROUGE**. Bold values indicate the best score per metric.
- The authors found a **plateau in the loss value for ranks greater than 16** — larger LoRA matrices did not improve performance.
- **Takeaway:** Ranks in the range of **4 to 32** offer a good trade-off between reducing trainable parameters and preserving performance.
- Optimizing the choice of rank is an ongoing research area, and **best practices may evolve** as more practitioners use LoRA.

---

## 9. Summary & Wrap-up

- LoRA is a **powerful fine-tuning method** that achieves great performance.
- The principles behind LoRA are useful **not just for training LLMs, but for models in other domains**.
- **Up next:** The final PEFT method of the week **does not change the LLM at all** and instead **focuses on training the input text**.

---

## Quick Revision Checklist

- [ ] LoRA = re-parameterization PEFT technique
- [ ] Freeze original weights; inject two small rank decomposition matrices (A and B)
- [ ] Product of A and B matches the dimensions of the original weights
- [ ] At inference: multiply A × B, add to original weights, replace in model
- [ ] Same parameter count as original → little to no added inference latency
- [ ] Often applied only to self-attention layers (where most parameters live)
- [ ] Example: 32,768 → 4,608 trainable parameters (86% reduction) at rank 8
- [ ] Can often be trained on a single GPU
- [ ] Swap small LoRA matrices per task; no need to store multiple full LLM copies
- [ ] FLAN-T5 ROUGE-1: full fine-tuning +0.19, LoRA +0.17 over baseline
- [ ] Rank: active research area; plateau above 16; 4–32 is a good range
