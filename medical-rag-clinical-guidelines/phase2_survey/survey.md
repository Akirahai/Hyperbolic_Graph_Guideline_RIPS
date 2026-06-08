# Phase 2: Survey — Medical RAG for Clinical Guideline Agents

**Total papers in database: 474 | Relevant to topic: 285 | Peer-reviewed: 153**

---

## Cluster A: Medical RAG Benchmarks & Foundational Frameworks

These papers define the evaluation infrastructure and establish what RAG systems can and cannot do in clinical settings.

### A1. MedRAG / MIRAGE (ACL Findings 2024)
- **Key paper**: "Benchmarking Retrieval-Augmented Generation for Medicine" (Xiong et al., 2024)
- arXiv:2402.13178 | ~200+ citations
- Introduces MIRAGE: 7,663 QA pairs from 5 medical datasets (MedQA, MedMCQA, PubMedQA, BioASQ, MMLU-Med)
- Tests 41 combinations: 4 corpora × 4 retrievers × multiple LLMs
- Key finding: MedRAG improves LLMs by up to 18%, reveals "lost-in-the-middle" effect
- **Log-linear scaling**: retrieval quality scales log-linearly with corpus size
- Canonical benchmark for this field

### A2. Rethinking RAG for Medicine (preprint 2025)
- "Rethinking Retrieval-Augmented Generation for Medicine: A Large-Scale, Systematic Expert Evaluation"
- arXiv:2511.06738
- Expert clinical evaluation of RAG systems — critiques automated metrics
- Finds that automated scores correlate poorly with clinician assessments

### A3. RAG Variants for Clinical Decision Support (MDPI Electronics 2025)
- "Evaluating Retrieval-Augmented Generation Variants for Clinical Decision Support"
- Tests 12 RAG variants: dense, sparse, hybrid, graph-based, multimodal, self-reflective, adaptive, security-focused
- On 250 de-identified patient vignettes
- **Finding**: Hybrid + graph-based RAG outperforms flat chunk retrieval for complex multi-condition patients

### A4. ASTRID — Automated Scalable Evaluation (ACL Findings 2025)
- "ASTRID — An Automated and Scalable TRIaD for the Evaluation of RAG-based Clinical QA Systems"
- Scalable automated evaluation framework for clinical RAG systems
- Addresses lack of gold-standard clinical QA evaluation

---

## Cluster B: GraphRAG Methods

Structured graph retrieval shows consistent superiority over flat retrieval for hierarchical and relational medical knowledge.

### B1. PathRAG (SIGIR 2025, 56 citations)
- "PathRAG: Pruning Graph-based Retrieval Augmented Generation with Relational Paths"
- Prunes graph retrieval by selecting relational paths rather than full subgraphs
- Reduces token overhead while preserving reasoning chains
- General-purpose but directly applicable to clinical KGs

### B2. TrumorGPT (IEEE Trans. AI 2025, 34 citations)
- Graph-Based RAG LLM for Fact-Checking
- Uses biomedical knowledge graphs as grounding for factual verification
- Multi-hop reasoning on KG subgraphs

### B3. MedRAG: KG-Elicited Reasoning (TheWebConf/WWW 2025)
- "MedRAG: Enhancing RAG with Knowledge Graph-Elicited Reasoning for Healthcare Copilot"
- ACM Web Conference 2025
- Integrates KG reasoning explicitly into the RAG loop (not just retrieval, but reasoning)
- Proposed for healthcare copilot applications

### B4. Agentic Medical KG — AMG-RAG (EMNLP Findings 2025)
- "Agentic Medical Knowledge Graphs Enhance Medical Question Answering"
- arXiv:2502.13010
- **Key innovation**: automates KG construction + continuous updates + reasoning integration
- F1 74.1% on MEDQA, 66.34% on MEDMCQA
- Closest to the RIPS project architecture

