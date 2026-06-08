# Phase 3: Deep Dive — Detailed Paper Notes

**15 papers read in full via ar5iv HTML or fetched from official sources**

---

## Paper 1: Guideline2Graph (IJCAI 2025) — USER REFERENCE

**Citation:** Zhao et al. (2025). "Guideline2Graph: Profile-Aware Multimodal Parsing for Executable Clinical Decision Graphs." IJCAI 2025. arXiv:2604.02477.

**Problem:** Clinical practice guidelines are long, multimodal PDFs with branching logic. LLM/VLM one-shot graph extraction fails on full documents because (a) context windows overflow, (b) cross-page control flow is not captured, (c) nodes duplicate across chunks.

**Core Contribution:** "Decomposition-First" 3-stage pipeline using a VLM as a structured-output oracle at each stage.

**Methodology:**
- **Stage 1 (Chunking):** Extract document profile (patient population, scope). Classify pages as CORE/AUXILIARY. Group consecutive CORE pages into chunks with lookahead boundary detection to avoid splitting tables/figures. Each chunk annotated with entry nodes R and exit nodes Z.
- **Stage 2 (Local Graph):** BFS expansion from entry nodes. For each node, retrieve top-k cosine neighbors, run VLM verifier to detect duplicates (using node + ancestor context + neighbor set triplet), generate children with transition conditions. Terminals Z are fixed and cannot be expanded.
- **Stage 3 (Global Merge):** Seed queue with all interface nodes (R ∪ Z) across chunks. For each, search cross-chunk candidates for duplicates via VLM semantic equivalence check. Merge duplicates by preferring non-terminals and earlier chunks.

**Results (on NCCN Prostate Cancer Guideline):**
| Method | Node Prec | Node Rec | Edge Prec | Edge Rec | Triplet Prec | Triplet Rec |
|--------|-----------|----------|-----------|----------|--------------|-------------|
| Doc2KG | 27.5% | 43.8% | 1.1% | 1.8% | 1.1% | 1.8% |
| AutoKG | 56.8% | 78.1% | 19.6% | 16.1% | 19.6% | 16.1% |
| **Guideline2Graph** | **57.7%** | **93.8%** | **69.0%** | **87.5%** | **69.0%** | **87.5%** |

Edge precision+recall dramatically better: +49.4pp precision, +71.4pp recall over AutoKG.

**Limitations:**
- Evaluated on ONE guideline (NCCN Prostate Cancer V4.2024)
- VLM identity not disclosed (reproducibility concern)
- No benchmark dataset (manually curated ground-truth for one document)
- Does not specify how the produced graph connects to a downstream RAG/retrieval system

**Connections:** Directly enables Stage 1 input for the Hyperbolic HKG Pipeline (Paper 2). CPGPrompt (Paper 14) is an alternative representation approach.

---

## Paper 2: Hyperbolic Embeddings for Health Knowledge Graphs (ICML 2025 position) — USER REFERENCE

**Citation:** Anonymous (2025). "Position: Hyperbolic Embeddings Are Essential for Health Knowledge Graphs in LLMs and Vector Databases." ICML 2025 position paper. OpenReview:Sz90WdONPz.

**Problem:** Biomedical retrieval relies on Euclidean embeddings that cannot efficiently encode ICD/SNOMED hierarchies (6–10 levels deep, thousands of leaves). In Euclidean space, hierarchical separation requires O(n) dimensions; in hyperbolic space, O(log n) dimensions suffice.

**Core Contribution:** Position paper + 5-stage pipeline proposal.

**Theory:**
- Poincaré ball 𝔻ᵈ = {x ∈ ℝᵈ : ‖x‖ < 1}, with curvature c < 0
- Volume grows exponentially with radius → mirrors tree branching structure
- Parent concepts cluster near center (‖φ‖ ≈ 0.05–0.35), leaf concepts near boundary (‖φ‖ ≈ 0.87–0.94)
- Poincaré distance: cosh⁻¹(1 + 2‖x−y‖²/((1−‖x‖²)(1−‖y‖²)))

