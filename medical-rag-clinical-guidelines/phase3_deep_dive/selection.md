# Phase 3: Deep Dive — Paper Selection

## Selection Criteria
Papers selected to cover the full pipeline of the RIPS project:
1. Guideline digitization (raw PDF → structured knowledge)
2. Knowledge graph construction and embedding
3. Hyperbolic geometry for hierarchical medical ontologies
4. RAG pipelines for medical QA
5. Agentic orchestration of retrieval + reasoning
6. Evaluation frameworks and benchmarks

## Selected Papers (15)

### HIGH PRIORITY — Core Pipeline Papers

1. **Guideline2Graph** (IJCAI 2025) — USER REFERENCE
   - *Why*: Directly addresses stage 1 of RIPS pipeline (CPG → graph). Best known method for multimodal guideline parsing.
   - arXiv:2604.02477

2. **Hyperbolic Embeddings for Health Knowledge Graphs** (ICML 2025 position) — USER REFERENCE  
   - *Why*: Directly addresses embedding stage. Proposes full 5-stage pipeline.
   - openreview.net/pdf?id=Sz90WdONPz

3. **MedRAG/MIRAGE Benchmark** (ACL Findings 2024)
   - *Why*: Canonical evaluation framework for medical RAG. Defines what 18% gain means.
   - arXiv:2402.13178

4. **AMG-RAG: Agentic Medical KG** (EMNLP Findings 2025)
   - *Why*: Closest architecture to RIPS project — agentic KG + medical RAG. Best published F1 on MEDQA.
   - arXiv:2502.13010

5. **MED-COPILOT: GraphRAG + Patient Case Retrieval** (preprint 2025)
   - *Why*: Most complete clinical pipeline — curates full WHO + NICE guidelines, uses GraphRAG + case-based retrieval.
   - arXiv:2603.00460

### HIGH PRIORITY — Retrieval & Reasoning

6. **PathRAG: Pruning Graph-based RAG** (SIGIR 2025)
   - *Why*: Best current graph RAG method by relational paths. Directly applicable to clinical KG traversal.
   - arXiv:2404.17723 (note: need to verify this is PathRAG vs. customer service paper)

7. **FHIR-RAG-MEDS** (preprint 2025)
   - *Why*: Only paper integrating patient FHIR records + guideline RAG — critical for the patient-case → guideline query use case.
   - arXiv:2509.07706

8. **NICE Clinical Guidelines RAG** (preprint 2025)
   - *Why*: Highest faithfulness scores on real clinical guidelines (99.5%). Direct implementation case study.
   - arXiv:2510.02967

9. **Rationale-Guided RAG for Medical QA** (NAACL 2025)
   - *Why*: Rationale-based query expansion is the most promising technique for handling conditional guideline logic ("IF patient has CKD AND is diabetic THEN...").

10. **Self-MedRAG** (preprint 2025)
    - *Why*: Self-reflective RAG reduces hallucination significantly in medical context. Critical for trustworthy clinical recommendations.
    - arXiv:2601.04531

### MEDIUM PRIORITY — Supporting Methods

11. **CPGPrompt** (2026, peer-reviewed)
    - *Why*: Most recent work on making guidelines LLM-executable. Complements Guideline2Graph.
    - arXiv: need to find

12. **Self-Correcting Agentic Graph RAG for Hepatology** (PMC 2025)
    - *Why*: Clinical deployment of agentic graph RAG with self-correction — closest to production system.

13. **Causal KG for Diabetes CDS** (J. Biomed. Inf. 2023)
    - *Why*: Foundational paper showing causal KG (not just associative) improves clinical decision quality. 32 citations.

14. **Graph Rewriting for Concurrent CPGs** (AIM 2023)
    - *Why*: Solves the multi-guideline conflict problem (HTN + CKD + Diabetes = 3 guidelines).

15. **Patho-AgenticRAG** (AAAI 2025)
    - *Why*: First multimodal agentic RAG at a top AI conference — demonstrates feasibility at AAAI.
    - arXiv:2508.02258
