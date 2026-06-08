# Medical RAG for Clinical Guideline Agents: A Systematic Literature Review

**Prepared for:** RIPS Project (Hyperbolic Graph Guideline)  
**Date:** June 2026  
**Scope:** Medical RAG systems for creating agents that query clinical practice guidelines to answer patient case queries  
**Papers Reviewed:** 15 (deep dive) from 51 curated from 474 total database entries  

---

## 1. Introduction

Clinical practice guidelines (CPGs) represent the highest level of evidence-based recommendations for medical care. They encode decision logic across thousands of clinical conditions — e.g., NICE UK alone maintains 643 active guidelines totaling over 40 million clinical tokens. Yet despite their authority, CPGs are overwhelmingly accessed by clinicians through manual keyword search or linear reading, creating significant barriers to guideline-concordant care.

The emergence of large language models (LLMs) and retrieval-augmented generation (RAG) creates an opportunity to build **clinical guideline agents** — systems that accept a patient case query in natural language, retrieve the relevant sections from the applicable guideline, and generate a structured clinical recommendation with citations.

Two fundamental challenges define this research area:
1. **Representation**: How to encode multi-level guideline hierarchies (ICD/SNOMED taxonomies with 6–10 levels, hundreds of thousands of concepts) into vector representations that preserve semantic and hierarchical structure
2. **Retrieval**: How to retrieve not just relevant guideline sections but the relational structure between conditions, treatments, and contraindications — especially for patients with multiple comorbidities

This review synthesizes the state of the art across five component areas: (A) medical RAG benchmarks, (B) GraphRAG methods, (C) guideline digitization, (D) hyperbolic/hierarchical embeddings, and (E) agentic orchestration.

---

## 2. Background

### 2.1 Retrieval-Augmented Generation for Medicine

RAG was proposed to overcome LLM hallucination and knowledge staleness by grounding generation in retrieved documents. For medicine, the canonical evaluation framework is **MedRAG/MIRAGE** (Xiong et al., ACL Findings 2024) — 7,663 questions from five datasets (MMLU-Med, MedQA-US, MedMCQA, PubMedQA, BioASQ). MedRAG demonstrates that RAG improves LLMs by up to 18% (GPT-3.5 from 60.69% → 71.57% on MIRAGE), with log-linear scaling of performance with retrieved snippet count (k≤32 optimal), and the "lost-in-the-middle" phenomenon where information in the middle of retrieved context is underutilized.

The best retrieval configuration identified: RRF-4 fusion (BM25 + Contriever + SPECTER + MedCPT) achieves 71.57% MIRAGE average. MedCPT (NCBI, ACL 2024) — a contrastively trained query/article encoder on PubMed citation pairs — is the best individual domain-specific retriever.

### 2.2 Knowledge Graphs for Clinical Decision Support

Knowledge graphs (KGs) encode medical knowledge as entity-relationship triples: (Hypertension, complicates, CKD), (Ramipril, treats, Hypertension + CKD). Structured KGs provide several advantages over flat text retrieval:
- **Relational reasoning**: Multi-hop queries (what treats a disease that complicates Hypertension?)
- **Completeness**: Cross-domain connections not explicit in any single guideline
- **Consistency**: Normalization to UMLS/SNOMED prevents synonym drift

Foundational KG construction work includes EHR-based Medical KG Construction (J. Biomed. Inf. 2023, 81 citations) and Causal KG for Diabetes CDSS (J. Biomed. Inf. 2023, 32 citations), which demonstrates that encoding causal (not just associative) edges improves clinical decision quality.

---

## 3. Five Research Streams and Current State of the Art

### 3.1 Stream A: Medical RAG Benchmarks

**State of the art:** MedRAG/MIRAGE (ACL 2024) is the canonical benchmark. ASTRID (ACL Findings 2025) provides automated scalable evaluation for clinical RAG.

**Best published numbers:** AMG-RAG (EMNLP 2025) achieves F1 74.1% on MEDQA with ~8B parameters, outperforming 70B fine-tuned models (Meditron-70B: 68.3%). MED-COPILOT (preprint 2025) achieves 93.7% MedQA accuracy with Gemini 2.5 + full GraphRAG + patient case pipeline.

**Guideline-specific:** NICE RAG (preprint 2025) achieves 99.5% faithfulness on 300 NICE guidelines with hybrid retrieval + O4-Mini — a 64.7 percentage point improvement over non-RAG baseline.

### 3.2 Stream B: GraphRAG Methods