### B5. Agentic RAG with KG for Multi-Hop Reasoning (preprint 2025)
- arXiv:2507.16507
- Complex multi-hop queries requiring graph traversal across concept nodes
- Addresses limitation of flat RAG for "patient with Hypertension AND CKD AND T2DM" type queries

### B6. CG-RAG (SIGIR 2025, 14 citations)
- "CG-RAG: Research Question Answering by Citation Graph Retrieval-Augmented LLMs"
- Uses citation network structure for RAG — graph traversal over citation DAGs
- Applicable to clinical guideline citation networks

### B7. Self-Correcting Agentic Graph RAG for Hepatology (PMC 2025)
- Clinical application of agentic graph RAG with self-correction loop
- Hepatology domain: liver disease guidelines
- Published in peer-reviewed clinical informatics journal

---

## Cluster C: Clinical Guideline Digitization & Structuring

Converting CPGs from unstructured PDFs into machine-executable formats.

### C1. Guideline2Graph (IJCAI 2025) — USER REFERENCE
- "Guideline2Graph: Profile-Aware Multimodal Parsing for Executable Clinical Decision Graphs"
- **3-stage pipeline**: chunk → per-chunk graph → global merge via VLM
- Tested on NCCN Prostate Cancer guideline
- Node recall 93.8%, edge precision 69.0%, edge recall 87.5% vs AutoKG (78.1%, 19.6%, 16.1%)
- Key limitation: tested on ONE guideline document; scalability unknown

### C2. CPGPrompt (2026, peer-reviewed)
- "CPGPrompt: Translating Clinical Guidelines into Large Language Model-Executable Decision Support"
- Most recent work in guideline digitization — directly translates CPG text into LLM-executable format
- **Differentiator**: focuses on LLM execution (prompt engineering), not graph structure

### C3. Graph Rewriting for Concurrent CPGs (Artificial Intelligence in Medicine 2023)
- "Using graph rewriting to operationalize medical knowledge for the revision of concurrently applied clinical practice guidelines"
- Formal computational method for handling concurrent (conflicting) guidelines
- **Key insight**: When a patient has multiple conditions, guidelines may contradict — graph rewriting formalizes conflict resolution

### C4. Decision Tree Extraction for CDSS (2025, peer-reviewed)
- "Decision Tree Extraction for Clinical Decision Support System With If-Else Pseudocode"
- Extracts decision trees from clinical text using LLMs
- Simpler representation than full graphs but directly executable

### C5. LLM-guided Clinical Decision Tree Extraction (2026)
- "Large language model-guided extraction of explainable clinical decision trees from complex clinical notes"
- Most recent (2026) — extends decision tree extraction to clinical notes (not just guidelines)

### C6. Extracting Clinical Guideline Information (PubMed/NLM 2025)
- Evaluation study of LLM extraction from clinical guidelines
- Two LLMs compared on guideline extraction accuracy
- **Finding**: LLMs achieve high recall but lower precision on structured rule extraction

---

## Cluster D: Medical Knowledge Graph Construction

Building KGs from heterogeneous clinical sources.

### D1. EHR-Based Medical KG Construction (J. Biomed. Inf. 2023, 81 citations)
- "Towards electronic health record-based medical knowledge graph construction, completion, and applications"
- Foundational survey: construction → completion → downstream applications
- **Most cited medical KG paper** in our database

### D2. Causal KG for Diabetes CDSS (J. Biomed. Inf. 2023, 32 citations)
- "Causal knowledge graph construction and evaluation for clinical decision support of diabetes"
- Builds causal (not just associative) KG from clinical narratives
- Explicitly encodes causal edges for diabetes management

### D3. KG Enrichment from Clinical Narratives (Int. J. Med. Info. 2023, 41 citations)
- "Knowledge graph enrichment from clinical narratives using NLP, NER, and biomedical ontologies"
- NLP pipeline for KG construction with UMLS, SNOMED alignment

### D4. LLM-Driven KG in Sepsis Care (JMIR 2024, 21 citations)
- Multi-center LLM-driven KG construction for sepsis protocol management
- Practical deployment: KG updated from multicenter clinical data

