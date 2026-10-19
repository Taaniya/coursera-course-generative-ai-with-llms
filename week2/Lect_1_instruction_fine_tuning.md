# Instruction Fine-Tuning: Study Notes

**Course:** Generative AI with Large Language Models
**Week 2:** Fine-tuning and evaluating large language models
**Lecture:** Instruction fine-tuning

---

## 1. Lecture Overview

- Week 1 introduced the Generative AI Project Lifecycle, example LLM use cases, and the kinds of tasks LLMs can carry out.
- This lesson covers:
  - Methods to improve the performance of an existing model for a specific use case.
  - Important metrics to evaluate a fine-tuned LLM and quantify its improvement over the base model.
- The lecture starts with how to fine-tune an LLM with **instruction prompts**.

---

## 2. Motivation: Limits of Prompt-Based Approaches

### Recap of in-context approaches
- Some models can identify instructions in a prompt and carry out **zero-shot inference** correctly.
- Others, such as **smaller LLMs**, may fail to carry out the task.
- Including one or more examples in the prompt (**one-shot / few-shot inference**) can help the model identify the task and generate a good completion.

### Two drawbacks of one-/few-shot inference
1. **Unreliable for smaller models:** it doesn't always work, even when five or six examples are included.
2. **Consumes context window:** examples in the prompt take up valuable space, reducing room for other useful information.

### The alternative
- **Fine-tuning:** a process that further trains a base model.

---

## 3. Pre-training vs. Fine-tuning

| Aspect | Pre-training | Fine-tuning |
|---|---|---|
| Data | Vast amounts of unstructured textual data | Dataset of labelled examples |
| Learning type | Self-supervised learning | Supervised learning |
| Goal | Train the LLM | Update the LLM's weights to improve completions for a specific task |

- The labelled examples are **prompt-completion pairs**.
- Fine-tuning extends the model's training to improve its ability to generate good completions for a specific task.

---

## 4. What Is Instruction Fine-Tuning?

- **Instruction fine-tuning** is one fine-tuning strategy that is particularly good at improving a model's performance on a **variety of tasks**.
- It trains the model with examples that demonstrate **how it should respond to a specific instruction**.
- The training dataset contains many prompt-completion pairs for the task of interest, **each including an instruction**.

### Examples of instructions by task
- **Summarisation:** examples beginning with "Summarise the following text" or a similar phrase.
- **Translation:** examples with instructions like "Translate this sentence".
- These pairs let the model learn to generate responses that **follow the given instructions**.

### Full fine-tuning
- Instruction fine-tuning where **all of the model's weights are updated** is called **full fine-tuning**.
- It produces a **new version of the model with updated weights**.
- Like pre-training, it needs enough **memory and compute budget** to store and process all the **gradients, optimisers and other components** updated during training.
- You can benefit from the **memory optimisation and parallel computing strategies** learned in the previous week.

---

## 5. Step-by-Step: How to Instruction Fine-Tune an LLM

### Step 1: Prepare the training data

- Many publicly available datasets were used to train earlier generations of language models, but **most are not formatted as instructions**.
- Developers have assembled **prompt-template libraries** that convert existing datasets into **instruction prompt datasets** for fine-tuning.
  - Example dataset: the large dataset of **Amazon product reviews**.
- Prompt-template libraries include many templates for different tasks and datasets.

#### Template examples for the Amazon Reviews dataset
Three prompts were shown, usable for fine-tuning models for:
- **Classification**
- **Text generation**
- **Text summarisation**

How the templates work:
- The original review (called **ReviewBody**) is passed into the template.
- It is inserted into text that starts with an instruction, such as:
  - "Predict the associated rating"
  - "Generate a star review"
  - "Give a short sentence describing the following product review"
- Result: a prompt containing **both an instruction and the example from the dataset**.

### Step 2: Split the dataset
- As in standard supervised learning, divide the instruction dataset into:
  - **Training** split
  - **Validation** split
  - **Test** split

### Step 3: Run the fine-tuning loop
1. Select prompts from the training dataset and pass them to the LLM, which **generates completions**.
2. **Compare** the LLM completion with the response specified in the training data.
   - Lecture example: the model classified a clearly very positive review as **"neutral"**, which was an understatement, showing the model didn't do a great job.
3. Compute the **loss**:
   - The LLM's output is a **probability distribution across tokens**.
   - Compare the distribution of the completion with that of the training label.
   - Use the standard **cross-entropy function** to calculate loss between the two token distributions.
4. Use the loss to **update model weights via standard backpropagation**.
5. Repeat over **many batches** of prompt-completion pairs and **several epochs**, so performance on the task improves.

### Step 4: Evaluate
- **Validation:** define separate evaluation steps using the **holdout validation dataset** to measure LLM performance. This gives the **validation accuracy**.
- **Final evaluation:** after fine-tuning completes, use the **holdout test dataset**. This gives the **test accuracy**.

---

## 6. Outcome

- Fine-tuning yields a new version of the base model, often called an **instruct model**, that is better at the tasks you are interested in.
- Fine-tuning with instruction prompts is the **most common way to fine-tune LLMs** today.
- **Convention for the rest of the course:** the term "fine-tuning" is assumed to always mean **instruction fine-tuning**.

---

## 7. Key Takeaways (Quick Review)

1. Zero-shot works for some models; smaller LLMs may fail.
2. One-/few-shot prompting has two drawbacks: unreliable for small models, and it uses up context window space.
3. Fine-tuning is **supervised** learning on **labelled prompt-completion pairs**, unlike self-supervised pre-training on unstructured text.
4. **Instruction fine-tuning** trains on examples that pair an instruction with the desired response, improving performance across a variety of tasks.
5. **Full fine-tuning** updates all weights and requires memory and compute for gradients, optimisers and other training components.
6. **Prompt-template libraries** convert existing datasets (e.g., Amazon reviews) into instruction datasets.
7. Split data into **training / validation / test**.
8. Training loop: generate completion → compare to label → **cross-entropy loss** over token distributions → **backpropagation** → repeat across batches and epochs.
9. Measure **validation accuracy** during training and **test accuracy** at the end.
10. The result is an **instruct model**; "fine-tuning" henceforth means instruction fine-tuning.
