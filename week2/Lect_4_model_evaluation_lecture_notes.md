# Model Evaluation — Lecture Notes

**Week 2:** Fine-tuning and evaluating large language models


> Notes are based strictly on the lecture transcript.

---

## 1. Lecture Overview

- Statements such as "the model demonstrated good performance on this task" or "the fine-tuned model showed a large improvement over the base model" need to be **formalized** with metrics.
- The lecture introduces metrics used by LLM developers to:
  - Assess the performance of your own models.
  - Compare against other models "out in the world."
- Specific goal: formalize the improvement of a **fine-tuned model over the pre-trained model** it started from.

---

## 2. Why Evaluating LLMs Is Hard

### Traditional machine learning
- Performance is assessed on **training and validation datasets**, where the output is already known.
- Simple metrics such as **accuracy** can be computed (fraction of all predictions that are correct).
- This works because the models are **deterministic**.

### Large language models
- Output is **non-deterministic** and **language-based**, so evaluation is much more challenging.

### Illustrative examples
| Sentence A | Sentence B | Relationship |
|---|---|---|
| Mike really loves drinking tea. | Mike adores sipping tea. | Quite similar in meaning |
| Mike does not drink coffee. | Mike does drink coffee. | Only one word different, but meaning is completely different |

- Humans can easily see these similarities and differences.
- When training a model on **millions of sentences**, you need an **automated, structured way** to make measurements.

---

## 3. The Two Metrics Introduced

| Metric | Full name | Primarily used for | Compares generated output against |
|---|---|---|---|
| **ROUGE** | Recall-Oriented Understudy for Gisting Evaluation | Assessing quality of automatically generated **summaries** | Human-generated reference summaries |
| **BLEU** | Bilingual Evaluation Understudy | Evaluating quality of **machine-translated text** | Human-generated translations |

- "BLEU" is French for "blue," so people sometimes say "blue," but the lecturer uses the original pronunciation "BLEU" as written.

---

## 4. Terminology

- **Unigram:** a single word.
- **Bigram:** two words.
- **n-gram:** a group of *n* words.

---

## 5. ROUGE in Detail

### Running example
- **Reference (human):** "It is cold outside."
- **Generated output:** "It is very cold outside."

### 5.1 ROUGE-1 (unigram-based)

Uses recall, precision, and F1 (similar to other ML tasks):

- **Recall** = (number of unigrams matched between reference and generated output) ÷ (number of unigrams in the **reference**)
  - In the example: perfect score of **1**, since all reference words appear in the generated output.
- **Precision** = (unigram matches) ÷ (number of unigrams in the **generated output**)
- **F1** = **harmonic mean** of recall and precision.

**Limitations:**
- Only focuses on individual words (hence the "1" in the name).
- Does **not consider word order**.
- Can be **deceptive**: it is easy to generate sentences that score well but are subjectively poor.
- Example: if the generated sentence were "It is **not** cold outside" (differing by just one word), the **scores would be the same**.

### 5.2 ROUGE-2 (bigram-based)

- Uses **bigrams** (pairs of words) from the reference and generated sentence.
- Acknowledges, in a very simple way, the **ordering of words**.
- Recall, precision, and F1 are calculated using **bigram matches** instead of single-word matches.
- Scores are **lower than ROUGE-1**.
- With **longer sentences**, there is a greater chance bigrams don't match, so scores may be even lower.
- Rather than continuing to ROUGE-3, ROUGE-4, etc., the lecture moves to a different approach (ROUGE-L).

### 5.3 ROUGE-L (Longest Common Subsequence)

- Looks for the **longest common subsequence (LCS)** present in both the generated output and the reference.
- In the example, the longest matching subsequences are **"It is"** and **"cold outside,"** each with **length 2**.
- The LCS value is used to compute recall, precision, and F1:
  - The **numerator** in both recall and precision is the **length of the LCS** (here, 2).
- Collectively these three quantities are the **ROUGE-L score**.

### 5.4 Important Caveats for All ROUGE Scores

- Scores must be **taken in context**.
- Scores can be used to compare model capabilities **only if determined for the same task** (e.g., summarization).
- ROUGE scores for **different tasks are not comparable** to one another.