Microsoft GraphRAG (Edge et al. 2024) established the community-level summarization paradigm: extract entities → map relationships → cluster communities (Leiden algorithm) → generate hierarchical summaries. This enables answering queries that span multiple documents.

**PathRAG** (arXiv:2502.14902) advances this by replacing community retrieval with flow-based relational path pruning. Resource flow propagates from source nodes with decay factor α, generating path reliability scores. PathRAG achieves 60.44% win rate vs. GraphRAG across 6 datasets while reducing token consumption by 16–44%. For clinical settings, path-based retrieval maps naturally to clinical reasoning chains: Symptom → Diagnosis → Treatment → Contraindication.

**AMG-RAG** (EMNLP Findings 2025, arXiv:2502.13010) introduces the agentic KG paradigm: the graph is constructed dynamically per-query rather than pre-built offline. Each medical term extracted from the query triggers PubMed search → node description → LLM relationship inference → Neo4j graph construction. Traversal uses confidence-weighted BFS/DFS: child confidence s(nⱼ) = s(nᵢ) × s(rᵢⱼ). Chain-of-thought reasoning integrates across all retrieved nodes before synthesis. AMG-RAG's dynamic approach eliminates stale knowledge but introduces latency.

**MED-COPILOT** (arXiv:2603.00460) presents the most complete clinical GraphRAG system: 118 WHO + 525 NICE guidelines (40M+ tokens) decomposed into TextUnits, entity-extracted with SNOMED/ICD normalization, relationship-encoded (indication, contraindication, monitoring, escalation), stored in LanceDB with BioClinicalBERT embeddings. Community-level GraphRAG retrieval is combined with a 36,000-case similar-patient retrieval (MIMIC-IV + Synthea). Ablation: GraphRAG adds +0.031 BLEU; patient cases add +0.028 BLEU; combined +0.035 BLEU.

**Medical-Graph-RAG** (ACL 2025, github.com/ImprintLab/Medical-Graph-RAG) introduces Triple Graph Construction combining UMLS (foundational ontology), MedC-K (clinical literature), and MIMIC-IV (patient data) into a unified hierarchical graph with U-Retrieval — validated on 9 medical QA benchmarks.

### 3.3 Stream C: Clinical Guideline Digitization

Converting CPG PDFs into machine-executable structures is the entry problem for guideline-grounded RAG.

**Guideline2Graph** (IJCAI 2025, arXiv:2604.02477) is the current state of the art. Its "decomposition-first" 3-stage VLM pipeline achieves 93.8% node recall, 87.5% edge recall on the NCCN Prostate Cancer guideline — dramatically outperforming AutoKG (78.1% / 16.1%). Key innovations: (1) lookahead boundary detection prevents splitting tables/figures across chunks; (2) BFS expansion with VLM duplicate detection prevents node proliferation; (3) cross-chunk global merge with provenance-aware deduplication. Main limitation: evaluated on a single guideline.

**CPGPrompt** (JAMIA 2026, arXiv:2601.03475) provides a complementary representation: instead of a graph, it extracts a decision tree that an LLM chatbot can traverse. Achieves F1=0.90–1.00 on binary referral decisions across headache, lower back pain, and prostate cancer domains. Key advantage: every traversal step is logged for auditability (critical for clinical regulation compliance). Key limitation: multi-class pathway assignment quality degrades (F1=0.47 for headache, complex branching).

**The integration gap:** Neither paper shows how to use the produced structure for downstream RAG retrieval. Guideline2Graph's output is a decision graph, not a UMLS-normalized knowledge graph suitable for embedding-based retrieval.

**Graph rewriting for concurrent CPGs** (Artificial Intelligence in Medicine, 2023) addresses the critical multi-guideline problem: when a patient has Hypertension + CKD + Diabetes, three separate guidelines may contradict each other. Graph rewriting formalizes the revision process using computational rules that detect and resolve conflicts between concurrently applied guidelines.

### 3.4 Stream D: Hyperbolic and Hierarchical Embeddings

Medical ontologies are intrinsically hierarchical: SNOMED CT has 370,000+ concepts organized in 19 hierarchies with up to 11 levels. ICD-10 has 70,000+ codes in a 4-level tree. Euclidean space cannot efficiently represent such structures: the volume of an n-ball grows polynomially (π^(d/2)r^d / Γ(d/2+1)), while tree volume grows exponentially with level.

