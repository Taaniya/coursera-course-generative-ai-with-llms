# Lecture Notes: Parameter-Efficient Fine-Tuning (PEFT)
**Week 2:** Fine-tuning and evaluating large language models


---

## 1. The Problem: Full Fine-Tuning Is Expensive

- Training LLMs is computationally intensive (as seen in Week 1).
- Full fine-tuning requires memory not only for the model itself but also for other parameters needed during training.
- Model weights for the largest models are on the order of **hundreds of gigabytes**.
- Additional memory must be allocated for:
  - Optimizer states
  - Gradients
  - Forward activations
  - Temporary memory throughout the training process
- These additional components can be **many times larger than the model** and can quickly become too large for consumer hardware.

### Storage problem with multiple tasks
- Full fine-tuning produces a **new version of the model for every task** you train on.
- Each version is the **same size as the original model**.
- Fine-tuning for multiple tasks can therefore create an **expensive storage problem**.

---

## 2. What PEFT Is

- **Full fine-tuning:** every model weight is updated during supervised learning.
- **PEFT:** only a **small subset of parameters** is updated.
- Two broad ways PEFT achieves this:
  1. **Freeze most of the original weights** and fine-tune only a subset of existing parameters (e.g., particular layers or components).
  2. **Leave the original weights untouched** and add a small number of **new parameters or layers**, fine-tuning only those new components.
- With PEFT, most if not all of the LLM weights are kept **frozen**.

---

## 3. Benefits of PEFT

| Benefit | Detail |
|---|---|
| **Fewer trained parameters** | Much smaller than the original LLM's parameter count; in some cases just 15–20% of the original LLM weights |
| **Manageable memory requirements** | Training memory needs become much more manageable |
| **Single-GPU training** | PEFT can often be performed on a single GPU |
| **Less catastrophic forgetting** | Because the original LLM is only slightly modified or left unchanged, PEFT is less prone to the catastrophic forgetting seen in full fine-tuning |
| **Small storage footprint** | Only a small number of weights are trained, resulting in a much smaller footprint overall, as small as **megabytes** depending on the task |
| **Easy multi-task adaptation** | PEFT weights are trained per task and can be easily swapped out for inference |

### How PEFT works at inference
- The new (PEFT) parameters are **combined with the original LLM weights** for inference.
- Task-specific PEFT weights can be **swapped** in and out, allowing efficient adaptation of the original model to multiple tasks.

---

## 4. Trade-offs Across PEFT Methods

There are several PEFT methods, each with trade-offs in:

- Parameter efficiency
- Memory efficiency
- Training speed
- Model quality
- Inference costs

---

## 5. Three Main Classes of PEFT Methods

### 5.1 Selective Methods
- Fine-tune **only a subset of the original LLM parameters**.
- Several approaches exist to decide which parameters to update. You can train:
  - Only certain **components** of the model
  - Specific **layers**
  - Even individual **parameter types**
- **Research findings:** performance of these methods is **mixed**, with significant trade-offs between parameter efficiency and compute efficiency.
- **Course scope:** the course **does not focus** on selective methods.

### 5.2 Reparameterization Methods
- Also work with the **original LLM parameters**.
- **Reduce the number of parameters to train** by creating **new low-rank transformations** of the original network weights.
- A commonly used technique of this type is **LoRA**, explored in detail in the next video.

### 5.3 Additive Methods
- Carry out fine-tuning by keeping **all original LLM weights frozen** and **introducing new trainable components**.
- Two main approaches:

#### a) Adapter Methods
- Add **new trainable layers** to the model architecture.
- Typically placed **inside the encoder or decoder components**, **after the attention or feedforward layers**.

#### b) Soft Prompt Methods
- Keep the model architecture **fixed and frozen**.
- Focus on **manipulating the input** to achieve better performance.
- Can be done by:
  - Adding **trainable parameters to the prompt embeddings**, or
  - Keeping the input fixed and **retraining the embedding weights**.
- A specific soft prompt technique covered in this lesson: **prompt tuning**.

---

## 6. Summary Comparison of the Three Classes

| Class | Original Weights | What Gets Trained | Examples Mentioned |
|---|---|---|---|
| **Selective** | Mostly frozen | Subset of original parameters (components, layers, parameter types) | Not covered in depth |
| **Reparameterization** | Original parameters used | New low-rank transformations of the original weights | LoRA |
| **Additive** | All frozen | New trainable components | Adapters, Soft prompts (prompt tuning) |

---

## 7. Key Points to Remember

1. Full fine-tuning's memory demands go well beyond just storing the model (optimizer states, gradients, activations, temporary memory).
2. PEFT updates only a small subset of parameters (or newly added ones), keeping most or all original LLM weights frozen.
3. PEFT can often run on a single GPU and reduces the risk of catastrophic forgetting.
4. PEFT weights can be as small as megabytes and can be swapped per task, avoiding the cost of storing a full model copy for each task.
5. The three main classes are **selective**, **reparameterization**, and **additive** methods, each with trade-offs in parameter efficiency, memory efficiency, training speed, model quality, and inference costs.
6. Selective methods show mixed results and are not a focus of the course.

---

## 8. What Comes Next

- **Next video:** LoRA (a reparameterization method) in detail, including how it reduces the memory required for training.
- **Later in this lesson:** prompt tuning (a soft prompt technique).
