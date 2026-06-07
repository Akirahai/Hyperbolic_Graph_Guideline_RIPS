# Guideline2Graph: Profile-Aware Multimodal Parsing for Executable Clinical Decision Graphs 

**Xuejiao Zhao - ICJA 2025**

[https://arxiv.org/pdf/2604.02477](https://arxiv.org/pdf/2604.02477) 


# Main Part


## **Overview**

**The Problem:** Clinical practice guidelines are long, multimodal documents whose branching recommendations are difficult to convert into executable clinical decision support (CDS). **Recent LLM/VLM **extractors are mostly local or text-centric, under-specifying section interfaces and failing to consolidate cross-page control flow across full documents into one coherent decision graph.

**The Core Insight — "Decomposition-First":** Rather than prompting a VLM to generate a decision graph in one shot (which fails at long documents), they break the problem into **three targeted sub-problems: **chunking → per-chunk graph generation → global merge. 



* VLMs are deliberately orchestrated to act as named entity recognizers (NER), boundary detectors, classifiers for structured output logic, and semantic rerankers, rather than being asked to do** one-shot graph generation.**

**The Output:** A directed labeled graph G = (V, E) where each node v is a clinical state/decision/recommendation, and each edge (u, ℓ, v) is a transition from u to v under condition ℓ (e.g., "PSA > 10 ng/mL").


## **The Pipeline**


### **Step-by-step examples:**


##### Stage 1 — Topology-Aware Chunk Generation (Algorithm 1)

**Goal:** Split the full CPG PDF into coherent decision segments, discarding non-decision pages.

**Steps:**



1. **Extract guideline profile M** from the first h header pages — captures metadata (title, code) and a compact scope context `ct` describing the intended patient population and clinical focus.
2. **Classify each page in parallel** using a prompted VLM:
    *  `CORE` (contains actionable decision content — algorithms, flowcharts, criteria, recommendations)
    * `AUXILIARY` (references, author lists, administrative text).
3. **Partition CORE pages into maximal consecutive runs.** H — prevents chunks from spanning long auxiliary gaps.
4. **For each run, build chunks incrementally** with a page buffer B and running memory `ctx`:
    * At each page, run `PREDICTBOUNDARY(B, Pᵢ, P⁺, ctx, L)` — the lookahead page P⁺ prevents cutting mid-table or separating a section header from its content.
    * When a boundary is triggered (or the run ends), finalize the buffer into a chunk, producing: description d, **entry node labels R** (roots), **terminal node labels Z** (exits), carry-forward pages K for inter-chunk continuity, and updated memory ctx'.
5. **Output:** A sequence of chunk triplets `P = {(Tⱼ, Rⱼ, Zⱼ)}` where Tⱼ is the full assembled context (profile + description + pages + memory).


##### **Stage 2 — Chunk-Level Graph Generation (Algorithm 2)**

**Goal:** For each chunk triplet (Tⱼ, Rⱼ, Zⱼ), build a local decision subgraph Gⱼ = (Vⱼ, Eⱼ) via breadth-first expansion.

**Steps:**



1. Pre-initialize Vⱼ with all terminal nodes Zⱼ — terminals are fixed and cannot be added during expansion.
2. Initialize queue Q with all root nodes Rⱼ.
3. **For each dequeued node candidate u:**
    * Retrieve top-k in-chunk neighbors S by cosine embedding similarity.
    * Run VLM verifier on (u, α, S) to check semantic equivalence — if a duplicate u* is found, redirect incoming edges to u* and skip.
    * Otherwise, register u as a new non-terminal node.
    * Call `GENERATECHILDREN(u, α, Tⱼ)` → outputs successor pairs {(v, e_uv)} where e_uv is the transition condition.
    * Enqueue each (v, (u, e_uv)).
4. Expansion continues until the queue is exhausted.


##### **Stage 3 — Global Aggregation (Algorithm 3)**

**Goal:** Merge all chunk graphs into a single document-level graph, resolving duplicates at chunk interfaces.

**Steps:**



1. Form the global union V = ∪ Vⱼ, E = ∪ Eⱼ, tracking provenance orig(v) = j for each node.
2. Seed queue with all interface nodes (all Rⱼ and Zⱼ across chunks) — these are where cross-chunk duplicates cluster.
3. **For each dequeued node x:**
    * Collect ancestor-edge context A = {(a, e) : a→x}.
    * Restrict duplicate search to cross-chunk candidates C = {y ∈ V : orig(y) ≠ orig(x)}.
    * Retrieve top-k semantic neighbors S from C, then VLM-verify equivalence from (x, A, S).
    * If duplicate x* found: choose primary p (prefer non-terminal, then earlier chunk), rewire all edges incident to x* toward p, and remove x* from graph and queue.
4. Output: globally consolidated graph G = (V, E).


---


# Appendix of Exp Details


## **Dataset Construction**

The system is evaluated on a single prostate clinical practice guideline: the **NCCN Clinical Practice Guidelines in Oncology: Prostate Cancer, Version 4.2024.**

**Ground-truth references for each unit are manually curated and adjudicated by human reviewers.** Six evaluation units are defined: five chunk-level graphs G₁…G₅ and one merged complete graph G_all.

There is **no automatically constructed dataset** (not like our project, I suppose).


## **Experiment 1 - Quantitative Graph Extraction Accuracy**


##### **Setup:** All methods are rerun on the same document inputs and normalized with the same post-processing interface before scoring. To ensure parity, all compared methods use the same **underlying VLM backbone**, so observed differences are attributable to** graph-construction strategy rather than model choice.**

**Metrics:** Precision and recall on **nodes, edges, and triplets (node–edge–node)**. S/T (supported-over-total) counts are reported for traceability.

**Baselines:**



* **Doc2KG** — general document-to-KG pipeline
* **AutoKG** — automated KG generation for LLMs

**Results on the complete merged graph (G_all):**


<table>
  <tr>
   <td><strong>Method</strong>
   </td>
   <td><strong>Node Prec.</strong>
   </td>
   <td><strong>Node Rec.</strong>
   </td>
   <td><strong>Edge Prec.</strong>
   </td>
   <td><strong>Edge Rec.</strong>
   </td>
   <td><strong>Triplet Prec.</strong>
   </td>
   <td><strong>Triplet Rec.</strong>
   </td>
  </tr>
  <tr>
   <td>Doc2KG
   </td>
   <td>27.5%
   </td>
   <td>43.8%
   </td>
   <td>1.1%
   </td>
   <td>1.8%
   </td>
   <td>1.1%
   </td>
   <td>1.8%
   </td>
  </tr>
  <tr>
   <td>AutoKG
   </td>
   <td>56.8%
   </td>
   <td>78.1%
   </td>
   <td>19.6%
   </td>
   <td>16.1%
   </td>
   <td>19.6%
   </td>
   <td>16.1%
   </td>
  </tr>
  <tr>
   <td><strong>Ours</strong>
   </td>
   <td><strong>57.7%</strong>
   </td>
   <td><strong>93.8%</strong>
   </td>
   <td><strong>69.0%</strong>
   </td>
   <td><strong>87.5%</strong>
   </td>
   <td><strong>69.0%</strong>
   </td>
   <td><strong>87.5%</strong>
   </td>
  </tr>
</table>


On the merged complete graph, our method shows clear structural gains over AutoKG: node recall improves by +15.7 points (93.8% vs. 78.1%), edge precision by +49.4 points (69.0% vs. 19.6%), and edge recall by +71.4 points (87.5 vs. 16.1). Triplet precision/recall show the same improvements, indicating better preservation of executable transition structure after global aggregation.

**Key nuance:** One nuance is node recall on G₁ and G₃, where AutoKG reaches 100.0% and the proposed method is lower. However, the structural metrics that determine executable control flow are substantially better — for example, G₃ edges: 100.0%/100.0% precision/recall vs. 28.6%/42.9% for AutoKG. AutoKG over-generates nodes (high recall, low precision), whereas this method is more precise about what constitutes a real decision node.
