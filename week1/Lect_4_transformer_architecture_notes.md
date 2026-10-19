# Transformer Architecture: Lecture Notes

**Week 1:** Generative AI use cases, project lifecycle, and model pre-training


---

## 1. Why Transformers Matter

- Building LLMs with the transformer architecture dramatically improved performance on natural language tasks compared with the earlier generation of **RNNs**.
- This led to an **explosion in generative capability**.
- The power of the architecture lies in its ability to learn the **relevance and context of all words in a sentence**:
  - Not just each word next to its neighbor, but **every word relative to every other word**.
  - **Attention weights** are applied to these relationships, so the model learns how relevant each word is to each other word, **no matter where they are in the input**.
- This lets the model learn, for example, *who has the book*, *who could have the book*, and whether it is even relevant to the wider context of the document.
- Attention weights are **learned during LLM training** (covered later in the week).

---

## 2. Self-Attention

- **Attention map:** a diagram that illustrates the attention weights between each word and every other word.
- In the stylized example, the word **"book"** is strongly connected to (paying attention to) the words **"teacher"** and **"student"**.
- This is called **self-attention**.
- Learning attention across the whole input **significantly improves the model's ability to encode language**.

---

## 3. High-Level Structure

- The transformer architecture is split into two distinct parts: the **encoder** and the **decoder**.
- The two components work in conjunction and share a number of similarities.
- The lecture's diagram is a simplified version derived from the original paper **"Attention Is All You Need"**.
- In the diagram, **inputs are at the bottom and outputs at the top**; the course tries to stay faithful to this convention.

---

## 4. Step-by-Step Data Flow

### Step 1: Tokenization
- ML models are "just big statistical calculators" that work with **numbers, not words**.
- Text must first be **tokenized**: words are converted into numbers, each number representing a position in a **dictionary of all possible words** the model can work with.
- Multiple tokenization methods exist, for example:
  - Token IDs matching **complete words**.
  - Token IDs representing **parts of words**.
- **Key rule:** once you select a tokenizer to train the model, you **must use the same tokenizer when generating text**.

### Step 2: Embedding Layer
- Input (now numbers) is passed to the **embedding layer**.
- This is a **trainable vector embedding space**: a high-dimensional space where each token is represented as a **vector** occupying a **unique location**.
- Each token ID in the vocabulary is matched to a multi-dimensional vector.
- Intuition: these vectors **learn to encode the meaning and context of individual tokens** in the input sequence.
- Embedding vector spaces have been used in NLP for some time; previous-generation algorithms such as **Word2Vec** use this concept.
- In the original transformer paper, the **vector size was 512**.
- Illustration with a vector size of 3:
  - Words can be plotted in a **3D space**, showing relationships between them.
  - Words located close together in the embedding space can be related.
  - The **distance between words can be calculated as an angle**, giving the model the ability to **mathematically understand language**.

### Step 3: Positional Encoding
- Positional encoding is added as token vectors enter the base of the encoder or decoder.
- The model processes all input tokens **in parallel**, so positional encoding **preserves word order information** and the relevance of a word's position in the sentence.

### Step 4: Self-Attention Layer
- The summed token vectors and positional encodings are passed to the **self-attention layer**.
- The model analyzes the **relationships between tokens** in the input sequence, attending to different parts of the input to better capture **contextual dependencies** between words.
- Self-attention weights are **learned during training** and stored in these layers; they reflect the importance of each word to all other words in the sequence.

### Step 5: Multi-Headed Self-Attention
- Self-attention does not happen just once. The transformer uses **multi-headed self-attention**:
  - **Multiple sets of self-attention weights (heads)** are learned **in parallel, independently** of each other.
  - The number of heads varies by model; **12 to 100** is a common range.
- Intuition: **each head learns a different aspect of language**. Examples:
  - One head may capture relationships between the **people entities** in a sentence.
  - Another may focus on the **activity** of the sentence.
  - Another may focus on other properties, such as whether words **rhyme**.
- You **do not dictate ahead of time** what each head learns:
  - Head weights are **randomly initialized**.
  - Given **sufficient training data and time**, each head learns different aspects of language.
- Some attention maps are easy to interpret (like the examples in the lecture); **others may not be**.

### Step 6: Feed-Forward Network
- After all attention weights are applied, the output is processed through a **fully connected feed-forward network**.
- The output is a **vector of logits** proportional to the probability score for each and every token in the tokenizer dictionary.

### Step 7: Softmax Layer
- Logits are passed to a final **softmax layer**, where they are **normalized into a probability score for each word**.
- The output includes a probability for **every word in the vocabulary**, so there are likely **thousands of scores**.
- **One token will have a score higher than the rest**: the most likely predicted token.
- Later in the course, there are **several methods to vary the final selection** from this probability vector.

---

## 5. Summary Flow

```
Text -> Tokenizer (words to token IDs)
     -> Embedding layer (token IDs to vectors)
     -> + Positional encoding
     -> Multi-headed self-attention
     -> Fully connected feed-forward network (logits)
     -> Softmax (probability for every vocabulary token)
     -> Most likely token (selection method can be varied)
```

---

## 6. Key Terms

| Term | Meaning (per lecture) |
|---|---|
| **Self-attention** | Learning the relevance of each word to every other word in the input |
| **Attention weights** | Learned values reflecting the importance of word relationships |
| **Attention map** | Diagram illustrating attention weights between all word pairs |
| **Encoder / Decoder** | The two distinct parts of the transformer architecture |
| **Tokenization** | Converting words into numbers (positions in a dictionary) |
| **Embedding layer** | Trainable high-dimensional vector space representing tokens |
| **Positional encoding** | Added to token vectors to preserve word order |
| **Multi-headed self-attention** | Multiple independent sets of attention weights learned in parallel |
| **Logits** | Vector of scores proportional to each token's probability |
| **Softmax** | Layer that normalizes logits into probability scores |

---

## 7. Study Takeaways

1. Transformers outperformed RNNs by relating **every word to every other word** via attention.
2. Attention weights are **learned in training**, not hand-specified.
3. Models work with **numbers**, so tokenization is required, and the **same tokenizer must be used for training and generation**.
4. Embeddings place tokens in a **vector space** that encodes meaning and context.
5. Tokens are processed **in parallel**, so **positional encoding** is needed to retain word order.
6. **Multiple attention heads** each learn different language aspects, with **random initialization** and no predefined roles.
7. The final output is a **probability distribution over the whole vocabulary**, from which the next token is chosen.
