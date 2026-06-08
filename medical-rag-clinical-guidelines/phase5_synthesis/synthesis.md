# Phase 5: Synthesis — Medical RAG for Clinical Guideline Agents

---

## 1. Taxonomy of Approaches

The literature can be organized into a 4-layer taxonomy matching the pipeline stages of a clinical guideline RAG agent:

```
Layer 1: Guideline Ingestion & Structuring
  ├── Text-only extraction: CPGPrompt, Text2MDT, MedDM
  ├── Multimodal graph extraction: Guideline2Graph ← SOTA
  └── KG construction from clinical text: EHR-KG, Causal-KG, AMG-RAG

Layer 2: Knowledge Representation
  ├── Euclidean KG embeddings: TransE, RotatE, traditional KGE methods
  ├── Hyperbolic embeddings: Poincaré ball, Lorentz model ← FOR HIERARCHIES
  └── Flat text chunks: naive RAG, BM25-only systems

Layer 3: Retrieval
  ├── Flat retrieval: MedRAG (BM25, MedCPT)
  ├── Graph traversal: GraphRAG, PathRAG, AMG-RAG
  ├── Hybrid (flat+graph): MED-COPILOT, Medical-Graph-RAG
  └── Adaptive/agentic: Patho-AgenticRAG, LogicRAG

Layer 4: Reasoning & Answer Generation
  ├── Single-pass: Standard RAG
  ├── Chain-of-thought: AMG-RAG, MED-COPILOT
  ├── Self-reflective: Self-MedRAG
  └── Multi-agent: SEMA-RAG, multi-agent consensus
```

---

## 2. Comparative Performance Table

### Medical QA Benchmarks

| System | MEDQA | MEDMCQA | Key Method | Venue |
|--------|-------|---------|------------|-------|
| Meditron-70B | 68.3% | 66.0% | Domain fine-tuning | ICML 2024 |
| MedRAG (GPT-3.5) | 71.57% | — | RAG (RRF-4) | ACL 2024 |
| AMG-RAG (~8B) | **73.9%** | **66.34%** | Agentic KG + CoT | EMNLP 2025 |
| MED-COPILOT (Gemini 2.5) | **93.7%** | — | GraphRAG + Cases | preprint 2025 |
| RAG² | SOTA+5.6% | — | Rationale-guided | NAACL 2025 |

**Key observation:** AMG-RAG at 8B parameters outperforms Meditron-70B without fine-tuning. MED-COPILOT's 93.7% on MedQA with Gemini 2.5 + full pipeline is the best published score.

### Guideline Retrieval Benchmarks

| System | Domain | Faithfulness | Key Method |
|--------|--------|-------------|------------|
| NICE RAG (O4-Mini) | 300 NICE guidelines | **99.5%** | Hybrid BM25+dense + reranking |
| NICE RAG (GPT-4.1) | 300 NICE guidelines | 98.7% accuracy | Same; fewer unsafe responses |
| Guideline2Graph | NCCN Prostate | Edge recall 87.5% | 3-stage VLM pipeline |
| CPGPrompt | 3 domains | F1 0.90–1.00 (binary) | Decision tree LLM execution |

### Graph Retrieval Benchmarks

| System | vs GraphRAG Win Rate | vs LightRAG Win Rate | Token Reduction |
|--------|---------------------|---------------------|-----------------|
| PathRAG | 60.44% | 58.46% | 16–44% |
| LogicRAG | +14.7pp (2WikiMQA) | — | Eliminates preprocessing |
| AMG-RAG | — | — | Dynamic KG (no offline cost) |

---

## 3. The Clinical Guideline → Patient Answer Pipeline

Based on synthesis of all 15 papers, the optimal pipeline for a system like RIPS is:

