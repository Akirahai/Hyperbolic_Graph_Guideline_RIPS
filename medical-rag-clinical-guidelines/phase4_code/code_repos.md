# Phase 4: Code & Tools — Open-Source Ecosystem for Medical RAG

---

## Core RAG Frameworks

### 1. MedRAG Toolkit + MIRAGE Benchmark
- **URL:** https://github.com/Teddy-XiongGZ/MedRAG
- **Benchmark:** https://github.com/Teddy-XiongGZ/MIRAGE
- **Stars:** ~800+
- **Language:** Python
- **Paper:** ACL Findings 2024
- **What it provides:**
  - 5 corpora (PubMed, StatPearls, Textbooks, Wikipedia, MedCorp) pre-chunked and indexed
  - 4 retrievers: BM25, Contriever, SPECTER, MedCPT
  - RRF fusion support
  - MIRAGE benchmark: 7,663 QA pairs from 5 datasets
  - Easy integration with any LLM (OpenAI, HuggingFace)
- **Relevance for RIPS:** Primary evaluation framework. Use MIRAGE to benchmark any guideline RAG system. MedCPT retriever is the best single retriever for biomedical text.

### 2. Microsoft GraphRAG
- **URL:** https://github.com/microsoft/graphrag
- **Stars:** 20,000+ (most popular GraphRAG implementation)
- **Language:** Python
- **Paper:** Edge et al. (2024), arXiv:2404.16130
- **What it provides:**
  - Full pipeline: document ingestion → entity extraction → relationship mapping → community clustering (Leiden algorithm) → hierarchical summaries
  - Global (community-level) + local (entity-level) query modes
  - Parquet-based indexing with Neo4j optional
  - API-first design with Azure OpenAI integration
- **Relevance for RIPS:** Best base for building the GraphRAG layer over clinical guidelines. MED-COPILOT uses a similar pipeline. Start here for the graph construction stage.
- **Limitation:** Designed for OpenAI API; needs adaptation for medical ontologies and Poincaré embedding.

### 3. AMG-RAG
- **URL:** https://github.com/MrRezaeiUofT/AMG-RAG
- **Language:** Python
- **Paper:** EMNLP Findings 2025
- **What it provides:**
  - Medical Entity Recognizer (MER) + PubMed/Wikipedia search integration
  - Neo4j knowledge graph with confidence-scored relationships
  - BFS/DFS traversal with confidence weighting
  - Chain-of-thought reasoning pipeline
  - Evaluation on MEDQA and MEDMCQA
- **Relevance for RIPS:** Most architecturally similar to what RIPS needs. Agentic KG construction + retrieval + reasoning. Can be adapted to use static guideline graphs instead of dynamic PubMed construction.

### 4. Medical-Graph-RAG (ACL 2025)
- **URL:** https://github.com/ImprintLab/Medical-Graph-RAG
- **Language:** Python
- **Paper:** ACL 2025 long paper
- **What it provides:**
  - Triple Graph Construction combining UMLS, MedC-K, and MIMIC-IV datasets
  - U-Retrieval: holistic insights from graph structure
  - Evidence-based response generation for medical LLMs
  - Validated on 9 medical QA benchmarks + 2 fact-checking + 1 long-form generation
  - Docker demo: hub.docker.com/repository/docker/jundewu/medrag-post/general
- **Relevance for RIPS:** Production-grade medical GraphRAG with multi-source KG. Good for understanding the UMLS integration layer.

---

## Knowledge Graph & Ontology Tools

### 5. geoopt — Riemannian Optimization in PyTorch
- **URL:** https://github.com/geoopt/geoopt
- **Stars:** ~2,000
- **Language:** Python (PyTorch)
- **Paper:** arXiv:2005.02819
- **What it provides:**
  - Manifold-aware PyTorch optimizers (Riemannian SGD, Adam)
  - Poincaré ball model, Hyperboloid model, κ-Stereographic model
  - Drop-in replacement for standard PyTorch optimizers
  - Examples: hyperbolic multiclass classification, Poincaré embeddings
- **Relevance for RIPS:** Essential library for implementing Stage 2 of the Hyperbolic HKG Pipeline. Required for training Poincaré embeddings over SNOMED/ICD hierarchy.
- **Usage for RIPS:**
  ```python
  import geoopt
  ball = geoopt.PoincareBall(c=1.0)  # c is learnable
  embedding = geoopt.ManifoldParameter(torch.zeros(n_nodes, dim), manifold=ball)
  optimizer = geoopt.optim.RiemannianAdam([embedding], lr=0.01)
  ```

### 6. hnswlib — Fast Approximate Nearest Neighbor
- **URL:** https://github.com/nmslib/hnswlib
- **Stars:** ~4,000
- **Language:** C++ / Python bindings
- **What it provides:**
  - HNSW (Hierarchical Navigable Small World) indexing
  - Custom distance function support (needed for Poincaré distance)
  - Fast ANN search for million-scale datasets
- **Relevance for RIPS:** Stage 3 of Hyperbolic HKG Pipeline. Wrapping Poincaré distance as custom metric.

