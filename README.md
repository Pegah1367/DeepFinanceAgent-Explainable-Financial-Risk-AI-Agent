# DeepFinanceAgent  
### Explainable Loan Risk Assistant (ML + RAG)

**DeepFinanceAgent** is an end-to-end **explainable financial AI system** that evaluates loan applications using a **deterministic machine-learning risk model** and answers loan-related questions using **Retrieval-Augmented Generation (RAG)**.  
The design follows **regulated-AI principles**: numerical decisions are made by ML and rules, while the LLM is used only for language and explanations.

---

## What This Project Does

### Loan Risk Assessment (ML)
- Uses a **scikit-learn Logistic Regression pipeline**
- Predicts **probability of default**
- Maps probability to **risk buckets**:
  - Low, Medium, High
- Generates a **clear decision**:
  - Approve / Approve with Conditions / Decline

### Loan Concept Assistant (RAG)
- Answers questions like:
  - “What does LTV mean?”
  - “Define DTIR1”
- Retrieves answers **only from approved sources**
- No hallucinated definitions

---

## Key Features

- **Deterministic Decisioning**  
  Risk scoring is fully ML-based (not LLM-based)

- **Rule-First Intent Routing**  
  Separates *loan applications* from *concept questions*

- **Explainability by Design**  
  Transparent thresholds, structured outputs, auditable logic

- **Robust Input Handling**  
  - Supports currency and percentage formats  
  - Prevents field mix-ups (e.g., income vs loan amount)

- **Updatable Knowledge Without Retraining**  
  Update definitions in `concepts.json` and rebuild the index

---

## Tech Stack

- Python  
- scikit-learn (risk model)  
- LangChain + FAISS (RAG)  
- sentence-transformers (embeddings)  
- Hugging Face Transformers (controlled generation)

---

## How It Works (High Level)

1. **User Input**
   - Loan application **or** concept question

2. **Intent Router**
   - Application → ML pipeline  
   - Concept → RAG pipeline

3. **Risk Model**
   - Predicts default probability
   - Assigns risk bucket
   - Produces decision

4. **Governed Output**
   - Schema-controlled explanation
   - Safe fallback if LLM output fails

---

## Example Usage

**Application**
> Income: 4380, Credit Score: 540, Loan: 450000, LTV: 78, DTIR1: 45, Age: 35  

**Output**
- Default probability
- Risk bucket: High
- Decision: Decline

**Concept**
> What does DTIR1 mean?

**Output**
- Definition retrieved from trusted knowledge

---

## Why This Project Matters

Most AI demos allow the LLM to make all decisions, mixing prediction, reasoning, and explanation in a way that is difficult to audit.

This project separates prediction, policy, and language, mirroring how real financial AI systems are built in regulated environments.

- Machine learning models handle numerical risk prediction  
- Business rules handle decision thresholds and policies  
- The LLM is used only to translate outcomes into clear, human-readable explanations  

This design ensures that outputs are:

- Auditable and deterministic at the decision level  
- Interpretable and transparent for stakeholders  
- Human-centered in language, making complex financial risk understandable to non-technical users  
- **Aligned with real-world communication needs between AI systems, analysts, and customers**




---

## Future Improvements

- Probability calibration
- Bias and fairness analysis
- SHAP-based interpretability
- FastAPI / Streamlit deployment
- End-to-end monitoring and logging

---