```
[Clinical Guidelines PDFs]
        ↓
STAGE 1: Guideline Digitization
  - Guideline2Graph (IJCAI 2025): multimodal 3-stage VLM pipeline
  - Output: Directed graph G = (V, E) with typed nodes and transition edges
  - CPGPrompt: parallel decision-tree representation for pathway execution

        ↓
STAGE 2: Knowledge Graph Enrichment
  - Normalize nodes against UMLS/SNOMED-CT/ICD-10 (entity normalization)
  - Add causal edges (Liang et al. 2023): HTN → complicates → CKD
  - Integrate drug ontologies: Ramipril → treats → HTN+CKD
  - Store in Neo4j with typed edges

        ↓
STAGE 3: Hyperbolic Embedding Training (Optional but recommended)
  - geoopt Poincaré ball embedding of guideline graph nodes
  - d=16–32 dims (vs 768 Euclidean) for hierarchy-preserving compression
  - Learn curvature c jointly; higher |c| for deep SNOMED hierarchies
  - Riemannian Adam optimizer; project to tangent space for ANN indexing

        ↓
STAGE 4: Retrieval Index
  - Primary: Neo4j graph index for structured traversal
  - Secondary: LanceDB vector index (BioClinicalBERT or MedCPT embeddings)
  - PathRAG path pruning for graph traversal
  - Hybrid BM25+dense for flat chunk fallback

        ↓
STAGE 5: Patient Case Processing
  - Parse FHIR resources: Condition, Medication, Observation, Demographics
  - Extract active diagnoses + lab values + medications
  - Identify guideline coverage: which guidelines apply to this patient?

        ↓
STAGE 6: Multi-Condition Query
  - Rationale-Guided query expansion (RAG²): generate clinical rationale first
  - Self-reflective loop (Self-MedRAG): verify answer groundedness
  - Handle concurrent guidelines: graph rewriting for conflict resolution
  - Retrieve top-k paths from KG + relevant chunks

        ↓
STAGE 7: Answer Generation
  - LLM (Claude or GPT-4) conditioned on retrieved KG paths + guideline text
  - Chain-of-thought reasoning (AMG-RAG style)
  - Explainability: show Poincaré disk subtrees, cited guideline sections
  - Safety check: verify recommendation against KG (TrumorGPT-style)
```

---

## 4. Critical Findings from the Literature

### Finding 1: Retrieval quality is the dominant factor
From MedRAG/MIRAGE (ACL 2024): switching from BM25 to RRF-4 fusion improves MedQA by 10+ points with the same LLM. The corpus matters more than the LLM: GPT-3.5 + MedRAG > GPT-4 without RAG.

**Implication for RIPS:** Invest heavily in retrieval quality. A simple GPT-3.5 with excellent guideline indexing will outperform GPT-4 without proper retrieval.

### Finding 2: Graph structure captures what flat retrieval misses
From MED-COPILOT ablation: GraphRAG adds +0.031 BLEU over no-RAG; patient case retrieval adds +0.028 BLEU; combined = +0.035 BLEU. The comorbidity edges (HTN ↔ CKD ↔ DM) in a graph are not recoverable from flat chunk retrieval.

**Implication for RIPS:** For polypharmacy / multi-comorbidity patients, graph traversal is necessary. A patient with HTN+CKD+T2DM will not get a coherent answer from flat RAG because the three relevant guideline sections are in different documents with no explicit cross-links.

### Finding 3: Faithfulness vs. accuracy trade-off
From NICE RAG: O4-Mini + RAG achieves 99.5% faithfulness but fewer safe responses than GPT-4.1 (98.7% accuracy, 67% unsafe reduction). The "safest" answers are not always the most faithful to guidelines.

**Implication for RIPS:** Design a two-model pipeline: O4-Mini for guideline grounding + safety classifier. Or use GPT-4.1 with strict source-grounding prompts.

### Finding 4: Agentic systems outperform static RAG for complex reasoning
From AMG-RAG: without CoT = 66.69%, without PubMed search = 67.16%, full system = 73.92%. Each component contributes ~7 percentage points. The combination is more than additive.

**Implication for RIPS:** Static guideline graphs need dynamic clinical reasoning. At query time, supplement pre-built KG with relevant literature search (e.g., PubMed for recent drug interactions).

### Finding 5: Hyperbolic embeddings are efficient but hard to deploy
From Hyperbolic HKG (ICML 2025 position): 5-32 dims hyperbolic vs 128-768 Euclidean for same hierarchy quality. But: hnswlib doesn't support Poincaré distance natively; Riemannian optimization requires geoopt; curvature tuning is non-trivial.

**Implication for RIPS:** For v1, use standard BioClinicalBERT embeddings. Hyperbolic embeddings are a significant R&D investment — worth implementing for v2 to handle deep SNOMED hierarchies efficiently.