**Hyperbolic Embeddings for Health Knowledge Graphs** (ICML 2025 position paper, openreview.net/pdf?id=Sz90WdONPz) argues that hyperbolic space — specifically the Poincaré ball 𝔻ᵈ = {x ∈ ℝᵈ : ‖x‖ < 1} — is theoretically ideal for medical ontologies: volume expands exponentially with radius, mirroring tree branching. Root concepts map near center (‖φ‖ ≈ 0.05), leaf concepts near boundary (‖φ‖ ≈ 0.94). A 5-stage pipeline is proposed: Ontology Ingestion → Hyperbolic Training (geoopt, Riemannian Adam, learnable curvature c) → Vector DB Integration (hnswlib with custom Poincaré distance) → LLM Query (exponential map projection) → Visualization (Poincaré disk with radial layout). Cited prior work shows 10–20% gains in Hits@k over Euclidean embeddings (Nickel & Kiela 2017; Sala et al. 2018; Chami et al. 2020).

The practical efficiency gain: d=5–32 hyperbolic dims achieve the same separation quality as d=128–768 Euclidean dims. This translates to faster retrieval, smaller index footprint, and better generalization.

**Foundational implementations:** Snomed2Vec (arXiv:2019) demonstrated Poincaré embeddings on SNOMED CT. Self-Supervised Graph Learning with Hyperbolic Embedding (IEEE JBHI, 2021) applied Poincaré embeddings to temporal health event prediction. Hyperbolic Hierarchical KG Embeddings for Biological Entities (PubMed 2023) validated the approach on protein-protein interaction and gene-disease prediction.

**The missing piece:** No paper demonstrates end-to-end RAG with hyperbolic embeddings in the retrieval layer, evaluated on clinical QA benchmarks. The position paper claim — that hyperbolic retrieval improves clinical QA — has not been empirically tested.

### 3.5 Stream E: Agentic Orchestration

Static RAG pipelines retrieve once and generate once. Agentic systems introduce loops, dynamic planning, and tool use.

**Self-MedRAG** (arXiv:2601.04531) adds a self-reflection module to standard RAG: hybrid BM25+Contriever retrieval (via RRF) → generate answer with rationale → NLI/LLM verification of groundedness → if insufficient, reformulate query and re-retrieve. Gains: +3.33pp MedQA, +10.72pp PubMedQA.

**Patho-AgenticRAG** (AAAI 2025, arXiv:2508.02258) demonstrates RL-trained agentic routing: a GRPO-trained router dynamically decides whether to invoke RAG, reformulate the query, or classify the tissue type before retrieval. Tissue-aware partitioning (19 classifiers) restricts search scope. Results: +13–38% on pathology VQA benchmarks vs. baselines.

**RAG²** (NAACL 2025) uses rationale-guided retrieval: the LLM generates a clinical rationale for the query first, then uses this rationale as an enhanced retrieval query. This is particularly powerful for conditional clinical logic ("IF CKD stage ≥3 AND eGFR < 45, THEN avoid NSAIDs") where the query may be phrased non-technically.

**FHIR-RAG-MEDS** (arXiv:2509.07706) introduces the critical dimension of patient context: HL7 FHIR resources (Condition, Medication, Observation) are serialized as structured context alongside guideline retrieval. This enables personalized recommendations conditioned on the specific patient's lab values, diagnoses, and medications.

---

## 4. Proposed Architecture for RIPS

Based on the synthesis of all reviewed papers, the following architecture is recommended for the RIPS clinical guideline agent:

### 4.1 Component Design

**Component 1: Guideline Digitization Layer**
- Input: CPG PDFs (MOH Singapore, NICE UK, WHO)
- Method: Guideline2Graph (IJCAI 2025) for multimodal graph extraction
- Output: Decision graph G = (V, E) with typed nodes and transition edges
- Parallel: CPGPrompt (JAMIA 2026) for decision tree representation (auditable pathway execution)
- Post-processing: Entity normalization to SNOMED-CT/ICD-10 via QuickUMLS or MetaMap

**Component 2: Knowledge Graph Construction**
- Tool: Neo4j graph database
- Edges: is-a, complicates, treats, contraindicates, associated-with (from Guideline2Graph) + causal edges from Causal-KG methods
- Drug-condition mapping: integrate ATC drug ontology
- Provenance: all nodes linked to source guideline section (line number, page)

**Component 3: Embedding & Indexing**
- Version 1 (immediate): BioClinicalBERT or MedCPT embeddings → LanceDB vector index; hybrid BM25+dense search
- Version 2 (R&D): geoopt Poincaré ball embeddings (d=16–32) trained via Riemannian Adam on the guideline graph; ANN via hnswlib with custom Poincaré distance
- PathRAG path extraction: on top of Neo4j for relational path retrieval

