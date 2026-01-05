# DeepFinanceAgent — Explainable Loan Risk Assistant  
**RAG + Classical ML + Controlled LLM Generation**

DeepFinanceAgent is a **full-stack AI system** for loan risk assessment that deliberately combines:

- **Classical machine learning** for numeric risk prediction  
- **Retrieval-Augmented Generation (RAG)** for factual loan-concept definitions  
- **A constrained Large Language Model (LLM)** for controlled natural-language explanations  

The system is designed to be **deterministic, auditable, and production-oriented**, avoiding the common pitfall of using LLMs as black-box decision engines.

---

## Core Design Principle

> **The LLM never makes credit decisions.**

All **risk prediction and approval logic** is handled by a trained **scikit-learn classification model**.  
The **LLM is used only where it adds value**: explanation, summarization, and language generation under strict constraints.

---

## Role of the LLM (Explicit)

### What the LLM DOES

The project uses **FLAN-T5** as a **controlled language generator** for:

1. **Human-readable explanations**  
   - Converts structured model outputs (probability, bucket, decision) into clear text
   - Mentions only user-provided features (income, credit score, LTV, DTIR1, age)
   - Enforced format:  
     - 1 short paragraph  
     - Exactly 3 bullet points  

2. **RAG-grounded concept explanations**  
   - Generates explanations **only from retrieved context**
   - If a concept is not present in `concepts.json`, the model responds:
     > *“Not found in provided context.”*

3. **Natural language smoothing**  
   - Removes rigid template language
   - Produces professional, consistent explanations suitable for end users or reports

---

### What the LLM DOES NOT Do

- ❌ Does **not** predict default probability  
- ❌ Does **not** determine risk bucket  
- ❌ Does **not** approve or decline loans  
- ❌ Does **not** invent definitions, thresholds, or policies  
- ❌ Does **not** access raw datasets or training data  

This separation ensures:
- **Reproducibility**
- **Regulatory explainability**
- **No hallucinated decisions**

---

## System Architecture

```text
User Input
   │
   ├── Rule-Based Intent Router
   │     ├── Concept Query → RAG → LLM (definition only)
   │     ├── Application Data → ML Model → Decision Engine
   │     └── Out-of-Scope → Safe Response
   │
   ├── Risk Model (scikit-learn)
   │     └── predict_proba → default_probability
   │
   ├── Offer Engine (deterministic rules)
   │     └── risk_bucket → decision
   │
   └── LLM (FLAN-T5)
         └── Explanation Generation (constrained)
Why This LLM Design Is Strong

Most AI demos:

Let the LLM decide

Blend reasoning, prediction, and explanation

Are impossible to audit

This project:

Uses the LLM as an interface layer

Keeps numerical reasoning deterministic

Separates policy, prediction, and language

This mirrors real financial AI systems used in regulated environments.

Features
RAG-Based Concept Knowledge

Definitions stored in concepts.json

FAISS vector index for retrieval

LLM answers only from retrieved content

Knowledge updates require no model retraining

Risk Prediction (ML)

Logistic Regression pipeline

Imputation + scaling + one-hot encoding

Outputs calibrated default probability

Threshold-based risk buckets (Low / Medium / High)

Stateful Chat Agent

Remembers provided fields

Prevents redundant questions

Supports step-by-step or single-message applications

Output Governance

Fixed JSON schema (response_schema.md)

Explanation validation (format + field mentions)

Fallback logic if LLM output violates constraints

Technology Stack

Python

scikit-learn (risk model)

LangChain + FAISS (retrieval)

sentence-transformers (embeddings)

FLAN-T5 (controlled LLM generation)

Gradio (interactive UI)

Why This Project Matters

This repository demonstrates:

Correct use of LLMs in regulated AI systems

Hybrid AI architecture (ML + RAG + LLM)

Production-grade input safety and orchestration

Explainability without hallucination

It is designed to show engineering judgment, not just model usage.