**Five-Stage Pipeline:**
1. **Ontology Ingestion:** Normalize SNOMED/ICD/UMLS → unified graph G₀ with typed nodes (entity) and edges (is-a, associated-with, treats, complicates)
2. **Hyperbolic Embedding Training:** Riemannian gradient descent (Bonnabel 2013), geoopt library, curvature c as trainable parameter, d=5–32 dims vs Euclidean d=128–768
3. **Vector DB Integration:** hnswlib with custom Poincaré distance; exponential map projection for approximate Euclidean indexing
4. **LLM Query:** Query → exponential map → Poincaré ball → ANN search → top-k nodes → RAG prompt assembly
5. **Visualization:** Poincaré disk with radial layout, subtree highlighting, cross-branch arc display

**Empirical Claims:** 10–20% gains in Hits@k when switching from Euclidean to Poincaré distance (citing Nickel & Kiela 2017, Sala et al. 2018, Chami et al. 2020).

**Key Advantages:**
- 5–32 hyperbolic dims vs 128–768 Euclidean for same quality → 3–26× compression
- Hierarchical separation preserved without crowding (exponential capacity growth)
- Cross-branch edges (HTN ↔ CKD) encoded as geodesic shortcuts

**Limitations:**
- Position paper: no new experiments, cites prior work
- ANN search with Poincaré distance requires custom implementation (hnswlib doesn't natively support it)
- Riemannian optimization is slower and harder to tune than Adam in Euclidean space

---

## Paper 3: MedRAG / MIRAGE Benchmark (ACL Findings 2024)

**Citation:** Xiong et al. (2024). "Benchmarking Retrieval-Augmented Generation for Medicine." ACL Findings 2024. arXiv:2402.13178.

**Problem:** No systematic evaluation of optimal RAG configurations for medical applications. Prior work is fragmented across different corpora, retrievers, and LLMs.

**Core Contribution:** MIRAGE — Medical Information Retrieval-Augmented Generation Evaluation.

**Benchmark Design:**
- 7,663 questions from 5 datasets: MMLU-Med (1,089), MedQA-US (1,273), MedMCQA (4,183), PubMedQA (500), BioASQ-Y/N (618)
- 4 evaluation requirements: zero-shot, multi-choice format, RAG architecture, question-only retrieval
- 5 corpora: PubMed (23.9M chunks), StatPearls (301K), Textbooks (125.8K), Wikipedia (29.9M), MedCorp (combined)
- 4 retrievers: BM25, Contriever, SPECTER, MedCPT; + RRF-2, RRF-4 fusions
- 6 LLMs: GPT-4, GPT-3.5, Mixtral, Llama2, MEDITRON, PMC-LLaMA

**Key Results:**
- MedRAG improves GPT-3.5 by +17.9% (60.69% → 71.57%); Mixtral by +13.1% (61.42% → 69.48%)
- GPT-3.5 + MedRAG reaches GPT-4-level performance
- RRF-4 (fusion of 4 retrievers) achieves SOTA at 71.57% on MIRAGE
- BM25 and MedCPT best individual retrievers
- Log-linear scaling: performance scales log-linearly with number of retrieved snippets (k≤32)
- "Lost-in-the-middle": accuracy drops when ground-truth appears in middle of context

**Practical Recommendations:**
- Use MedCorp (combined corpus) for robustness; PubMed for research tasks
- k=32 retrieved snippets is optimal; diminishing returns beyond that
- RRF-2 (BM25+MedCPT) is best cost-effective retriever combination

**Limitations:** Vanilla RAG only (no graph-based, no self-reflective); multi-choice format constraint; rationale quality not evaluated.

**Relevance to RIPS:** Defines evaluation protocol — RIPS should be evaluated on MIRAGE + guideline-specific QA. Shows that retriever quality matters enormously and that corpus specificity matters.

---

## Paper 4: AMG-RAG — Agentic Medical KG (EMNLP Findings 2025)

**Citation:** (Authors). "Agentic Medical Knowledge Graphs Enhance Medical Question Answering: Bridging the Gap Between LLMs and Evolving Medical Knowledge." EMNLP Findings 2025. arXiv:2502.13010.

**Problem:** Static LLMs cannot track evolving medical knowledge. Manual KG maintenance is expensive. Existing KG-RAG systems have rigid retrieval not adaptive to query structure.

**Core Contribution:** AMG-RAG — automated KG construction + adaptive traversal + CoT reasoning, all agent-driven.

**Architecture (4 components):**
1. **Question Parsing:** LLM extracts medical terms {n₁, ..., nₘ} from query (max M terms)
2. **Node Exploration:** Traverses Neo4j MKG with confidence-weighted BFS/DFS. Child confidence: s(nⱼ) = s(nᵢ) · s(rᵢⱼ). Terminates at threshold τ or document limit.
3. **CoT Generation:** For each entity, generates reasoning trace cᵢ integrating node descriptions and relationships
4. **Answer Synthesis:** Aggregates {c₁, ..., cₘ} into final answer â with confidence score ŝ

**KG Construction (dynamic per query):**
- MER (Medical Entity Recognizer) → PubMed search → node descriptions d(nᵢ)
- LLM infers relationships: rᵢⱼ, sᵢⱼ = LLM(d(nᵢ), d(nⱼ))
- Confidence rubric: 10 = direct/strong, 7-9 = moderate, 4-6 = weak, 1-3 = minimal
- Neo4j stores nodes + relationships + confidence scores

**Results:**
- MEDQA: F1 74.1%, Accuracy 73.9% (vs Meditron-70B: F1 68.3%) — surpasses 70B with ~8B params
- MEDMCQA: Accuracy 66.34% (vs Meditron-70B: 66.0%)
- With PubMedSearch: 73.92% accuracy; Without search: 67.16% (−6.76pp ablation gap)
- Without CoT: 66.69% (−7.23pp ablation gap)
- Expert validation: 8.9/10 node quality, 8.8/10 relationship relevance

**Limitations:**
- Latency from external PubMed search calls during KG construction
- No direct integration with clinical treatment guidelines (identified as future work)
- Domain: rapidly evolving medical literature; not tested on static guideline execution

**Key Insight for RIPS:** The dynamic KG construction is powerful but expensive (per-query PubMed calls). For RIPS, the KG could be pre-built from guidelines and updated periodically rather than per-query.

---

## Paper 5: MED-COPILOT (preprint 2025)

**Citation:** Authors. "MED-COPILOT: A Medical Assistant Powered by GraphRAG and Similar Patient Case Retrieval." arXiv:2603.00460.

**Problem:** Clinical decisions require integration of (1) evidence-based guidelines and (2) analogical reasoning from similar past cases. Existing systems address only one pathway.

**Core Contribution:** Dual-pathway clinical decision support: GraphRAG over guidelines + case-based patient retrieval.

**Architecture:**
- **Guideline Layer:** 118 WHO + 525 NICE guidelines (40M+ tokens). Graph construction via: entity extraction → SNOMED-CT/ICD-10 normalization → relation encoding (indication, contraindication, monitoring, escalation) → provenance linking. Indexed in LanceDB with BioClinicalBERT embeddings. Community-level GraphRAG retrieval.
- **Patient Case Layer:** 18K MIMIC-IV ICU cases (rule-converted to SOAP) + 18K Synthea cases (agent-generated SOAP). Hybrid similarity: keyword-conditioned (diagnoses, comorbidities, interventions) + semantic embedding. Dual scoring satisfies eligibility constraints while robustly matching trajectories.
- **Reasoning:** Claude (or GPT-4.1-mini, Gemini 2.5) with retrieved evidence; token-level saliency visualization.

**Results:**
| Model | ROUGE-L | BERTScore-F1 | MedQA | MMLU-Clinical |
|-------|---------|--------------|-------|---------------|
| Gemini 2.5 (base) | 0.147 | 0.829 | 0.867 | 0.875 |
| Gemini 2.5 + RAG | 0.214 | 0.833 | 0.892 | 0.905 |
| Gemini 2.5 + Full | **0.300** | **0.862** | **0.937** | **0.971** |

Ablation: GraphRAG adds +0.031 BLEU; patient retrieval adds +0.028 BLEU; combined = +0.035 BLEU (slightly subadditive).

**Limitations:**
- MIMIC+Synthea creates mixed distribution; Synthea cases are synthetic
- Graph abstraction may miss contextual nuances
- No gold-standard patient similarity retrieval metric exists

**Key Insight for RIPS:** This is the closest existing system to the RIPS project. The dual GraphRAG+case-based pathway is the most complete clinical decision support architecture. The 40M-token guideline corpus (WHO+NICE) is the largest curated collection in the literature.

---

## Paper 6: PathRAG (preprint 2025, highly cited)

**Citation:** Chen et al. (2025). "PathRAG: Pruning Graph-based Retrieval Augmented Generation with Relational Paths." arXiv:2502.14902.

**Problem:** Existing GraphRAG methods (GraphRAG, LightRAG) retrieve overly broad community subgraphs, introducing noise and token overhead. Flat entity lists lose relational structure.

**Core Contribution:** Flow-based path pruning to extract key relational chains from KGs for RAG.

**Methodology:**
- **Stage 1 - Node Retrieval:** LLM extracts keywords → semantic embedding matching → top-N nodes
- **Stage 2 - Path Retrieval (flow-based pruning):** Resource flow propagates from source nodes with decay α divided by out-degree. Distance-aware penalty applies. Early stopping at threshold θ. Path reliability = average edge resource value.
- **Stage 3 - Answer Generation:** Retrieved paths converted to text, sorted ascending by reliability score, inserted into prompt.

**Results (6 datasets: Agriculture, Legal, History, CS, Biology, Mix):**
- Average win rate vs GraphRAG: 60.44% (range: 55.28%–66.13%)
- Average win rate vs LightRAG: 58.46% (range: 55.20%–63.71%)
- Token efficiency: 16–44% reduction in token consumption

**Clinical Relevance:** Path-based retrieval naturally maps to clinical reasoning chains: Symptom → Diagnosis → Treatment → Contraindication. Preserves the "why" of connections between nodes, making recommendations explainable.

---

## Paper 7: FHIR-RAG-MEDS (preprint 2025)

**Citation:** Authors. "FHIR-RAG-MEDS: Integrating HL7 FHIR with Retrieval-Augmented Large Language Models for Enhanced Medical Decision Support." arXiv:2509.07706.

**Problem:** Clinical decision support requires patient-specific context (structured EHR) + evidence-based guidelines. Current RAG systems use one but not both.

**Core Contribution:** Integration of HL7 FHIR patient records (structured data standard) with RAG-based LLM for personalized medical decision support.

**Architecture:**
- FHIR patient record (structured: diagnoses, medications, lab values, demographics) → serialized to text context
- RAG layer retrieves relevant guideline passages based on patient's conditions
- LLM generates personalized recommendation conditioned on both FHIR context + guideline retrieval

**Key Insight for RIPS:** FHIR integration is critical for making RAG systems patient-specific. A patient with HTN + CKD + T2DM should query guidelines for all three conditions simultaneously. FHIR resource types (Condition, Medication, Observation) map naturally to KG entity types.

---

## Paper 8: NICE Clinical Guidelines RAG (preprint 2025)

**Citation:** Authors. "Grounding Large Language Models in Clinical Evidence: A Retrieval-Augmented Generation System for Querying UK NICE Clinical Guidelines." arXiv:2510.02967.

**Problem:** Healthcare professionals cannot quickly query NICE guidelines (some exceed 100 pages) under time pressure.

**System Architecture:**
- 300 NICE guidelines → XML to Markdown → hierarchical semantic chunking (chunks ≥600 tokens split at logical breakpoints; chunks ≤200 tokens merged) → 10,195 text chunks
- Hybrid retrieval: BM25 sparse + dense (Voyage-3-Large / text-embedding-3-large)
- Cross-encoder reranking
- LLMs: GPT-4.1, O4-Mini, Claude Sonnet 4 with strict source-grounding prompts

**Results:**
- O4-Mini + RAG faithfulness: 99.5% (vs baseline 34.8% without RAG) = +64.7pp gain
- Context Precision: 1.0 (perfect grounding) for all RAG models
- GPT-4.1: 98.7% accuracy, 67% reduction in unsafe responses vs O4-Mini
- MRR: 0.814; Recall@1: 81%; Recall@10: 99.1%

**Evaluation:** 70 manually curated QA pairs + synthetic QA generation (7,901 pairs); 7 Subject Matter Experts validated clinical safety.

**Limitations:** Synthetic queries may not reflect real clinician patterns; multi-source queries not tested; errors in source guidelines propagate faithfully.

**Key Insight for RIPS:** Shows that pure text-chunk RAG on well-curated guidelines can achieve near-perfect faithfulness. The limiting factor is not retrieval quality but guideline accuracy. Cross-encoder reranking is essential (improves precision significantly). 300 guidelines is a reasonable scale.

---

## Paper 9: Self-MedRAG (preprint 2025)

**Citation:** Authors. "Self-MedRAG: a Self-Reflective Hybrid Retrieval-Augmented Generation Framework for Reliable Medical Question Answering." arXiv:2601.04531.

**Problem:** Standard RAG retrieves and generates in one pass without evaluating retrieval quality or answer groundedness. Medical hallucinations have life-safety implications.

**Architecture:**
- **Hybrid Retrieval:** BM25 (sparse) + Contriever (dense) fused via Reciprocal Rank Fusion (RRF)
- **Self-Reflection Module:** NLI-based or LLM-based verification of whether generated answer is grounded in retrieved context
- **Iterative Refinement:** If reflection identifies insufficient grounding → query reformulation → re-retrieval → re-generation (loop)

**Results:**
- MedQA: 80.00% → 83.33% (+3.33pp)
- PubMedQA: 69.10% → 79.82% (+10.72pp)

**Key Insight for RIPS:** Self-reflection is critical for safety in clinical settings. Iterative retrieval with query reformulation handles complex conditional queries better than single-pass RAG.

---

## Paper 10: CPGPrompt (JAMIA 2026)

**Citation:** Authors. "CPGPrompt: Translating Clinical Guidelines into LLM-Executable Decision Support." JAMIA 33(4):855–862, 2026. arXiv:2601.03475.

**Problem:** Clinical guidelines are narrative text that cannot be directly executed by LLMs without structured intermediary.

**Approach:**
- Extract decision tree from CPG PDF using LLM with structured prompting
- Two node types: Simple Feature Check Nodes (single condition) + Multi-Criteria Check Nodes (multi-condition with threshold)
- Chatbot navigation agent traverses tree with structured yes/no queries to evaluate patient cases
- Full audit trail logged for transparency

**Results:**
- Binary referral classification:
  - Headache: F1=0.90, Recall=1.00
  - Lower back pain: F1=1.00, Recall=1.00
  - Prostate cancer: F1=1.00, Recall=1.00
- Multi-class pathway: Headache F1=0.47; Back pain F1=0.72; Prostate cancer F1=0.77

**Key Differences vs Guideline2Graph:**
| Aspect | CPGPrompt | Guideline2Graph |
|--------|-----------|-----------------|
| Output | Decision tree (if-else) | Labeled directed graph |
| Execution | LLM chatbot traversal | RAG retrieval |
| Multi-modal | Text only | Text + Figures (VLM) |
| Evaluation | Multi-domain | Single guideline, higher accuracy |
| Scalability | Manual validation needed | Automated pipeline |

**Key Insight for RIPS:** CPGPrompt is better for sequential guideline execution (clinical pathway following). Guideline2Graph is better for non-linear retrieval queries. RIPS could use CPGPrompt's decision tree structure as a "navigation layer" over Guideline2Graph's output graph.

---

## Paper 11: Rationale-Guided RAG for Medical QA (NAACL 2025)

**Citation:** Sohn et al. (2025). "Rationale-Guided Retrieval Augmented Generation for Medical Question Answering." NAACL 2025 long paper.

**Problem:** Standard RAG retrievers are biased to training corpus and retrieve imprecise documents for complex medical queries. Missing relevant information when query lacks precise technical terms.

**Architecture (RAG²):**
- LLM generates rationale (explanation) for the query → rationale used as enhanced retrieval query
- Perplexity-based labels identify useful vs. distracting passages
- Corpus-balancing: retrieves evenly from 4 biomedical corpora to reduce source bias

**Results:**
- Up to +6.1% over SOTA LLMs
- Up to +5.6% over prior best medical RAG

**Clinical Relevance:** Rationale-guided retrieval is powerful for complex queries like "What is the first-line treatment for Grade 2 hypertension with CKD Stage 3 and T2DM?" — the LLM generates the clinical reasoning context (rationale) before retrieving, which acts as semantic disambiguation.

---

## Paper 12: Patho-AgenticRAG (AAAI 2025 Workshop)

**Citation:** Authors. "Patho-AgenticRAG: Towards Multimodal Agentic Retrieval-Augmented Generation for Pathology." AAAI 2025. arXiv:2508.02258.

**Problem:** Pathology VLMs hallucinate because visual RAG knowledge bases miss diagnostic visual cues.

**Architecture:**
- Knowledge base: 600+ pathology textbooks (~200K pages), embedded via ColQwen2 (multimodal)
- Agentic Router: decomposes queries into subtasks, plans retrieval strategies (RL-trained via GRPO)
- VRAG Agent: multi-turn retrieval, ranking, evidence summarization
- Tissue-aware partitioning: 19 anatomical tissue classifiers restrict search scope
- Fusion: text + image similarity combined via statistical measures (std, kurtosis, max)

**Results:**
- Quilt-VQA: +13.37% vs Patho-R1 (75.80 vs 64.72%)
- MedXpertQA: +38.00% (60.00 vs 22.00%)
- OmniMedVQA: +19.32% (90.11 vs 70.79%)

**Key Insight for RIPS:** The RL-trained agentic router that dynamically decides when to invoke RAG vs. direct reasoning is directly applicable to clinical decision support. Not all patient queries need full guideline retrieval; some have obvious answers.

---

## Paper 13: TrumorGPT — Graph-based RAG for Fact-Checking (IEEE Trans. AI 2025)

**Citation:** Authors. "TrumorGPT: Graph-Based Retrieval-Augmented Large Language Model for Fact-Checking." IEEE Transactions on Artificial Intelligence, 2025.

**Problem:** Health misinformation spreads rapidly on social media. LLMs alone cannot verify medical claims reliably.

**Architecture:**
- Medical KG: entity-relationship triples (head entity, relationship, tail entity)
- Topic-enhanced TextRank for biomedical concept extraction
- Jaccard-based scoring between query graph and reference graphs
- GraphRAG retrieval of matching knowledge subgraphs

**Results:** 88.5% accuracy, 91.4% precision, 85.0% recall on 600 PolitiFact health claims. Outperforms GPT-4 (83.3%).

**Clinical Relevance:** Fact-checking mechanism could validate LLM-generated clinical recommendations against authoritative guideline knowledge.

---

## Paper 14: LogicRAG — Adaptive Reasoning Without Pre-built Graphs (2025)

**Citation:** Authors. "You Don't Need Pre-built Graphs for RAG: Retrieval Augmented Generation with Adaptive Reasoning." arXiv:2508.06105.

**Problem:** Pre-built KG construction is expensive, slow to update, and introduces offline processing overhead.

**Core Contribution:** Builds query-specific DAGs at inference time rather than offline.
- Query → DAG where nodes = subproblems, edges = logical dependencies
- DFS topological linearization → ordered resolution
- Context pruning (summarize) + graph pruning (merge semantically similar subproblems)

**Results:**
- 2WikiMQA: 64.7% (vs HippoRAG2 50.0%) = +14.7pp
- MuSiQue: 30.4% (vs 27.0%) = +3.4pp

**Clinical Relevance for RIPS:** Offers an alternative to static guideline KGs: build reasoning structures on-demand from raw guideline text. Trades pre-processing cost for inference-time computation.

---

## Paper 15: Two-Layer RAG for Low-Resource Medical QA (peer-reviewed 2024)

**Citation:** Authors. "Two-Layer Retrieval-Augmented Generation Framework for Low-Resource Medical Question-Answering." Peer-reviewed 2024.

**Problem:** Medical RAG on resource-constrained systems with small LLMs.

**Architecture:**
- Layer 1: Retrieve documents → segment → generate per-segment summaries
- Layer 2: Synthesize summaries → final answer
- Uses 7B quantized LLMs (NousHermes2 7B DPO)

**Results:** Comparable to GPT-4 on coherence (5/5 vs 4/5), zero hallucination for both models. Enables use of small open-source LLMs in clinical settings.

**Clinical Relevance:** Two-layer summarization is useful when guidelines are very long and exceed single-pass context. The RIPS project may need hierarchical summarization for multi-page guideline chapters.

---

## Cross-Paper Summary

| Paper | Primary Contribution | Evaluation | Venue | Year |
|-------|---------------------|------------|-------|------|
| Guideline2Graph | CPG PDF → executable graph | NCCN Prostate | IJCAI | 2025 |
| Hyperbolic HKG | 5-stage hyperbolic pipeline | Conceptual | ICML | 2025 |
| MedRAG/MIRAGE | Benchmark (7,663 QA) | 5 datasets | ACL | 2024 |
| AMG-RAG | Agentic KG + CoT RAG | MEDQA, MEDMCQA | EMNLP | 2025 |
| MED-COPILOT | GraphRAG + Patient Retrieval | ICU notes, MMLU | preprint | 2025 |
| PathRAG | Flow-based path pruning | 6 datasets | preprint | 2025 |
| FHIR-RAG-MEDS | FHIR integration with RAG | Clinical CDSS | preprint | 2025 |
| NICE RAG | 300 guidelines, hybrid retrieval | 70 QA pairs | preprint | 2025 |
| Self-MedRAG | Self-reflective hybrid RAG | MedQA, PubMedQA | preprint | 2025 |
| CPGPrompt | CPG → decision tree → LLM execution | 3 domains | JAMIA | 2026 |
| RAG² | Rationale-guided retrieval | Medical QA | NAACL | 2025 |
| Patho-AgenticRAG | Multimodal agentic RAG | 3 pathology QA | AAAI | 2025 |
| TrumorGPT | KG GraphRAG fact-checking | 600 claims | IEEE Trans AI | 2025 |
| LogicRAG | Inference-time DAG construction | 2WikiMQA, MuSiQue | peer-reviewed | 2025 |
| Two-Layer RAG | Hierarchical summarization RAG | Low-resource | peer-reviewed | 2024 |