**Component 4: Patient Context Processor**
- Parse HL7 FHIR resources: Condition (diagnoses), MedicationRequest, Observation (labs), Patient (demographics)
- Map FHIR codes (SNOMED-CT, ICD-10, LOINC) to KG entity types
- Identify applicable guidelines: which guidelines cover the patient's active conditions?

**Component 5: Multi-Condition Query Engine**
- Rationale-guided query expansion (RAG²): generate clinical rationale → use as enhanced query
- PathRAG traversal on clinical KG for each relevant condition
- Cross-guideline conflict detection (Graph Rewriting, AIM 2023)
- Hybrid retrieval: PathRAG paths + flat BM25 chunks as fallback
- Top-k (k=32 per MedRAG findings) ranked by Poincaré distance + edge relevance

**Component 6: Answer Generation & Verification**
- LLM: Claude or GPT-4.1 with strict source-grounding prompts
- Chain-of-thought reasoning over retrieved paths (AMG-RAG style)
- Self-reflection module (Self-MedRAG): NLI verification of answer groundedness; query reformulation if insufficient
- Safety classifier: detect unsafe recommendations; escalate to human clinician
- Output: Structured recommendation + cited guideline sections + confidence score

**Component 7: Evaluation**
- Primary: MIRAGE benchmark (7,663 QA from MedRAG toolkit)
- Secondary: Guideline-concordance evaluation (custom dataset of known correct recommendations)
- Safety: Number of unsafe responses (NICE RAG metric)
- Faithfulness: RAGAs faithfulness score (NICE RAG: 99.5% achievable)

### 4.2 Implementation Roadmap

**Phase 1 (Months 1–3): Baseline pipeline**
- Collect and preprocess 50–100 clinical guidelines (MOH + NICE)
- Run Guideline2Graph to extract decision graphs
- Build Neo4j KG with SNOMED normalization
- BioClinicalBERT embeddings + LanceDB vector index
- Standard RAG (BM25 + dense hybrid) with GPT-4 generation
- Baseline evaluation on MIRAGE + custom QA pairs

**Phase 2 (Months 4–6): Graph RAG enhancement**
- Implement PathRAG on clinical KG
- Add patient context via FHIR integration
- Implement rationale-guided query expansion (RAG²)
- Add self-reflection loop (Self-MedRAG)
- Re-evaluate: expected +8–15pp on clinical guideline QA

**Phase 3 (Months 7–9): Hyperbolic embeddings**
- Train Poincaré ball embeddings on SNOMED/ICD hierarchy using geoopt
- Integrate with hnswlib for approximate nearest-neighbor search
- Compare Euclidean vs. Poincaré retrieval on MIRAGE
- Validate the 10–20% Hits@k claim from ICML 2025 position paper

**Phase 4 (Months 10–12): Multi-guideline conflict resolution**
- Implement graph rewriting rules for concurrent CPG conflicts
- Build benchmark of known inter-guideline contradictions
- Clinical validation with medical professionals

---

## 5. Open Problems

Seven critical gaps were identified in the literature:

1. **No end-to-end hyperbolic RAG pipeline**: The ICML 2025 position paper claim is unvalidated empirically. RIPS can be the first to test this.

2. **Guideline graph → retrieval integration gap**: Guideline2Graph produces excellent graphs but no downstream RAG system uses them. Connecting Guideline2Graph → PathRAG → clinical QA is an open research question.

3. **Multi-guideline conflict resolution**: No automated system handles contradictions between concurrent guidelines at query time.

4. **Real patient query evaluation**: All benchmarks use synthetic or exam-format queries. No dataset of real clinician guideline queries exists publicly.

5. **Dynamic guideline versioning**: Guidelines update annually; no system automates KG updates from official sources.

6. **Multilingual guideline support**: Most systems are English-only; regional guidelines (MOH Singapore, German S3 guidelines) are underserved.

7. **Clinician trust and explainability**: No user study comparing explanation interfaces for clinical trust calibration.

---

## 6. Conclusion

Medical RAG for clinical guideline agents is a rapidly maturing field. The core pipeline components — guideline digitization (Guideline2Graph), knowledge graph construction (Medical-Graph-RAG), graph-based retrieval (PathRAG), and agentic reasoning (AMG-RAG, Self-MedRAG) — are individually state-of-the-art but have not been integrated into a single end-to-end system. MED-COPILOT comes closest (93.7% MedQA, dual GraphRAG+patient retrieval on 40M+ tokens of guidelines) but lacks hyperbolic embeddings and FHIR integration.

