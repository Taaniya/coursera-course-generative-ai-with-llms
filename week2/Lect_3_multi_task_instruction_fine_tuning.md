# Multi-Task Instruction Fine-Tuning: Lecture Notes

**Week 2:** Fine-tuning and evaluating large language models


---

## 1. Multi-Task Fine-Tuning

### Definition
- Multi-task fine-tuning is an **extension of single-task fine-tuning**.
- The training dataset is made up of **example inputs and outputs for multiple tasks**.

### Example tasks in the mixed dataset
- Summarization
- Review rating
- Code translation
- Entity recognition

### How it works
- The model is trained on this **mixed dataset** so its performance improves **on all tasks simultaneously**.
- Over many epochs of training, the **losses calculated across examples** are used to **update the model's weights**.
- The result is an **instruction-tuned model** that has learned to be good at many different tasks at the same time.

### Key benefit
- It **avoids the issue of catastrophic forgetting**.

### Drawback
- It **requires a lot of data**: as many as **50,000 to 100,000 examples** in the training set.
- The lecture says the effort of assembling this data can be **worthwhile**, because the resulting models are often **very capable** and suitable where **good performance at many tasks** is desirable.

---

## 2. The FLAN Family of Models

### What FLAN is
- Instruct model variants differ based on the **datasets and tasks used during fine-tuning**.
- **FLAN** stands for **Fine-Tuned Language Net**.
- It is a **specific set of instructions used to fine-tune different models**.
- FLAN fine-tuning is the **last step of the training process**, so the original paper's authors called it the **"metaphorical dessert to the main course of pre-training."**

### FLAN variants
| Variant | Description |
|---|---|
| **FLAN-T5** | The FLAN-instruct version of the **T5** foundation model |
| **FLAN-PaLM** | The FLAN-instruct version of the **PaLM** foundation model |

### FLAN-T5
- Described as a **great general-purpose instruct model**.
- Fine-tuned on **473 datasets** across **146 task categories**.
- The datasets were **chosen from other models and papers**.
- The original paper is available through a reading exercise after the video.

---

## 3. Example Dataset: SAMSum (Dialogue Summarization)

### Overview
- SAMSum is one example of a prompt dataset used for **summarization tasks in FLAN-T5**.
- It is part of the **MUFFIN** collection of tasks and datasets.
- It is used to train language models to **summarize dialogue**.
- Contains **16,000 messenger-like conversations with summaries**.

### How the dataset was created
- **Linguists** crafted the dialogues and summaries, for the express purpose of producing a **high-quality training dataset** for language models.
- Linguists were asked to create conversations **similar to those they write daily**, reflecting the **proportion of topics** in their real-life messenger conversations.
- **Other language experts** then wrote **short summaries** that included **important pieces of information** and the **names of the people** in the dialogue.

### Prompt template for SAMSum
- The template contains **several different instructions that all ask the model to do the same thing: summarize a dialogue**.
- Example phrasings from the lecture:
  - "Briefly summarize that dialogue."
  - "What is a summary of this dialogue?"
  - "What was going on in that conversation?"
- **Including different ways of saying the same instruction helps the model generalize and perform better.**
- As with the prompt templates seen earlier:
  - The **dialogue** from the dataset is inserted wherever the **dialogue field** appears in the template.
  - The **summary** is used as the **label**.
- After applying the template to **each row** of the dataset, it can be used to **fine-tune a dialogue summarization task**.

---

## 4. Additional Fine-Tuning for a Specific Use Case

### Motivation
- FLAN-T5 is a great general-use model with good capability across many tasks, but there may still be **room for improvement on your specific use case**.

### Scenario in the lecture
- You are a **data scientist building an app to support a customer service team** that processes requests received through a **chatbot**.
- The team needs a **summary of every dialogue** to:
  - Identify the **key actions the customer is requesting**
  - Determine **what actions should be taken in response**

### The gap
- SAMSum gives FLAN-T5 some ability to summarize conversations.
- However, SAMSum examples are **mostly conversations between friends about day-to-day activities**.
- They **don't overlap much with the language structure** seen in customer service chats.

### The solution
- Perform **additional fine-tuning** of FLAN-T5 using a dialogue dataset **much closer to the conversations your bot handles**.
- This is the **exact scenario explored in the lab this week**.

---

## 5. Example Dataset: DialogSum (Domain-Specific Summarization)

### Overview
- An **additional domain-specific summarization dataset** used to improve FLAN-T5's ability to summarize **support chat conversations**.
- Consists of **over 13,000 support chat dialogues and summaries**.
- It is **not part of FLAN-T5's training data**, so the model **has not seen these conversations before**.

### Example used in the lecture
- A typical support chat: a conversation between a **customer and a staff member at a hotel check-in desk**.
- A **template is applied** so the **instruction to summarize** the conversation appears at the **start of the text**.

---

## 6. Before vs. After Fine-Tuning on DialogSum

### FLAN-T5 before additional fine-tuning
- **Does okay**: identifies that the conversation was about a **reservation for Tommy**.
- **Weaker than the human-generated baseline summary**, which includes important information such as **Mike asking for information to facilitate check-in**.
- The completion **invented information not in the original conversation**: the **name of the hotel** and the **city it was located in**.

### FLAN-T5 after fine-tuning on DialogSum
- **Closer to the human-produced summary**.
- **No fabricated information**.
- Includes **all of the important details**, including the **names of both people** in the conversation.

---

## 7. Fine-Tuning on Your Own Data

- The example used the **public DialogSum dataset** to demonstrate fine-tuning on custom data.
- In practice, you get the **most out of fine-tuning by using your company's own internal data**, for example, **support chat conversations from your customer support application**.
- This helps the model learn:
  - The **specifics of how your company likes to summarize conversations**
  - **What is most useful to your customer service colleagues**
- This example will be covered in the **lab**, where you can try it yourself.

---

## 8. Looking Ahead

- A key consideration when fine-tuning is **how to evaluate the quality of your model's completions**.
- The **next video** covers **several metrics and benchmarks** to determine:
  - **How well your model is performing**
  - **How much better your fine-tuned version is than the original base model**

---

## Quick-Reference Summary

| Concept | Key Point |
|---|---|
| Multi-task fine-tuning | Train on mixed examples from multiple tasks to improve all tasks simultaneously |
| Main benefit | Avoids catastrophic forgetting |
| Main drawback | Needs lots of data (roughly 50,000 to 100,000 examples) |
| FLAN | Fine-Tuned Language Net: instruction set used as the "dessert" after pre-training |
| FLAN-T5 | Fine-tuned on 473 datasets across 146 task categories |
| SAMSum | 16,000 messenger-like conversations with summaries, written by linguists |
| Varied instruction phrasing | Helps the model generalize and perform better |
| DialogSum | 13,000+ support chat dialogues and summaries; not in FLAN-T5's training data |
| Result of extra fine-tuning | Closer to human summary, no fabricated details, includes names of both participants |
| Best practice | Fine-tune on your company's own internal data |
| Next topic | Metrics and benchmarks for evaluating model performance |