### 5.5 Problem: Bad Completions Can Get Good Scores

- Example generated output: **"cold, cold, cold, cold"**
  - Contains one word from the reference, repeated multiple times.
  - Still scores quite highly; the **ROUGE-1 precision will be perfect**.

**Fix: clipping function**
- Limits the number of unigram matches to the **maximum count of that unigram within the reference**.
- Here, "cold" appears **once** in the reference, so the **modified precision** with clipping gives a **dramatically reduced score**.

**Remaining weakness:**
- If the generated words are all present but in a **different order**, the score is still perfect.
- Example: **"outside cold it is"** scores perfectly, even with modified (clipped) precision, because all words in the output are present in the reference.

### 5.6 Choosing the Right ROUGE Variant
- Using a different ROUGE score can help.
- Experimenting with an n-gram size that gives the most useful score depends on:
  - The **sentence**
  - The **sentence size**
  - Your **use case**

### 5.7 Practical Tooling
- Many language model libraries, e.g. **Hugging Face** (used in Week 1's lab), include implementations of ROUGE.
- In this week's lab, you will use ROUGE to **compare model performance before and after fine-tuning**.

---

## 6. BLEU in Detail

- Useful for evaluating the quality of **machine-translated text**.
- Calculated as the **average precision over multiple n-gram sizes**.
  - Like ROUGE-1 precision, but calculated for a **range of n-gram sizes** and then **averaged**.
- Quantifies translation quality by checking **how many n-grams in the machine-generated translation match those in the reference translation**.
- Calculating by hand would require multiple calculations followed by averaging the results.
- In practice, use **pre-written libraries** (e.g., from providers like Hugging Face).

### Worked example (library results)
- **Reference (human):** "I am very happy to say that I am drinking a warm cup of tea."
- **First candidate:** "I am very happy that I am drinking a cup of tea." → **BLEU score = 0.495**
- As candidates get **closer to the original sentence**, the score gets **closer to 1**.

---

## 7. Key Takeaways on Using ROUGE and BLEU

- Both are **quite simple metrics** and **relatively low cost to calculate**.
- Use them for **simple reference as you iterate** over your models.
- **Do not use them alone** to report the final evaluation of an LLM.
- **Use ROUGE** for **diagnostic evaluation of summarization** tasks.
- **Use BLEU** for **translation** tasks.
- For **overall evaluation** of model performance, you need evaluation **benchmarks** developed by researchers (covered in the next video).

---

## 8. Quick Reference Summary

| Concept | Key Point |
|---|---|
| Traditional ML evaluation | Deterministic; simple metrics like accuracy |
| LLM evaluation | Non-deterministic, language-based; harder |
| ROUGE-1 | Unigram recall, precision, F1; ignores order |
| ROUGE-2 | Bigram-based; captures some ordering; lower scores |
| ROUGE-L | Based on longest common subsequence length |
| Clipping | Caps unigram matches at reference count; reduces inflated precision |
| ROUGE limitation | Same-words-different-order can still score perfectly |
| ROUGE comparability | Only comparable within the same task |
| BLEU | Average precision across multiple n-gram sizes; for translation |
| Both metrics | Simple, cheap, good for iteration, not sufficient alone for final evaluation |
| Next step | Evaluation benchmarks from researchers |

---

## 9. Self-Check Questions (derived from lecture content)

1. Why is evaluating LLMs harder than evaluating traditional ML models?
2. What does ROUGE primarily assess, and what does BLEU primarily assess?
3. Define unigram, bigram, and n-gram.
4. How are recall, precision, and F1 calculated for ROUGE-1?
5. Why can ROUGE-1 be deceptive? Give the "not cold" example.
6. How does ROUGE-2 differ from ROUGE-1, and why are its scores lower?
7. What is the longest common subsequence, and how is it used in ROUGE-L?
8. Why can't ROUGE scores from different tasks be compared?
9. How does the clipping function fix the "cold, cold, cold, cold" problem, and what problem remains?
10. How is the BLEU score computed?
11. Why shouldn't ROUGE and BLEU be used alone for the final evaluation of an LLM?