### D5. Clinical KG with Multi-LLMs via RAG (2026 preprint)
- "Clinical Knowledge Graph Construction and Evaluation with Multi-LLMs via Retrieval-Augmented Generation"
- Uses RAG to bootstrap KG construction — using existing KG to construct expanded KG (circular improvement)

### D6. BioKGrapher (2024, 8 citations)
- "BioKGrapher: Initial evaluation of automated knowledge graph construction from biomedical text"
- Automated pipeline for biomedical KG with LLMs
- Evaluation framework for automated KG quality

---

## Cluster E: Hyperbolic & Hierarchical Embeddings

Encoding deep medical ontologies with geometry-aware representations.

### E1. Hyperbolic Embeddings for Health Knowledge Graphs (ICML 2025 position) — USER REFERENCE
- Argues hyperbolic space is essential for HKGs in LLM-driven pipelines
- Proposes 5-stage pipeline: Ontology Ingestion → Hyperbolic Training → VDB Integration → LLM Query → Visualization
- 10-20% gains in Hits@k retrieval vs Euclidean
- Uses geoopt (PyTorch Riemannian optimization), hnswlib for ANN

### E2. Self-Supervised Graph Learning with Hyperbolic Embedding (IEEE JBHI 2021)
- Hyperbolic Poincaré embeddings for temporal health event prediction
- Learns patient trajectory representations in Poincaré disk
- Shows hierarchical disease progression maps

### E3. Snomed2Vec (arXiv 2019)
- "Snomed2Vec: Random Walk and Poincaré Embeddings of a Clinical Knowledge Base"
- First systematic application of Poincaré embeddings to SNOMED CT
- Foundational for hyperbolic medical ontology embedding

### E4. Hyperbolic Hierarchical KG Embeddings for Biological Entities (PubMed 2023)
- Evaluates hyperbolic embeddings on biological KGs (gene-disease, protein-protein)
- Confirms hierarchical structure preservation advantage

### E5. Multi-Ontology Integration with Dual-Axis Propagation (2025 preprint)
- arXiv:2508.21320
- Integrates SNOMED/ICD/UMLS into unified representation using dual-axis propagation
- Addresses the multi-ontology alignment challenge in the Guideline2Graph → HKG pipeline

---

## Cluster F: Agentic Medical Systems

End-to-end medical AI agents combining retrieval, reasoning, and tool use.

### F1. MED-COPILOT (arXiv 2025)
- arXiv:2603.00460
- **Most comprehensive pipeline**: GraphRAG + similar patient case retrieval
- Curates 118 WHO + 525 NICE guidelines (40M+ clinical tokens)
- Interactive transparent clinical reasoning interface
- Directly comparable to RIPS project goals

### F2. Patho-AgenticRAG (AAAI 2025, 9 citations)
- "Patho-AgenticRAG: Towards Multimodal Agentic Retrieval-Augmented Generation for Pathology"
- AAAI Workshop 2025 — multimodal (image + text) agentic RAG for pathology
- Demonstrates agentic orchestration of retrieval + reasoning + tool-use

### F3. FHIR-RAG-MEDS (preprint 2025)
- arXiv:2509.07706
- Integrates HL7 FHIR (patient record standard) with RAG-based LLM system
- **Critical gap addressed**: patient record context + guideline retrieval (not just pure QA)
- Most directly implements the "patient case → guideline query" pipeline

### F4. Self-MedRAG (preprint 2025)
- arXiv:2601.04531
- Self-reflective hybrid RAG: critiques own retrieval and generation
- Special tokens for retrieval necessity, relevance, factuality
- Reduces hallucination by 15% compared to standard RAG

### F5. CORE-Acu (preprint 2026)
- arXiv:2603.08321
- LLM + KG safety verification for acupuncture clinical decision support
- Structured reasoning traces + KG grounding to combat black-box hallucination

