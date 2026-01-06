# DeepFinanceAgent — Explainable Loan Risk Assistant (RAG + ML + Governance)

An end-to-end **explainable financial risk assistant** that combines a **deterministic scikit-learn risk model** with a **Retrieval-Augmented Generation (RAG)** knowledge layer (LangChain + FAISS) to deliver **auditable** loan decisions and **grounded** concept explanations.

This project is designed with a **regulated-AI mindset**: the LLM is used for *language*, while **risk scoring and decisioning remain deterministic** and schema-controlled. :contentReference[oaicite:0]{index=0}

---

## What this does

### 1) Loan Application Risk Assessment (ML)
- Trains (or loads) a **Logistic Regression** pipeline with:
  - Numeric preprocessing: median imputation + scaling
  - Categorical preprocessing: most-frequent imputation + one-hot encoding
- Produces a **default probability** (0–1)
- Maps probability → **risk bucket**:
  - `Low` if `p < 0.30`
  - `Medium` if `0.30 ≤ p < 0.70`
  - `High` if `p ≥ 0.70`
- Translates risk bucket → **offer decision**:
  - Low → `Approve`
  - Medium → `Approve_with_conditions`
  - High → `Decline` :contentReference[oaicite:1]{index=1}

### 2) Loan Concept Q&A (RAG)
- Stores loan concepts and definitions in **`concepts.json`**
- Builds a **FAISS** vector index from approved sources:
  - `concepts.json` (domain definitions)
  - `response_schema.md` (output contract / governance)
  - optional `data_dictionary.csv` (field descriptions)
- Answers questions like:
  - “What does DTIR1 mean?”
  - “Define LTV”
- Uses **direct concept lookup first** (best for auditability), and falls back to retrieval + controlled generation when needed. :contentReference[oaicite:2]{index=2}

### 3) Output Governance (Schema + Guards)
- Enforces a predictable structure (schema-driven output)
- Adds input normalization and safety:
  - Parses user inputs like `"4,380"`, `"45%"`, `"$3500"`
  - Clamps probabilities to valid bounds
  - Converts age ranges like `"25-34"` to numeric (average)
- Prevents common extraction errors (e.g., loan amount accidentally captured as income) :contentReference[oaicite:3]{index=3}

---

## Demo behavior (examples)

### Application
User:
> I want to apply for a loan. My income is 4380 and my credit score is 540.  
> Loan amount is 450000, LTV is 78, DTIR1 is 45, and my age is 35.

Assistant:
- Computes default probability
- Assigns risk bucket (Low/Medium/High)
- Returns a final offer decision (Approve / Approve_with_conditions / Decline)

### Concepts
User:
> What does DTIR1 mean?

Assistant:
- Retrieves definition from `concepts.json`
- Responds with a grounded explanation and the source

---

## System Architecture

**User Message**
→ Intent Router (**rule-first**)
→ (A) **Application Path**
- Field extraction (regex)
- Risk model inference (LogReg pipeline)
- Risk bucket + Offer engine
- Explanation generator (LLM with strict format + fallback)
- Final schema JSON

→ (B) **Concept Path**
- Direct concept lookup in `concepts.json`
- Else: RAG retrieval (FAISS) + controlled LLM generation

Key design principle:
- **Deterministic numeric reasoning**
- **LLM is a “language layer,” not the decision-maker** :contentReference[oaicite:4]{index=4}

---

## Tech Stack

- **Python**
- **scikit-learn**: preprocessing + Logistic Regression pipeline
- **LangChain**: document wrappers, splitting
- **FAISS**: vector similarity search
- **sentence-transformers**: embeddings (`all-MiniLM-L6-v2`)
- **Transformers**: `google/flan-t5-base` for controlled text generation :contentReference[oaicite:5]{index=5}

---

## Repository Contents (suggested)

> Adapt names to match your repo.

├── notebook/
│ └── Final_project_with_lang_chain.ipynb
├── data/
│ ├── Loan_Default_Cleaned.csv
│ └── data_dictionary.csv
├── config/
│ ├── concepts.json
│ └── response_schema.md
├── artifacts/
│ ├── risk_model.joblib
│ └── agent_artifacts.json
├── faiss_index/
│ └── (FAISS files)
└── README.md


---

## Installation

