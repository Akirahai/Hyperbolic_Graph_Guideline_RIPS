# Phase 1: Frontier — Medical RAG for Clinical Guideline Agents

**Topic:** Research on Medical RAG in the context of creating medical agents that query clinical guidelines to answer clinical queries for patient cases.

**Reference papers from user:** Guideline2Graph (IJCAI 2025), Hyperbolic Embeddings for Health Knowledge Graphs (ICML 2025 position paper).

---

## Key Trending Directions (2024–2026)

### 1. GraphRAG for Clinical Decision Support
The dominant trend: moving from flat text chunks to structured graph-based retrieval for medical QA. Microsoft's GraphRAG (2024) demonstrated that community-level graph summaries outperform naive RAG on knowledge-intensive tasks. This has been adopted in medical contexts:
- **Agentic Medical Knowledge Graphs (EMNLP 2025)** — automates KG construction + continuous updating + integrated reasoning for medical QA. Achieves F1 74.1% on MEDQA.
- **A self-correcting Agentic Graph RAG for clinical decision support in hepatology** (PMC 2025) — adds self-correction loop to clinical GraphRAG.
- **MedRAG: KG-Elicited Reasoning for Healthcare Copilot** (TheWebConf/WWW 2025) — embeds KG reasoning explicitly in RAG pipeline.
- **PathRAG** (SIGIR/ACM 2025) — pruning graph RAG using relational paths, highly cited (56).

### 2. Clinical Guideline Structured Extraction & Execution
Converting natural language CPGs into executable machine formats:
- **Guideline2Graph** (IJCAI 2025, user reference) — profile-aware multimodal parsing of CPG PDFs into executable decision graphs with VLM. State-of-the-art on prostate cancer NCCN guideline.
- **CPGPrompt** (2026) — translating clinical guidelines into LLM-executable decision support.
- **Graph rewriting for CPG operationalization** (Artificial Intelligence in Medicine) — formal methods for executing concurrent guidelines.
- **Decision Tree Extraction for CDSS** (2025) — if-else pseudocode extraction.

### 3. Medical RAG Benchmarks
Systematic evaluation infrastructure is now mature:
- **MedRAG benchmark (MIRAGE)** (ACL Findings 2024) — 7,663 questions, 5 datasets, 41 retriever/LLM combos. Canonical evaluation framework. MedRAG improves GPT-3.5 by up to 18%.
- **ASTRID — Automated, Scalable TRIaD for RAG-based Clinical QA Evaluation** (ACL Findings 2025) — scalable automated evaluation.
- **Comprehensive RAG Evaluation for Medical QA** (2024) — 12 RAG variants on 250 patient vignettes.

### 4. Hyperbolic & Hierarchical Embeddings for Medical Knowledge
Encoding deep ontological hierarchies efficiently:
- **Hyperbolic Embeddings for Health Knowledge Graphs** (ICML 2025 position, user reference) — argues hyperbolic spaces are essential for HKGs in LLM pipelines. 10-20% gains in Hits@k.
- **Self-Supervised Graph Learning with Hyperbolic Embedding for Health Events** (IEEE JBHI) — Poincaré embeddings for temporal health event prediction.
- **Snomed2Vec** — Poincaré embeddings from SNOMED hierarchy (early pioneer).
- **Multi-Ontology Integration with Dual-Axis Propagation** (2025) — SNOMED/ICD multi-ontology representation.

### 5. Agentic Medical Systems
Full pipelines combining retrieval, reasoning, and tool use:
- **MED-COPILOT** (2025) — GraphRAG + similar patient case retrieval, curates 118 WHO + 525 NICE guidelines (40M+ tokens).
- **Patho-AgenticRAG** (AAAI 2025) — multimodal agentic RAG for pathology.
- **Rationale-Guided RAG for Medical QA** (NAACL 2025) — uses rationales to guide retrieval.
- **Self-MedRAG** (2025) — self-reflective hybrid RAG for reliable medical QA.

