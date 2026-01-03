# Chatbot Output Schema (Always Return This Structure)

## 1) Risk_Assessment
- risk_bucket: Low | Medium | High
- confidence: 0.0 - 1.0
- short_summary: 2-3 lines

## 2) Offer
- decision: Approve | Approve_with_conditions | Decline | Manual_review
- proposed_terms:
  - loan_amount: (if applicable)
  - interest_rate: (optional, if you decide to include)
  - duration_months: (optional)
- conditions: list of conditions (if any)

## 3) Reasons_Data (CSV-based)
- bullet list of 3-6 data-driven reasons (refer to key features)

## 4) Reasons_Policy (PDF-based, optional for now)
- bullet list of 1-3 policy-driven reasons (later via RAG)

## 5) Evidence
- key_features_used: {feature: value}
- cohort_stats: short stats (optional)
- notes: any assumptions

## 6) Next_Actions
- missing_fields_needed (if any)
- recommended_verifications / documents
