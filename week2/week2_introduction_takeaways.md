
## Week 2: Fine-tuning and Evaluating Large Language Models
### Lecture: Introduction — Study Notes

---

## 1. Context and Recap

- **Last week** covered:
  - Transformer networks, described as a key foundation for large language models (LLMs).
  - The generative AI project lifecycle.
- **This week** covers:
  - Instruction tuning (instruction fine-tuning) of LLMs.
  - How to carry out fine-tuning in an efficient way.
- Instructors for the week: Mike and Shelby.

---

## 2. Instruction Fine-Tuning

### 2.1 Why it is needed
- A **base model** is the initially pre-trained model.
- It has encoded a lot of good information, usually about the world, so it *knows about things*.
- However, it does **not necessarily know how to respond to prompts or questions**. When instructed to do a certain task, it may not know how to respond.
- **Instruction fine-tuning** changes the model's behavior so it becomes more helpful to the user.

### 2.2 Why it was a major breakthrough
- Pre-training on general text from the internet and other sources teaches the model to **predict the next word**.
- **Predicting the next word on the internet is not the same as following instructions.**
- The notable result: take an LLM trained on **hundreds of billions of words** from the internet, then fine-tune it with a **much smaller dataset** on following instructions, and it learns to follow instructions.
- It was described as one of the major breakthroughs in the history of LLMs.

### 2.3 Catastrophic forgetting
- **Definition (as described):** When the model is trained on some extra data during instruction fine-tuning, it **forgets** much or a big chunk of the knowledge it had before.
- It is something to **watch out for**, and is covered later in the course.
- **Technique mentioned to combat it:** perform instruction fine-tuning across a **really broad range of different instruction types**.
  - It is not enough to tune only on the single thing you want the model to do; you may need to be **broader** than that.

---

## 3. Two Types of Fine-Tuning Worth Doing

1. **Instruction fine-tuning** (discussed above).
2. **Fine-tuning for a specific application:** when a developer fine-tunes a model for their own, specialized application.

---

## 4. The Problem with Full Fine-Tuning

- Full fine-tuning means taking a giant model and **fine-tuning every single parameter**.
- Consequences:
  - You have a big model to **store and deploy**.
  - It is very **compute-expensive and memory-expensive**.
  - It can be **cost-prohibitive**.

---

## 5. Parameter Efficient Fine-Tuning (PEFT)

### 5.1 What it is
- **Parameter efficient fine-tuning (PEFT)** is a set of methods that mitigate the concerns of full fine-tuning.

### 5.2 Key benefits
- Can achieve **similar performance** to full fine-tuning on a lot of tasks.
- Allows techniques that:
  - **Freeze the original model weights**, or
  - **Add adaptive layers on top** of the model,
  - ...with a **much smaller memory footprint**.
- This makes it possible to **train for multiple tasks**.
- Puts fine-tuning of generative AI models in the hands of **everyday, cost-conscious users** who have cost constraints.

### 5.3 LoRA
- **LoRA** is one specific PEFT technique mentioned.
- Uses **low-rank matrices** rather than full fine-tuning.
- Generates a lot of excitement and demand because of its performance results.
- Delivers **good performance with minimal compute and memory requirements**.

---

## 6. Practical Developer Perspectives

### 6.1 Prompting vs. fine-tuning
- Many developers **start with prompting**.
  - Sometimes it gives **good enough performance**, which is fine.
  - Sometimes **prompting hits a performance ceiling**.
- When prompting hits the ceiling, fine-tuning with **LoRA or other PEFT techniques** is described as **critical for unlocking that extra level of performance**.

### 6.2 Cost debate: giant model vs. smaller fine-tuned model
- Developers debate the **cost of using a giant model** (which has many benefits) versus **fine-tuning a smaller model** for their application.

### 6.3 Data control and model size
- If you are concerned about **where your data is going**, the model may need to **run in your control**.
- In that case, having a model of an **appropriate size** is important.

### 6.4 Customer demand
- Many customers want to tune for **very specific tasks and domains**.
- PEFT lets them achieve similar results to full fine-tuning on many tasks.

---

## 7. Key Terms Quick Reference

| Term | Meaning (per lecture) |
|---|---|
| Base model | The initially pre-trained model; holds knowledge about the world but may not respond well to prompts |
| Instruction fine-tuning | Fine-tuning that changes model behavior to be more helpful in following instructions |
| Catastrophic forgetting | Model forgets much of its prior knowledge after training on extra data |
| Full fine-tuning | Fine-tuning every parameter; compute, memory, and cost intensive |
| PEFT | Parameter efficient fine-tuning; methods to reduce the cost of fine-tuning |
| LoRA | A PEFT technique using low-rank matrices; strong performance with minimal compute/memory |

---

## 8. Main Takeaways

1. Pre-training teaches next-word prediction, which is **not** the same as following instructions; **instruction fine-tuning** bridges that gap.
2. A **much smaller dataset** is enough to fine-tune a large pre-trained model to follow instructions.
3. Beware of **catastrophic forgetting**; use a **broad range of instruction types** to help combat it.
4. **Full fine-tuning** is expensive in compute, memory, storage, and deployment.
5. **PEFT** (e.g., **LoRA**) achieves similar performance with a much smaller footprint and supports multi-task training.
6. Developers often **start with prompting**, then turn to **PEFT/LoRA** when prompting hits its performance ceiling.
7. Model **size and cost** decisions matter, including when **data must stay under your control**.

---

## 9. What Comes Next

- The next video: Mike begins with **instruction fine-tuning** in detail.
