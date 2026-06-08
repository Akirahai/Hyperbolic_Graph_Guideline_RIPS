# Phase 5: Research Gaps & Open Problems

---

## Critical Gap 1: No End-to-End Hyperbolic RAG Pipeline

**What exists:** The ICML 2025 position paper proposes a 5-stage Hyperbolic HKG Pipeline, but this is a position paper — no implementation or empirical evaluation is provided. geoopt handles training. hnswlib can wrap Poincaré distance.

**What's missing:**
- No paper demonstrates a fully working RAG system with hyperbolic embeddings in the retrieval layer
- No comparison between Euclidean vs. Poincaré retrieval on clinical QA tasks
- No evidence that the 10–20% Hits@k gains from Nickel & Kiela (2017) transfer to RAG retrieval quality

**Research opportunity:** Build the first end-to-end clinical RAG system with Poincaré ball embeddings for SNOMED/ICD hierarchy nodes, and evaluate on MIRAGE. This would directly validate the core claim of the ICML position paper.

---

## Critical Gap 2: Guideline Graph → Retrieval Integration

**What exists:** Guideline2Graph produces an excellent graph (87.5% edge recall on NCCN Prostate). CPGPrompt produces an executable decision tree. But neither paper shows how to use the produced structure for RAG.

**What's missing:**
- No paper evaluates whether a guideline graph improves RAG over flat text on medical QA
- No published system takes Guideline2Graph output → PathRAG/GraphRAG retrieval → clinical QA
- The "graph" from Guideline2Graph is a decision graph, not a knowledge graph — it contains nodes like "Grade 2 Hypertension" and edges like "→ initiate Ramipril" but not entity relationships needed for UMLS alignment

**Research opportunity:** Build the bridge: Guideline2Graph output → entity normalization (UMLS) → KG with typed relations → PathRAG retrieval → clinical QA evaluation on MIRAGE.

---

## Critical Gap 3: Multi-Guideline Conflict Resolution

**What exists:** Graph rewriting for concurrent CPGs (AIM 2023) proposes a formal computational method. MED-COPILOT handles multi-guideline retrieval from 643 documents but without explicit conflict handling.

**What's missing:**
- No systematic evaluation of how many clinically important contradictions exist between concurrent guidelines (e.g., Hypertension + CKD + Diabetes guidelines on antihypertensive choice)
- No automated detection of guideline contradictions at query time
- No published clinical NLP dataset annotating guideline contradictions

**Research opportunity:** Create a benchmark of known inter-guideline contradictions (consult clinical pharmacists), then evaluate conflict detection and resolution methods on it.

---

## Critical Gap 4: Real Patient Query Evaluation

**What exists:** MIRAGE (7,663 MCQ from exams), NICE RAG (70 manually curated QA pairs), CPGPrompt (synthetic vignettes). All are synthetic or exam-format.

**What's missing:**
- No dataset of real clinician queries against guidelines in clinical settings
- No evaluation using real patient EHR data + actual guideline queries
- MED-COPILOT uses MIMIC-IV for patient cases but only for plan generation, not for question answering about guidelines

**Research opportunity:** Partner with a clinical partner to create an annotated dataset of de-identified {patient case, guideline query, expected recommendation} triples from real clinical practice. This would be the gold standard for evaluating RIPS.

---

## Critical Gap 5: LLM Update Handling for Dynamic Guidelines

**What exists:** AMG-RAG does dynamic KG updates from PubMed at query time. But clinical guidelines have official update cycles (annually or on evidence availability).

**What's missing:**
- No system tracks guideline versions and updates its KG automatically when WHO/NICE/MOH publish updates
- No evaluation of performance degradation when guidelines are updated but KG is stale
- No framework for detecting which parts of a graph become outdated when a guideline section changes

**Research opportunity:** Design a guideline versioning system that monitors official sources, triggers incremental KG updates, and evaluates retrieval quality before/after guideline updates.

---

## Critical Gap 6: Multilingual and Non-English Guideline Support

**What exists:** MedExpQA (multilingual benchmark, arXiv:2404.05590) covers some languages. Guideline2Graph was tested on English NCCN only.

**What's missing:**
- Most WHO guidelines are available in 6 UN languages; NICE is English-only
- No system handles multilingual clinical guidelines in an integrated graph
- Regional guidelines (e.g., MOH Singapore, NICE UK, S3 German guidelines) have language barriers

**Research opportunity:** Extend Guideline2Graph and RAG pipeline to multilingual CPGs, with cross-lingual entity normalization to SNOMED concepts.

---

## Critical Gap 7: Explainability and Clinician Trust

**What exists:** MED-COPILOT provides token-level saliency visualization. Guideline2Graph provides subtree highlighting. CPGPrompt provides decision audit logs.

**What's missing:**
- No user study comparing explainability approaches for clinician trust
- No evaluation of whether clinicians actually use the provided explanations
- No "proof of guideline citation" — a cryptographic or reference chain from recommendation back to official guideline source

**Research opportunity:** Conduct a clinician user study comparing explanatory interfaces (saliency vs. graph visualization vs. citation-based) to measure trust calibration.

---

## Summary of Gaps by Priority for RIPS Project

| Gap | Priority | Effort | Impact |
|-----|----------|--------|--------|
| Guideline Graph → Retrieval Integration | **CRITICAL** | High | Core pipeline gap |
| Real Patient Query Evaluation | **CRITICAL** | High | Gold standard evaluation |
| Multi-Guideline Conflict Resolution | HIGH | Medium | Safety-critical |
| End-to-End Hyperbolic RAG | HIGH | High | Key differentiator |
| Dynamic Guideline Updates | MEDIUM | Medium | Maintenance |
| Multilingual Guidelines | LOW | High | Future scope |
| Clinician Trust Evaluation | LOW | Medium | Human factors |