The RIPS project's key differentiator is the combination of: (1) Guideline2Graph-quality clinical graph extraction, (2) hyperbolic Poincaré embeddings for efficient hierarchical encoding, and (3) patient-specific FHIR context for personalized recommendations. This combination has not been implemented or evaluated anywhere in the current literature, representing a clear and impactful research contribution.

The most immediately actionable finding: **hybrid BM25+dense retrieval with cross-encoder reranking achieves 99.5% faithfulness on clinical guidelines** (NICE RAG). Even before implementing hyperbolic embeddings, a RIPS v1 using standard dense retrieval on a well-curated guideline corpus will dramatically outperform current clinical practice.

---

## References

[@xiong2024medrag] Xiong et al. (2024). "Benchmarking Retrieval-Augmented Generation for Medicine." ACL Findings 2024. arXiv:2402.13178.

[@zhao2025guideline2graph] Zhao et al. (2025). "Guideline2Graph: Profile-Aware Multimodal Parsing for Executable Clinical Decision Graphs." IJCAI 2025. arXiv:2604.02477.

[@anonymous2025hyperbolic] Anonymous (2025). "Position: Hyperbolic Embeddings Are Essential for Health Knowledge Graphs in LLMs and Vector Databases." ICML 2025 position paper. (preprint)

[@rezaei2025amgrag] Rezaei et al. (2025). "Agentic Medical Knowledge Graphs Enhance Medical Question Answering." EMNLP Findings 2025. arXiv:2502.13010.

[@medcopilot2025] Authors (2025). "MED-COPILOT: A Medical Assistant Powered by GraphRAG and Similar Patient Case Retrieval." arXiv:2603.00460. (preprint)

[@chen2025pathrag] Chen et al. (2025). "PathRAG: Pruning Graph-based Retrieval Augmented Generation with Relational Paths." arXiv:2502.14902. (preprint)

[@fhirrag2025] Authors (2025). "FHIR-RAG-MEDS: Integrating HL7 FHIR with Retrieval-Augmented Large Language Models for Enhanced Medical Decision Support." arXiv:2509.07706. (preprint)

[@niceguidelines2025] Authors (2025). "Grounding Large Language Models in Clinical Evidence: A RAG System for Querying UK NICE Clinical Guidelines." arXiv:2510.02967. (preprint)

[@selfmedrag2025] Authors (2025). "Self-MedRAG: a Self-Reflective Hybrid Retrieval-Augmented Generation Framework for Reliable Medical QA." arXiv:2601.04531. (preprint)

[@cpgprompt2026] Authors (2026). "CPGPrompt: Translating Clinical Guidelines into LLM-Executable Decision Support." JAMIA 33(4):855–862.

[@sohn2025rationale] Sohn et al. (2025). "Rationale-Guided Retrieval Augmented Generation for Medical Question Answering." NAACL 2025.

[@pathoagentic2025] Authors (2025). "Patho-AgenticRAG: Towards Multimodal Agentic Retrieval-Augmented Generation for Pathology." AAAI 2025. arXiv:2508.02258.

[@trumorgpt2025] Authors (2025). "TrumorGPT: Graph-Based Retrieval-Augmented LLM for Fact-Checking." IEEE Transactions on Artificial Intelligence, 2025.

[@logicrag2025] Authors (2025). "You Don't Need Pre-built Graphs for RAG: Retrieval Augmented Generation with Adaptive Reasoning." arXiv:2508.06105.

[@graphrewriting2023] Authors (2023). "Using graph rewriting to operationalize medical knowledge for the revision of concurrently applied clinical practice guidelines." Artificial Intelligence in Medicine.

[@liang2023causal] Liang & Shi (2023). "Causal knowledge graph construction and evaluation for clinical decision support of diabetes." J. Biomedical Informatics.

[@medgraphrag2025] Authors (2025). "Medical Graph RAG: Evidence-based Medical Large Language Model via Graph Retrieval-Augmented Generation." ACL 2025.

[@nickel2017poincare] Nickel & Kiela (2017). "Poincaré Embeddings for Learning Hierarchical Representations." NeurIPS 2017.

[@sala2018hyperbolic] Sala et al. (2018). "Hyperbolic Disk Embeddings for Directed Acyclic Graphs." ICML 2018.