### 7. PathRAG
- **URL:** https://github.com/BUPT-GAMMA/PathRAG
- **Language:** Python
- **Paper:** arXiv:2502.14902
- **What it provides:**
  - Flow-based path pruning algorithm on knowledge graphs
  - Node retrieval → path pruning → LLM prompting pipeline
  - Works with any LLM API
  - Supports custom KG construction from documents
- **Relevance for RIPS:** Core graph retrieval algorithm. Apply PathRAG over the clinical guideline graph built by Guideline2Graph to retrieve relational paths (e.g., "Hypertension → complicates → CKD → treated-by → ACEi") for RAG context.

---

## Guideline Digitization Tools

### 8. CPGPrompt
- **URL:** https://github.com/bionlplab/CPGPrompt
- **Language:** Python
- **Paper:** JAMIA 2026 (arXiv:2601.03475)
- **What it provides:**
  - Pipeline to convert narrative CPG text → structured decision tree (JSON)
  - Two node types: Simple Feature Check and Multi-Criteria Check
  - Chatbot execution engine for patient case evaluation
  - Evaluation on 3 clinical domains with synthetic vignettes generator
- **Relevance for RIPS:** Complementary to Guideline2Graph. Use CPGPrompt for linear pathway execution, Guideline2Graph for non-linear KG retrieval. The decision tree output integrates with Neo4j.
- **Missing:** No public repo found for Guideline2Graph (IJCAI 2025) — paper is under anonymous review process.

---

## Medical LLM & Embedding Tools

### 9. MED-COPILOT Demo
- **URL:** https://huggingface.co/spaces/Cryo3978/Med_GraphRAG
- **Language:** Python (HuggingFace Spaces)
- **What it provides:**
  - Live demo of dual GraphRAG + patient case retrieval
  - 40M+ token curated WHO + NICE guideline corpus
  - Similar patient retrieval from 36K cases
  - Token-level saliency visualization
- **Relevance for RIPS:** Reference implementation and demo of the target architecture.

### 10. BioClinicalBERT / BioLinkBERT (HuggingFace)
- **URL:** https://huggingface.co/emilyalsentzer/Bio_ClinicalBERT
- **What it provides:** BERT pretrained on MIMIC clinical notes — best encoder for clinical text (outperforms PubMedBERT on clinical notes)
- **Relevance for RIPS:** Stage 4 (LLM Query) — use for query encoding before projecting to Poincaré space.

### 11. MedCPT — Medical Contrastive Pre-Training
- **URL:** Available via HuggingFace (ncbi/MedCPT-Query-Encoder, ncbi/MedCPT-Article-Encoder)
- **Paper:** Zhang et al. (2023), ACL 2024
- **What it provides:** Contrastively trained query/article encoders on PubMed citation graph — best medical retrieval encoder for PubMed-scale corpora
- **Relevance for RIPS:** Best retriever for guideline text retrieval (outperforms BM25 on MIRAGE for research tasks).

---

## Graph Databases & Infrastructure

### 12. Neo4j
- **URL:** https://neo4j.com
- **What it provides:** Native graph database, Cypher query language, full-text search, vector indexes (Neo4j 5.11+)
- **Used by:** AMG-RAG, Medical-Graph-RAG, MED-COPILOT
- **Relevance for RIPS:** Backend for clinical KG storage. Supports hybrid vector + graph queries for the retrieval stage.

### 13. LanceDB
- **URL:** https://github.com/lancedb/lancedb
- **What it provides:** Column-oriented vector database optimized for hybrid search (vector + metadata filters)
- **Used by:** MED-COPILOT for guideline embeddings
- **Relevance for RIPS:** Alternative to Pinecone/Weaviate for storing hyperbolic embeddings. Custom distance function support needed for Poincaré metric.

---

## Benchmarks & Evaluation

### 14. MedGraphRAG (ACL 2025) Docker Demo
- **URL:** hub.docker.com/repository/docker/jundewu/medrag-post/general
- **Validated on:** 9 medical QA benchmarks + 2 fact-checking datasets
- **Relevance for RIPS:** Turnkey evaluation harness for medical graph RAG systems.

---

## Summary Table

| Repo | Stars | Venue | Purpose for RIPS |
|------|-------|-------|-----------------|
| MedRAG / MIRAGE | 800+ | ACL 2024 | Evaluation benchmark — must use |
| Microsoft GraphRAG | 20,000+ | MS Research | Base GraphRAG pipeline |
| AMG-RAG | ~100 | EMNLP 2025 | Closest architecture to RIPS |
| Medical-Graph-RAG | ~300 | ACL 2025 | Production medical GraphRAG |
| geoopt | 2,000+ | arXiv | Poincaré ball optimization |
| hnswlib | 4,000+ | — | ANN index for Poincaré search |
| PathRAG | ~200 | arXiv | Graph path retrieval algorithm |
| CPGPrompt | ~50 | JAMIA 2026 | CPG → decision tree execution |
| MED-COPILOT demo | — | preprint | Reference clinical RAG demo |
| BioClinicalBERT | — | MIMIC | Query encoder |
| MedCPT | — | ACL 2024 | Best medical retriever |
| Neo4j | — | — | KG storage + query |
| LanceDB | — | — | Vector DB storage |