### 6. KG Construction from Clinical Text
Building KGs from clinical narratives:
- **EHR-based Medical KG Construction** (J. Biomedical Informatics 2023) — 81 citations, foundational.
- **KG Construction in Sepsis Care** (JMIR 2024) — multicenter LLM-driven KG construction.
- **Clinical KG with Multi-LLMs via RAG** (2026 preprint) — multi-LLM verification.
- **Causal KG for Clinical Decision Support** (J. Biomedical Informatics 2023) — 32 citations.

---

## Key Papers Identified (≥20 citations, recent)

| Title | Venue | Year | Citations |
|-------|-------|------|-----------|
| RAG with KG for Customer Service QA | SIGIR 2024 | 2024 | 231 |
| EHR-based Medical KG construction | J Biomed Inf | 2023 | 81 |
| PathRAG: Pruning Graph-based RAG | SIGIR 2025 | 2025 | 56 |
| KG-Enhanced RAG for FMEA | J. Mfg | 2024 | 57 |
| LLM+RAG for Diabetes Education | JMIR | 2024 | 45 |
| Custom LLM RAG accuracy comparison | Arthroscopy | 2024 | 44 |
| KG enrichment from clinical narratives | Int J Med Info | 2023 | 41 |
| Parametric RAG | ACM/IEEE | 2025 | 40 |
| TrumorGPT: Graph-based RAG | IEEE Trans AI | 2025 | 34 |
| Causal KG for CDS diabetes | J Biomed Inf | 2023 | 32 |
| MedRAG benchmark (MIRAGE) | ACL Findings 2024 | 2024 | ~200* |
| Guideline2Graph | IJCAI 2025 | 2025 | - |
| Hyperbolic HKG | ICML position 2025 | 2025 | - |
| Agentic Medical KG (AMG-RAG) | EMNLP Findings 2025 | 2025 | - |

---

## Most Relevant Subtopics for RIPS Project

The RIPS project context — a medical agent that:
1. Parses clinical practice guidelines into structured knowledge
2. Stores in a vector/graph database
3. Retrieves relevant guideline chunks for patient queries
4. Generates clinical recommendations via LLM

Directly relevant research threads:
1. **Guideline → Graph** (Guideline2Graph, CPGPrompt, graph rewriting)
2. **KG + RAG for Clinical QA** (MedRAG, AMG-RAG, ClinicalRAG, KG-RAG)
3. **Hierarchical Embedding** (Hyperbolic HKGs, Snomed2Vec, ontology embeddings)
4. **Agentic Retrieval + Reasoning** (Patho-AgenticRAG, Self-MedRAG, MED-COPILOT)
5. **Evaluation** (MedRAG/MIRAGE benchmark, ASTRID)

---

## Emerging Research Gaps

1. **No unified pipeline** from raw CPG PDFs → hyperbolic KG → clinical QA. Each component is studied separately.
2. **Evaluation on real patient cases**: Most benchmarks use MCQs, not real longitudinal patient queries against guidelines.
3. **Multi-guideline conflict resolution**: Patient with CKD + Hypertension + Diabetes may be addressed by 3+ separate guidelines — handling contradictions is unsolved.
4. **Structured guideline retrieval** vs. flat text: Guideline2Graph makes graphs but doesn't yet connect to hyperbolic embedding layer.
5. **Low-dimension efficiency**: Hyperbolic embeddings promise 5-32 dims vs. 768+ Euclidean — not yet exploited in RAG indexing.

---

## Top arXiv Preprints to Watch (2025)

- NICE Clinical Guidelines RAG system (arXiv:2510.02967) — 64.7pp faithfulness gain
- RAR² Thought-Driven Retrieval (arXiv:2509.22713) — chain-of-thought retrieval for medical QA
- FHIR-RAG-MEDS (arXiv:2509.07706) — integrates FHIR patient records with RAG
- MedExpQA multilingual benchmark (arXiv:2404.05590)
- Self-MedRAG self-reflective framework (arXiv:2601.04531)
- MED-COPILOT (arXiv:2603.00460) — GraphRAG + patient case retrieval
