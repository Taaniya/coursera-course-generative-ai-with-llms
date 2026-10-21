# Lecture notes: Model Optimization for Deployment

**Week 3:** Reinforcement learning and LLM-powered applications


---

## 1. Overview

After adapting and aligning an LLM to your tasks, the next step is integrating the model into applications. This lecture introduces the most important techniques for optimizing a model before deploying it for inference. It is an introduction, not an exhaustive treatment.

---

## 2. Key Questions to Ask Before Deployment

### 2.1 How the LLM will function in deployment
- How fast do you need the model to generate completions?
- What compute budget is available?
- Are you willing to trade off model performance for improved inference speed or lower storage?

### 2.2 Additional resources the model may need
- Will the model interact with external data or other applications?
- If so, how will you connect to those resources?

### 2.3 How the model will be consumed
- What will the intended application or API interface through which the model is consumed look like?

---

## 3. The Inference Challenge

- LLMs create inference challenges in:
  - Computing requirements
  - Storage requirements
  - Ensuring low latency for consuming applications
- These challenges persist whether deploying **on-premises** or **to the cloud**.
- They become even more of an issue when deploying to **edge devices**.
- One of the primary ways to improve application performance is to **reduce the size of the LLM**:
  - A smaller model loads more quickly, which reduces inference latency.
  - The challenge is reducing size **while still maintaining model performance**.
- Some techniques work better than others for generative models, and there are tradeoffs between accuracy and performance.

### Three techniques covered
1. **Distillation**
2. **Post-training quantization (PTQ)**
3. **Model pruning**

---

## 4. Technique 1: Model Distillation

### 4.1 Definition
- A larger **teacher model** trains a smaller **student model**.
- The student statistically mimics the teacher's behavior, either:
  - just in the final prediction layer, or
  - in the model's hidden layers as well.
- The lecture focuses on the first option (final prediction layer).
- The smaller student model is then used for inference, lowering storage and compute budget.

### 4.2 The Process
1. Start with your **fine-tuned LLM as the teacher model**; create a smaller LLM as the **student model**.
2. **Freeze the teacher model's weights** and use it to generate completions for your training data.
3. At the same time, generate completions for the training data using the student model.
4. Knowledge distillation is achieved by minimizing a loss function called the **distillation loss**.

### 4.3 Distillation Loss and the Temperature Trick
- The distillation loss uses the **probability distribution over tokens** produced by the teacher's **softmax layer**.
- Problem: the teacher is already fine-tuned on the training data, so its probability distribution likely closely matches the ground truth and has little variation in tokens.
- Solution: add a **temperature parameter** to the softmax function.
  - A higher temperature increases the creativity of the generated language (as learned in lesson 1).
  - With temperature **greater than 1**, the probability distribution becomes **broader and less strongly peaked**.
  - This softer distribution provides a set of tokens similar to the ground truth tokens.

### 4.4 Terminology

| Term | Meaning |
|---|---|
| **Soft labels** | The teacher model's output (computed with temperature) |
| **Soft predictions** | The student model's predictions (computed with temperature) |
| **Hard predictions** | The student model's outputs using the standard softmax (no temperature variation) |
| **Hard labels** | The ground truth training data labels |
| **Distillation loss** | Loss between soft labels and soft predictions |
| **Student loss** | Loss between hard predictions and hard labels |

### 4.5 Training the Student
- In parallel with the above, train the student to generate correct predictions based on the ground truth training data.
- For this part, **do not vary the temperature**; use the standard softmax function.
- The **combined distillation loss and student loss** are used to update the student's weights via **backpropagation**.

### 4.6 Benefits and Limitations
- **Key benefit:** the smaller student model can be used for inference in deployment instead of the teacher model.
- **Effectiveness:**
  - In practice, distillation is **not as effective for generative decoder models**.
  - It is typically **more effective for encoder-only models**, such as **BERT**, which have a lot of representation redundancy.
- **Important note:** distillation trains a **second, smaller model** for inference. It does **not reduce the size of the initial LLM** in any way.

---

## 5. Technique 2: Post-Training Quantization (PTQ)

### 5.1 Background
- Quantization was introduced in Week 1 via **quantization-aware training (QAT)**.
- After a model is trained, you can perform **post-training quantization (PTQ)** to optimize it for deployment.

### 5.2 What PTQ Does
- Transforms a model's weights to a **lower-precision representation**, such as:
  - 16-bit floating point
  - 8-bit integer
- Benefits: reduces the model size, memory footprint, and compute resources needed for model serving.
- Unlike distillation, this actually reduces the size of the LLM.

### 5.3 Scope of Quantization
- Can be applied to:
  - just the **model weights**, or
  - both **weights and activation layers**.
- In general, approaches that **include activations** can have a **higher impact on model performance**.

### 5.4 Calibration
- Quantization requires an extra **calibration step** to statistically capture the **dynamic range** of the original parameter values.

### 5.5 Tradeoffs
- Quantization sometimes results in a **small percentage reduction** in model evaluation metrics.
- That reduction can often be **worth the cost savings and performance gains**.

---

## 6. Technique 3: Model Pruning

### 6.1 Goal
- Reduce model size for inference by **eliminating weights that contribute little to overall model performance**.
- These are weights with values **very close to or equal to zero**.

### 6.2 Categories of Pruning Methods
- Methods that require **full retraining** of the model.
- Methods that fall under **parameter-efficient fine-tuning (PEFT)**, such as **LoRA**.
- Methods that focus on **post-training pruning**.

### 6.3 Theory vs. Practice
- **In theory:** pruning reduces model size and improves performance.
- **In practice:** there may not be much impact on size and performance if only a **small percentage of the model weights are close to zero**.

---

## 7. Comparison Summary

| Technique | Core idea | Reduces size of the original LLM? | Notes |
|---|---|---|---|
| **Distillation** | Teacher model trains a smaller student model | No (creates a second, smaller model) | Less effective for generative decoder models; more effective for encoder-only models like BERT |
| **Post-training quantization** | Convert weights (and optionally activations) to lower precision (e.g., FP16, INT8) | Yes | Needs a calibration step; including activations can affect performance more; small metric drop possible |
| **Pruning** | Remove weights close to or equal to zero | Yes | May have little impact if few weights are near zero |

---

## 8. Key Takeaways

- Before deployment, consider **inference speed, compute budget, performance tradeoffs, external resource integration, and how the model will be consumed**.
- LLM inference poses challenges in compute, storage, and latency, especially on edge devices.
- **Quantization, distillation, and pruning** all aim to reduce model size to improve performance during inference **without impacting accuracy**.
- Optimizing your model for deployment helps ensure your application functions well and gives users the best possible experience.

---

## 9. Quick Review Questions

1. What three sets of questions should you ask when integrating an LLM into an application?
2. Why does reducing model size reduce inference latency?
3. What roles do the teacher and student play in distillation?
4. Why is a temperature parameter added to the softmax function during distillation?
5. What is the difference between soft labels/predictions and hard labels/predictions?
6. Why is distillation less effective for generative decoder models than for encoder-only models like BERT?
7. Does distillation reduce the size of the original LLM? Why or why not?
8. What is the difference between QAT and PTQ?
9. Why does quantization need a calibration step?
10. Which weights does pruning target, and why might pruning have limited practical impact?
