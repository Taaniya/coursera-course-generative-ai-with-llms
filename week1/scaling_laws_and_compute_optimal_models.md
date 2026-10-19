# Lecture Notes: Scaling Laws and Compute-Optimal Models

**Week 1 Module:** Generative AI use cases, project lifecycle, and model pre-training


---

## 1. Overview

This lecture covers research on the relationship between **model size**, **training configuration**, and **performance**, in an effort to determine just how big models need to be.

- The goal of pre-training is to maximize the model's performance on its learning objective, i.e. **minimizing the loss when predicting tokens**.
- Two options to achieve better performance:
  1. Increase the **size of the training dataset**.
  2. Increase the **number of parameters** in the model.
- In theory, you could scale either or both.
- A third consideration is the **compute budget**: the number of GPUs available and the time available for training.

---

## 2. Unit of Compute: Petaflop/s-day

- **Petaflop per second day (pflops/s-day)**: the number of floating point operations performed at a throughput of one petaflop per second, running for an entire day.
- **One petaflop** = one quadrillion floating point operations per second.
- For training transformers, 1 petaflop/s-day is approximately equivalent to:
  - **8 NVIDIA V100 GPUs** at full efficiency for one full day.
  - **2 NVIDIA A100 GPUs** (a more powerful processor needs fewer chips for the same compute).

---

## 3. Scale of Compute Budgets Across Models

A chart compared the petaflop/s-days required to pre-train different model variants:

| Model family | Architecture type |
|---|---|
| BERT, RoBERTa | Encoder-only |
| T5 | Encoder-decoder |
| GPT-3 | Decoder-only |

- Models within each family differ in the **number of parameters**, ranging from a few hundred million (BERT base) to **175 billion** (largest GPT-3 variant).
- The chart's y-axis is **logarithmic**: each vertical increment is a power of 10.
- Key data points:
  - **T5 XL (3B parameters):** close to **100** petaflop/s-days.
  - **GPT-3 (175B parameters):** approximately **3,700** petaflop/s-days.
- **Takeaway:** a huge amount of compute is required to train the largest models. Bigger models take more compute to train and generally also require more data to achieve good performance.

---

## 4. Relationships Between the Three Scaling Choices

There are well-defined relationships between **dataset size, model size, and compute budget**. Researchers have explored the trade-offs among them.

### 4.1 Compute budget vs. performance (OpenAI paper)

- **Y-axis:** test loss (a proxy for model performance; smaller is better).
- **X-axis:** compute budget in petaflop/s-days. Larger values can be achieved by more compute power, longer training, or both.
- Each thin blue line shows the loss over a single training run.
- Looking at where the loss starts to decline more slowly in each run reveals a clear relationship between compute budget and performance.
- This relationship can be approximated by a **power law** (shown as a pink line).

### 4.2 Power law

- A **power law** is a mathematical relationship between two variables where one is proportional to the other raised to some power.
- On a graph with **both axes logarithmic**, power law relationships appear as **straight lines**.
- The relationship holds **as long as model size and training dataset size don't inhibit the training process**.

### 4.3 The practical constraint

- Taken at face value, one could just increase compute budget to improve performance.
- In practice, compute is generally a **hard constraint**, set by:
  - the hardware available,
  - the time available for training,
  - the financial budget of the project.
- With compute **held fixed**, the two remaining levers are **training dataset size** and **number of parameters**.

### 4.4 Dataset size and model size each follow a power law with test loss

OpenAI researchers found that these two quantities also show a power law relationship with test loss when the other two variables are held fixed.

- **Dataset size varied** (compute budget and model size fixed): as training data volume increases, model performance continues to improve.
- **Model size varied** (compute budget and dataset size fixed): as the number of parameters increases, test loss decreases, indicating better performance.

---

## 5. The Chinchilla Paper (Compute-Optimal Models)

**Question:** What is the ideal balance between model size, data size, and compute?

- Research and industry communities have published a lot of empirical data on pre-training compute-optimal models.
- In a **2022 paper**, a group of researchers led by **Jordan Hoffmann, Sebastian Borgeaud, and Arthur Mensch** studied the performance of language models of various sizes and quantities of training data.
- **Goal:** find the optimal number of parameters and volume of training data for a given compute budget.
- The resulting compute-optimal model was named **Chinchilla**, so the paper is often called the "Chinchilla paper."

### 5.1 Key findings

1. **Many ~100B+ parameter LLMs (like GPT-3) may be:**
   - **Over-parameterized**: more parameters than needed for a good understanding of language.
   - **Under-trained**: they would benefit from seeing more training data.
2. The authors **hypothesize** that **smaller models can achieve the same performance as much larger ones if trained on larger datasets**.
3. **Optimal training dataset size ≈ 20× the number of parameters.**
   - Example: a **70B parameter** model (Chinchilla) → ideal dataset of **1.4 trillion tokens** (20 × 70B).
4. Comparison of models in the lecture's table:
   - The **last three models** in the table were trained on datasets **smaller than the Chinchilla-optimal size**, so they may be **under-trained**.
   - **LLaMA** was trained on **1.4 trillion tokens**, close to the Chinchilla-recommended number.
5. The **compute-optimal Chinchilla model outperforms non-compute-optimal models such as GPT-3** on a large range of downstream evaluation tasks.

---

## 6. Implications and Trends

- Following the Chinchilla results, teams have recently started to develop **smaller models that achieve similar or better results** than larger models trained in a non-optimal way.
- Going forward, expect a **deviation from the "bigger is always better" trend** of the last few years as more teams optimize their model design.

### Example: BloombergGPT

- Trained in a **compute-optimal way following the Chinchilla loss**.
- Achieves good performance with a size of **50 billion parameters**.
- An interesting example where **pre-training a model from scratch was necessary** to achieve good task performance (the reason is discussed in the next video of the week).

---

## 7. Key Takeaways (Quick Review)

- Pre-training goal: minimize token-prediction loss.
- Levers: dataset size, model size (parameters), compute budget.
- **1 petaflop/s-day ≈ 8 V100 GPUs ≈ 2 A100 GPUs for one day.**
- Compute needs grow dramatically with model size (T5 XL 3B ≈ ~100 pflops/s-days; GPT-3 175B ≈ ~3,700).
- Compute budget, dataset size, and model size each have a **power-law** relationship with test loss when the others are held fixed.
- Compute is usually a **hard constraint** in practice.
- **Chinchilla:** optimal data ≈ **20× parameters** (70B → 1.4T tokens).
- Large models like GPT-3 may be **over-parameterized and under-trained**.
- Compute-optimal smaller models can match or beat larger non-optimal ones; the trend is moving away from "bigger is always better."
- BloombergGPT (50B) is a compute-optimal example where pre-training from scratch was necessary.