### Option A — Google Colab (recommended)
1. Upload dataset + config files:
   - `Loan_Default_Cleaned.csv`
   - `concepts.json`
   - `response_schema.md`
   - (optional) `data_dictionary.csv`
2. Run the notebook end-to-end.

### Option B — Local environment
Create a venv, then install:

```bash
pip install -U \
  pandas numpy scikit-learn joblib \
  langchain langchain-community langchain-text-splitters \
  faiss-cpu sentence-transformers \
  transformers torch

## How to Run

### 1) Train (or Load) the Risk Model
The project will **train the Logistic Regression risk model** if a saved artifact is not found.  
If the artifact exists, it will **load the model and continue** without retraining.

**Logic**
- If `risk_model.joblib` **does not exist** → train the model and save artifacts
- If `risk_model.joblib` **exists** → load the model and continue

**Outputs**
- `risk_model.joblib` — trained scikit-learn pipeline (preprocessing + Logistic Regression)
- `agent_artifacts.json` — metadata used by the agent (feature columns, required fields, thresholds, etc.)

---

### 2) Build (or Load) the RAG Index
The project builds a **FAISS vector index** using trusted knowledge sources.  
If the index already exists, it is **reused from disk**.

**Trusted sources used for indexing**
- `concepts.json` — loan concept definitions
- `response_schema.md` — output contract / schema rules
- `data_dictionary.csv` (optional) — field definitions and descriptions

**Caching**
- The index is stored in: `faiss_index/`  
- If `faiss_index/` exists → load it
- If not → build it and save it

---

### 3) Chat Interaction
You can interact with the assistant in two ways:

**A) Concept questions (RAG)**
Ask definitions and explanations:
- “Define LTV”
- “What does DTIR1 mean?”
- “Explain credit score in lending”

**B) Loan application inputs (ML decisioning)**
Provide application fields:
- `income`, `credit_score`, `loan_amount`, `ltv`, `dtir1`, `age`

Example:
> I want to apply for a loan. My income is 4380 and my credit score is 540.  
> Loan amount is 450000, LTV is 78, DTIR1 is 45, and my age is 35.

The system returns:
- default probability
- risk bucket (Low / Medium / High)
- recommended decision (Approve / Approve_with_conditions / Decline)

---

## Key Features (Portfolio Highlights)

### Deterministic Decisioning
- Risk scoring is performed by a **scikit-learn pipeline**, not by the LLM
- Threshold mapping and offer rules are **transparent and auditable**

### Rule-First Intent Routing
- Prevents irrelevant questions (e.g., “what is LangChain?”) from being routed into loan assessment
- Reduces hallucination risk by limiting the LLM to the correct workflow

### Robust Field Extraction & Input Normalization
- Regex-based extraction for required fields
- Supports formatting like:
  - income with commas/currency (`"4,380"`, `"$4,380"`)
  - percent values (`"45%"`)
  - age ranges (`"25-34"` → converts to a numeric value)
- Guards against common mix-ups (e.g., **loan amount** captured as **income**)

### RAG Knowledge That Updates Without Retraining
- Update definitions in `concepts.json`
- Rebuild the FAISS index
- No ML retraining is required for knowledge updates

### Controlled Explanations (Format + Safety)
- Explanations follow strict output formatting rules
- Fallback logic ensures a valid response even if the LLM output fails validation

---

## Model Evaluation
- Uses **ROC-AUC** as a threshold-independent evaluation metric
- Uses stratified train/test splitting to preserve class distribution

---

## Notes on Responsible / Regulated AI
This project demonstrates patterns aligned with regulated financial AI systems:
- Separation of **prediction**, **policy/decision**, and **language**
- Retrieval from approved sources (traceability)
- Defensive input handling and schema control
- Deterministic scoring with transparent thresholds

---

## Future Improvements
- Probability calibration (Platt scaling / isotonic regression)
- Fairness and bias analysis (if sensitive attributes are available and permitted)
- Global + local interpretability (coefficients, SHAP)
- Production API/UI (FastAPI or Streamlit) with monitoring and logging
- Stronger unit tests for extraction, schema compliance, and routing

<img width="697" height="140" alt="Untitled1" src="https://github.com/user-attachments/assets/a40246d3-1a86-4346-a1e8-c2823e3803d3" />
