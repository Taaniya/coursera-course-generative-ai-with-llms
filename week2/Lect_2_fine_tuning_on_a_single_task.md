# Lecture Notes: Fine-tuning on a Single Task

**Week 2:** Fine-tuning and evaluating large language models


---

## 1. Overview

- LLMs are famous for performing many different language tasks within a single model.
- However, an application may only need to perform a **single task**.
- In that case, a pre-trained model can be **fine-tuned to improve performance only on the task of interest**.
- Example task: **summarization**, using a dataset of examples for that task.

---

## 2. Data Requirements for Single-Task Fine-tuning

- Good results can be achieved with **relatively few examples**.
- Often **500 to 1,000 examples** can result in good performance.
- This contrasts with the **billions of pieces of text** the model saw during pre-training.

---

## 3. Potential Downside: Catastrophic Forgetting

### Definition
- Fine-tuning on a single task may lead to a phenomenon called **catastrophic forgetting**.

### Why It Happens
- **Full fine-tuning modifies the weights of the original LLM.**
- This leads to great performance on the single fine-tuning task, but it can **degrade performance on other tasks**.

### Example from the Lecture
- Fine-tuning can improve a model's ability to perform **sentiment analysis** on a review, producing a quality completion.
- But the model may **forget how to do other tasks**.
- Before fine-tuning, the model could carry out **named entity recognition (NER)**, correctly identifying **"Charlie" as the name of the cat** in a sentence.
- After fine-tuning, the model **can no longer carry out this task**:
  - It confuses the entity it is supposed to identify.
  - It exhibits behavior related to the **new task** (sentiment analysis).

---

## 4. Options to Avoid Catastrophic Forgetting

### Option 0: Decide Whether It Actually Matters for Your Use Case
- First, decide whether catastrophic forgetting actually impacts your use case.
- If all you need is **reliable performance on the single task** you fine-tuned on, it may not be an issue that the model can't generalize to other tasks.

### Option 1: Multi-Task Fine-tuning
- If you want or need the model to **maintain its multi-task generalized capabilities**, fine-tune on **multiple tasks at one time**.
- Good multi-task fine-tuning may require **50 to 100,000 examples across many tasks**.
- This requires **more data and more compute** to train.
- The lecture says this option will be discussed in more detail shortly (next video).

### Option 2: Parameter Efficient Fine-Tuning (PEFT)
- Perform PEFT **instead of full fine-tuning**.
- PEFT is a **set of techniques** that:
  - **Preserves the weights of the original LLM**.
  - Trains only a **small number of task-specific adapter layers and parameters**.
- PEFT shows **greater robustness to catastrophic forgetting**, since most of the pre-trained weights are left unchanged.
- PEFT is described as an **exciting and active area of research**, covered later in the week.

---

## 5. Summary Comparison

| Aspect | Single-task full fine-tuning | Multi-task fine-tuning | PEFT |
|---|---|---|---|
| Goal | Best performance on one task | Maintain multi-task generalized capabilities | Avoid catastrophic forgetting by keeping original weights |
| Data needed | ~500–1,000 examples | ~50–100,000 examples across many tasks | Not specified in this lecture |
| Effect on original weights | Modified | Not specified in this lecture | Preserved; only small task-specific adapter layers/parameters trained |
| Risk of catastrophic forgetting | Yes | Addressed by training on multiple tasks | More robust (most pre-trained weights unchanged) |
| Cost | Lower data need | More data and compute | Not specified in this lecture |

---

## 6. Key Takeaways

1. If only one task matters, fine-tune a pre-trained model on that task.
2. Only about 500–1,000 examples can give good performance for a single task.
3. Full fine-tuning changes the original LLM's weights, which can cause **catastrophic forgetting** on other tasks.
4. Catastrophic forgetting is only a problem if you need the model to generalize beyond the fine-tuned task.
5. To retain multi-task ability: use **multi-task fine-tuning** (needs 50–100,000 examples across tasks, more data and compute).
6. Alternatively: use **PEFT**, which preserves original weights and trains only small task-specific adapter layers/parameters, making it more robust to forgetting.

---

## 7. Study Questions

1. Why might a model need only 500–1,000 examples for single-task fine-tuning, compared with pre-training data?
2. What is catastrophic forgetting, and why does full fine-tuning cause it?
3. In the lecture's example, what ability did the model lose after sentiment-analysis fine-tuning?
4. When might catastrophic forgetting not be a concern?
5. What are the two options for maintaining multi-task capabilities, and how do they differ?
6. Why is PEFT more robust to catastrophic forgetting?
