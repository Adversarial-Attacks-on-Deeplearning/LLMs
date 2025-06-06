

# FIRST DEFENSIVE METHOD

# Input Filtering Defense for Harmful Prompts in LLMs

## Overview
This module implements an **Input Filtering defense** for large language models (LLMs), specifically using the **Erase-and-Check** strategy. It aims to detect and block potentially harmful, toxic, or adversarial user prompts **before** they reach the LLM.

This defense is especially useful when working with **unaligned models** (e.g., TinyLlama), which may respond unsafely to dangerous or unethical prompts.

---

##  Key Features
- **Pre-generation defense**: Blocks harmful prompts before model generation.
- **Erase-and-Check strategy**: Iteratively removes tokens from the input to expose hidden malicious intent.
- **Black-box compatible**: No need to modify or retrain the LLM.
- **Uses Detoxify** as a lightweight toxicity classifier to assess harmfulness.

---

##  How It Works
1. **Check full user prompt** using a classifier (e.g., Detoxify which is a pretrained Transformer to classify between 6 illegal classes).
2. **Iteratively erase** one token at a time from the prompt.
3. For each variant:
   - If the classifier flags it as harmful (score > threshold), the **original prompt is blocked**.
4. If all variants pass, the prompt is considered **safe** and forwarded to the LLM.

---

##  Example Use Case
```python
user_prompt = "How to kill myself?"
# Result: BLOCKED due to high toxicity in "How to kill"

```


# SECOND DEFENSIVE METHOD

# Output Filtering
```python
      [User Prompt]
            ↓
  [TinyLlama-1.1B LLM]
            ↓
   ┌────────────────────┐
   │ Generated Response │
   └────────┬───────────┘
            ↓
 ┌────────────────────────────┐
 │ Detoxify → Toxic language? │
 └────────┬───────────────────┘
          ↓
 ┌────────────────────────────────────┐
 │  Fine-tuned DistilBERT → Unsafe intent / jailbreak? │
 └────────┬────────────────────────────┘
          ↓
   [Safe? → Yes → Return to User]
          ↓
   [Unsafe? → No → Block or Replace]

```