### F6. SEMA-RAG (preprint 2026)
- arXiv:2605.17101
- Self-Evolving Multi-Agent RAG framework for medical QA
- Iterative retrieval with multi-agent verification

### F7. AI-Based CDS for Primary Care — Real World (preprint 2025)
- arXiv:2507.16947
- Live deployment evaluation — AI consultation system at Penda Health clinics (Nairobi)
- Real-world performance metrics, not just benchmark scores

---

## Cluster G: Clinical QA & Evaluation

Task-specific evaluation and systems for clinical guideline QA.

### G1. NICE Clinical Guidelines RAG (preprint 2025)
- arXiv:2510.02967
- RAG system specifically for UK NICE clinical guidelines
- O4-Mini + RAG: 99.5% faithfulness (up from 34.8% without RAG) — 64.7pp gain
- Demonstrates guideline-specific RAG dramatically outperforms pure LLM

### G2. Rationale-Guided RAG for Medical QA (NAACL 2025)
- "Rationale-Guided Retrieval Augmented Generation for Medical Question Answering"
- NAACL 2025 long paper
- Uses rationales (explanations) as query expansion signals for retrieval
- Improves both recall and answer quality

### G3. Performance of RAG LLMs in Guideline-Concordant Answers (JMIR 2025, 6 citations)
- Direct evaluation of whether RAG LLMs give guideline-concordant answers
- Clinical guideline adherence as primary metric (not generic QA)

### G4. RAG Evaluation in Clinical Guidelines (European Spine J 2025, 6 citations)
- "Evaluation of retrieval-augmented generation and large language models in clinical guideline adherence"
- Spine surgery domain — highly specific guideline adherence evaluation

### G5. Rationale-Guided Medical QA (ACL 2024)
- Condition-Gated Reasoning for Context-Dependent Biomedical QA
- Handles conditional logic in medical QA ("IF patient has CKD THEN...")
- Relevant to executing conditional guideline logic

---

## Summary: Landscape Coverage

| Cluster | Papers | Peer-Reviewed | Key Venues |
|---------|--------|---------------|------------|
| A: Medical RAG Benchmarks | 8 | 5 | ACL 2024, MDPI, PMC |
| B: GraphRAG Methods | 9 | 6 | SIGIR 2025, EMNLP 2025, WWW 2025 |
| C: Guideline Digitization | 8 | 6 | IJCAI 2025, AIM 2023 |
| D: KG Construction | 7 | 5 | J. Biomed. Inf, JMIR |
| E: Hyperbolic Embeddings | 5 | 3 | ICML 2025, IEEE JBHI |
| F: Agentic Systems | 8 | 2 | AAAI 2025, arXiv |
| G: Clinical QA & Eval | 6 | 4 | NAACL 2025, JMIR |
| **Total (unique)** | **51** | **31** | |

---

## Papers Flagged for Deep Dive (Phase 3)

1. Guideline2Graph (user reference, IJCAI 2025)
2. Hyperbolic HKG Pipeline (user reference, ICML 2025 position)
3. MedRAG/MIRAGE benchmark (ACL Findings 2024)
4. Agentic Medical KG — AMG-RAG (EMNLP Findings 2025)
5. MED-COPILOT (arXiv 2025) — most comprehensive pipeline
6. FHIR-RAG-MEDS (arXiv 2025) — patient record + guideline RAG
7. PathRAG — pruning graph RAG (SIGIR 2025)
8. CPGPrompt — guideline to LLM-executable (2026)
9. NICE Clinical Guidelines RAG (arXiv 2025)
10. Self-MedRAG — self-reflective hybrid RAG (arXiv 2025)
11. Causal KG for Diabetes CDS (J. Biomed. Inf 2023) — foundational clinical KG
12. Graph rewriting for concurrent CPGs (AIM 2023) — conflict resolution
13. Rationale-Guided RAG (NAACL 2025)
14. Self-Correcting Agentic Graph RAG for Hepatology (PMC 2025)
15. Multi-Ontology Integration (preprint 2025)
