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
## Why This LLM Design Is Strong

**Most AI demos:**
- Let the LLM **make decisions**
- Blend **reasoning, prediction, and explanation** into a single black box
- Are **difficult or impossible to audit**

**This project:**
- Uses the **LLM strictly as an interface layer**
- Keeps **numerical reasoning deterministic** (handled by classical ML)
- **Separates concerns**: policy, prediction, and language

This architecture closely mirrors **real-world financial AI systems** operating in **regulated environments**, where transparency, traceability, and reproducibility are mandatory.

---

## Features

### RAG-Based Concept Knowledge
- Loan and risk definitions are stored in a curated `concepts.json`
- A **FAISS vector index** enables fast semantic retrieval
- The LLM answers **only from retrieved context**
- Knowledge updates require **no model retraining**

### Risk Prediction (ML)
- **Logistic Regression** pipeline for default risk estimation
- Robust preprocessing:
  - Missing-value imputation
  - Feature scaling
  - One-hot encoding for categorical variables
- Outputs a **calibrated default probability**
- Uses **threshold-based risk buckets**: Low / Medium / High

### Stateful Chat Agent
- Remembers previously provided applicant fields
- Prevents redundant or repeated questions
- Supports both **step-by-step** and **single-message** loan applications

### Output Governance
- Enforced **fixed JSON schema** (`response_schema.md`)
- Explanation validation:
  - Required format
  - Mandatory feature mentions
- Deterministic **fallback logic** if LLM output violates constraints

---

## Technology Stack
- **Python**
- **scikit-learn** — risk prediction model
- **LangChain + FAISS** — retrieval and vector search
- **sentence-transformers** — semantic embeddings
- **FLAN-T5** — controlled LLM-based text generation
- **Gradio** — interactive chat interface

---

## Why This Project Matters

This repository demonstrates:
- Correct and **responsible use of LLMs in regulated AI systems**
- A **hybrid AI architecture** combining ML, RAG, and LLMs
- **Production-grade input safety**, orchestration, and validation
- **Explainability without hallucination**

The project is intentionally designed to showcase **engineering judgment**, not just model usage.

YOU can see sample of sentences in :
https://github.com/Pegah1367/DeepFinanceAgent-Explainable-Financial-Risk-AI-Agent/blob/main/Untitled1.png
https://github.com/Pegah1367/DeepFinanceAgent-Explainable-Financial-Risk-AI-Agent/blob/main/Untitled2.png
https://github.com/Pegah1367/DeepFinanceAgent-Explainable-Financial-Risk-AI-Agent/blob/main/Untitled3.png
https://github.com/Pegah1367/DeepFinanceAgent-Explainable-Financial-Risk-AI-Agent/blob/main/Untitled4.png
