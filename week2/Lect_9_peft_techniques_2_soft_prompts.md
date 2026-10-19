# Lecture Notes: PEFT Techniques 2 – Soft Prompts

**Week 2:** Fine-tuning and evaluating large language models


---

## 1. Context and Motivation

- With LoRA, the goal was to find an efficient way to update the model's weights without retraining every parameter.
- PEFT also includes **additive methods** that aim to improve model performance **without changing the weights at all**.
- This lecture covers a second parameter-efficient fine-tuning method: **prompt tuning**.

---

## 2. Prompt Tuning vs. Prompt Engineering

Prompt tuning sounds similar to prompt engineering, but the two are quite different.

### Prompt Engineering
- You work on the **language of the prompt** to get the completion you want.
- Can be simple (trying different words or phrases) or more complex (including examples for one-shot or few-shot inference).
- Goal: help the model understand the nature of the task and generate a better completion.
- **Limitations:**
  - Requires a lot of manual effort to write and try different prompts.
  - Limited by the length of the context window.
  - You may still not achieve the performance you need for your task.

### Prompt Tuning
- You **add additional, trainable tokens** to your prompt.
- The **supervised learning process** determines their optimal values.

---

## 3. Soft Prompts

### Definition and Mechanics
- The set of trainable tokens is called a **soft prompt**.
- It is **prepended to the embedding vectors** that represent your input text.
- Soft prompt vectors have the **same length as the embedding vectors** of the language tokens.
- Including roughly **20 to 100 virtual tokens** can be sufficient for good performance.

### Hard Tokens vs. Soft Prompts
| Aspect | Natural language tokens ("hard") | Soft prompts |
|---|---|---|
| Nature | Each corresponds to a **fixed location** in the embedding vector space | **Not fixed, discrete words** of natural language |
| Values | Fixed | **Virtual tokens** that can take on **any value** within the continuous, multidimensional embedding space |

- Through supervised learning, the model learns the values for these virtual tokens that **maximize performance for a given task**.

---

## 4. Full Fine-Tuning vs. Prompt Tuning

| Aspect | Full fine-tuning | Prompt tuning |
|---|---|---|
| Training data | Input prompts and output completions (labels) | (Supervised learning, same style of task data) |
| LLM weights | **Updated** during supervised learning | **Frozen**; the underlying model is not updated |
| What gets updated | Model weights | **Embedding vectors of the soft prompt**, updated over time to optimize the model's completion |
| Trainable parameters | Millions to billions | Only a few |

- Prompt tuning is a **very parameter-efficient strategy** because only a few parameters are trained.

---

## 5. Task Switching and Flexibility

- Similar to LoRA, you can **train a different set of soft prompts for each task** and swap them out at inference time.
- To use them for inference, you **prepend your input prompt with the learned tokens**.
- To switch tasks, you simply **change the soft prompt**.
- Soft prompts are **very small on disk**, making this kind of fine-tuning **extremely efficient and flexible**.
- The **same LLM is used for all tasks**; only the soft prompts are switched at inference time.

---

## 6. Performance of Prompt Tuning

### Source
- Original paper exploring the method by **Brian Lester and collaborators at Google**.
- The authors compared prompt tuning to several other methods across a range of model sizes.

### The Figure
- **X-axis:** model size
- **Y-axis:** SuperGLUE score (the evaluation benchmark covered earlier in the week, which grades model performance on a number of different language tasks)

| Line color | Method |
|---|---|
| Red | Full fine-tuning on a single task |
| Orange | Multi-task fine-tuning |
| Green | Prompt tuning |
| Blue | Prompt engineering only |

### Findings
- Prompt tuning **does not perform as well as full fine-tuning for smaller LLMs**.
- As model size increases, **prompt tuning performance increases too**.
- At around **10 billion parameters**, prompt tuning can be **as effective as full fine-tuning**.
- At that scale it offers a **significant boost in performance over prompt engineering alone**.

---

## 7. Interpretability of Learned Virtual Tokens

- **Potential issue:** soft prompt tokens can take any value in the continuous embedding space, so the trained tokens **do not correspond to any known token, word, or phrase** in the LLM's vocabulary.
- However, **nearest-neighbor analysis** of tokens close to the soft prompt location shows they **form tight semantic clusters**.
  - The words closest to the soft prompt tokens have **similar meanings**.
  - These words usually have some **meaning related to the task**.
  - This suggests the prompts are **learning word-like representations**.

---

## 8. Comparing the Two PEFT Methods Covered

| Method | How it works |
|---|---|
| **LoRA** | Uses **rank decomposition matrices** to update model parameters efficiently |
| **Prompt tuning** | **Trainable tokens are added to the prompt**; model weights are left untouched |

- Both let you fine-tune models with the potential for improved task performance while using **much less compute than full fine-tuning**.
- **LoRA is broadly used in practice** because of its comparable performance to full fine-tuning for many tasks and datasets.
- LoRA is practiced in this week's lab.

---

## 9. Week 2 Recap

1. **Instruction fine-tuning**
   - Adapts a foundation model.
   - Prompt templates and datasets used to train the **FLAN-T5** model were shown.
   - Evaluation metrics and benchmarks such as **ROUGE** and **HELM** measure success during fine-tuning.
   - Proven very effective and useful across a wide range of natural language use cases and tasks.
   - With **just a few hundred examples**, you can fine-tune a model to your specific task.

2. **Parameter-efficient fine-tuning (PEFT)**
   - Reduces the amount of compute required to fine-tune a model.
   - Two methods covered: **LoRA** and **prompt tuning**.

3. **QLoRA**
   - **LoRA combined with the quantization techniques** from week one to further reduce memory footprint.

4. **PEFT in practice**
   - Used heavily to minimize compute and memory resources.
   - Ultimately reduces fine-tuning cost, lets you make the most of your compute budget, and speeds up development.

---

## 10. Key Takeaways (Quick Review)

- Prompt tuning ≠ prompt engineering: tuning **trains** extra tokens; engineering **manually edits** prompt language.
- A **soft prompt** is a set of trainable virtual tokens prepended to input embeddings (typically 20–100 tokens).
- LLM weights stay **frozen**; only soft prompt embeddings are trained.
- Different soft prompts can be swapped in for different tasks using the **same LLM**.
- Prompt tuning matches full fine-tuning at roughly **10B+ parameters** but lags on smaller models.
- Learned virtual tokens aren't real words, but their **nearest neighbors form semantic clusters** related to the task.
- **LoRA** is the more broadly used method in practice; **QLoRA** adds quantization for further memory savings.