### Finding 6: Guideline execution requires decision trees, not just graphs
From CPGPrompt: decision tree traversal achieves F1=0.90–1.00 on binary referral decisions. For a concrete clinical query ("Should this patient be referred?"), a decision tree is more precise than a KG path. But for open-ended queries, graph retrieval is more flexible.

**Implication for RIPS:** Use BOTH representations: (1) Decision tree (CPGPrompt-style) for algorithmic pathway execution, (2) KG (Guideline2Graph-style) for flexible NL query answering.

### Finding 7: Self-reflection dramatically improves reliability on complex queries
From Self-MedRAG: +10.72pp on PubMedQA (research questions) — the most complex query type. Self-reflection adds the most value when the initial retrieval is imperfect.

**Implication for RIPS:** Implement a self-reflection module. When retrieved guideline passages are weak (low similarity score), trigger query reformulation + re-retrieval.

---

## 5. Timeline of Key Developments

```
2017 — Nickel & Kiela: Poincaré embeddings for hierarchical data (foundational)
2018 — Sala et al.: Hyperbolic disk embeddings
2019 — Snomed2Vec: First Poincaré embeddings for SNOMED CT
2021 — Hyperbolic graph learning for temporal health events
2023 — Causal KG for diabetes CDSS; EHR-based KG construction (81 cites)
       → Semantically enabling CDSS recommendations
2024 — MedRAG/MIRAGE benchmark (ACL 2024) ← canonical benchmark
       → Microsoft GraphRAG (community-level summarization)
       → PathRAG: flow-based path pruning
2025 — AMG-RAG (EMNLP 2025): agentic KG + CoT for medical QA
       → Guideline2Graph (IJCAI 2025): CPG PDF → executable graph
       → Hyperbolic HKG position paper (ICML 2025)
       → NICE RAG: 99.5% faithfulness on 300 guidelines
       → MED-COPILOT: 93.7% MedQA with GraphRAG + patient cases
       → Patho-AgenticRAG (AAAI 2025): multimodal agentic RAG
       → Self-MedRAG: self-reflective hybrid RAG
2026 — CPGPrompt (JAMIA): decision tree execution for CPGs
       → Multi-agent consensus for medical reasoning (SEMA-RAG, etc.)
```

---

## 6. Architecture Comparison of Most Relevant Systems

| System | KG Source | Embedding | Retrieval | Reasoning | Patient Context |
|--------|-----------|-----------|-----------|-----------|-----------------|
| MED-COPILOT | WHO+NICE guidelines | BioClinicalBERT | GraphRAG community | LLM + CoT | MIMIC-IV cases |
| AMG-RAG | PubMed (dynamic) | LLM descriptions | BFS/DFS confidence | CoT chains | None |
| Medical-Graph-RAG | UMLS+MedC-K+MIMIC | — | U-Retrieval | Evidence-based | MIMIC-IV |
| FHIR-RAG-MEDS | Clinical guidelines | — | RAG | LLM | FHIR records |
| NICE RAG | 300 NICE guidelines | Voyage-3-large | Hybrid BM25+dense+rerank | GPT-4.1 | None |
| **RIPS (proposed)** | CPG guidelines | Hyperbolic (Poincaré) | PathRAG + hybrid | CoT + Self-reflect | FHIR |

---

## 7. Key Design Decisions for RIPS

| Decision | Option A | Option B | Recommendation |
|----------|----------|----------|---------------|
| Guideline representation | Flat text chunks | Graph (Guideline2Graph) | Graph — preserves structure |
| Graph embedding | Euclidean (BioClinicalBERT) | Hyperbolic (geoopt Poincaré) | Euclidean v1; Hyperbolic v2 |
| Retrieval | Flat BM25+dense | PathRAG on graph | PathRAG (relational paths) |
| Reasoning | Single-pass LLM | CoT + self-reflection | CoT + Self-MedRAG loop |
| Multi-guideline | Ignore | Graph rewriting (AIM 2023) | Graph rewriting for conflicts |
| Patient context | None | FHIR integration | FHIR (maps to KG entity types) |
| Evaluation | Custom | MIRAGE benchmark | MIRAGE + guideline-specific |
