

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
 │Fine-tuned DistilBERT →  jailbreak? │
 └────────┬────────────────────────────┘
          ↓
   [Safe? → Yes → Return to User]
          ↓
   [Unsafe? → No → Block or Replace]

```
## Dataset used for Fine-Tuning DistilBert:

3880 Total Samples

1940 safe samples:

[

1- 1925 from Quora Questions Dataset   


2- 15 Tricky Safe Prompts added manually

 ]


1940 harmful samples:
[

1-  520 from Advbench Dataset



2-  1405 from JailbreakHub



3-  15 tricky jailbreak prompts added manually
]



# THIRD DEFENSIVE METHOD

# LLM Self Defense 
LM Self Defense is a zero-shot defense method that uses a large language model (LLM) to analyze its own output and determine whether it is harmful. Rather than relying on external classifiers, fine-tuning, or preprocessing, this approach wraps the generated text in a simple classification prompt and asks another LLM instance to verify if the response contains harmful content.

This method is:

⚡ Fast — No fine-tuning or multi-step generation required

🧠 Self-contained — Uses the LLM’s own capabilities to detect harmful content

🔐 Effective — Shown to reduce attack success rates to nearly 0, even under strong adversarial prompts

🧩 How It Works
A user prompt (possibly adversarial) is submitted.

The LLM generates a response (LLM_gen).

That response is embedded into a classification prompt and passed to a second LLM instance (LLM_filter).

The filter LLM classifies the text as either:

"Yes, this is harmful"

"No, this is not harmful"

If harmful, the response is blocked before reaching the user.


# System Architecture 

User Prompt (T_in)
       ↓
 GPT-3.5 (LLM_gen) → Generates → T_resp
       ↓
 Embed T_resp into harm detection prompt
       ↓
 GPT-3.5 (LLM_filter) → "Yes" / "No"
       ↓
 If "Yes" → Block
 If "No"  → Return to User

# LLM Self Defense vs. Input/Output Filtering
| Method               | Description                                                         | Pros                                                          | Cons                                                           |
| -------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------- |
| **Input Filtering**  | Detect harmful *prompts* before they reach the model.               | Simple to implement. Can block obvious attacks early.         | Can't catch clever jailbreaks or subtle adversarial inputs.    |
| **Output Filtering** | Use a trained classifier or keywords to detect harmful *responses*. | Post-hoc control over LLM output. Flexible.                   | Needs external classifier or tuning. False positives possible. |
| **LLM Self Defense** | Ask the LLM itself to classify its output via a structured prompt.  | No training needed. Zero-shot. Strong defense shown in paper. | Depends on LLM’s reasoning. May require strict prompting.      |


