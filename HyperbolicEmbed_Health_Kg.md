# Position: Hyperbolic Embeddings Are Essential for Health Knowledge Graphs in LLMs and Vector Databases

**Authors: Anonymous (under review) Venue: ICML 2025 (preliminary, under review) **

[https://openreview.net/pdf?id=Sz90WdONPz](https://openreview.net/pdf?id=Sz90WdONPz) 


# Main Part


## Overview

Current biomedical retrieval systems overwhelmingly rely on **Euclidean or spherical embeddings, **which cannot efficiently capture the deep hierarchical structures (e.g., multi-level ICD or SNOMED taxonomies) **inherent in health knowledge graphs (HKGs)**. This position paper argues that **hyperbolic embeddings **— operating in negatively curved spaces such as the Poincaré ball — should become the standard for encoding and retrieving HKGs within **LLM-driven pipelines**. 

The key theoretical advantage is that hyperbolic volume expands exponentially with radius, mirroring how tree nodes proliferate at each ontology level, allowing hierarchical information to be compressed with minimal distortion in far fewer dimensions. The authors propose a **five-stage Hyperbolic HKG Pipeline **and cite empirical evidence from prior works showing 10–20% gains in Hits@k retrieval metrics when switching from Euclidean to hyperbolic distance metrics.


## The Pipeline

The paper's central contribution is the **Hyperbolic HKG Pipeline** — a five-stage end-to-end framework for encoding and querying health knowledge graphs with negative-curvature geometry.


### Step-by-step examples:


#### Stage 1 — Ontology Ingestion & Preprocessing

**Goal: **Gather all heterogeneous medical data sources and normalize them into a single unified graph schema.

**Steps:**



1. **Collect source ontologies** — pull from **SNOMED CT, ICD-10/11, UMLS, drug ontologies, gene-phenotype databases, and de-identified patient record metadata.**
2. **Entity normalization** — standardize concept identifiers across sources (e.g., "Type 2 Diabetes" in** ICD = E11, in SNOMED = 44054006**). Resolve naming conflicts and aliases.
3. **Edge typing** — label each edge with its relationship type:
    * `is-a` (taxonomic parent-child)
    * `associated-with` (co-occurrence, causal)
    * `treats`/`contraindicates` (drug-disease)
    * `complicates` (e.g., Hypertension → CKD)
4. **Unified schema output** — produce a single heterogeneous graph `G₀ = (V₀, E₀)` with typed nodes and edges, ready for embedding.

**Input:** Raw ontology files (OWL, RDF), clinical databases **Output:** Normalized graph `G₀` with typed nodes and edges 

**Example (RIPS context):**

Node: "Hypertension" [ICD: I10, SNOMED: 38341003]

  is-a → "Cardiovascular disease"

  complicates → "Chronic Kidney Disease" [ICD: N18]

  complicates → "Hypertensive CKD" [ICD: I12]

  treated-by → "Amlodipine", "Ramipril", "Hydrochlorothiazide"


#### Stage 2 — Hyperbolic Embedding Training

**Goal:** Learn vector representations of every node in `G₀` inside a Poincaré ball, such that hierarchical structure is preserved with minimal distortion.

**Steps:**



1. **Initialize embeddings** — place all nodes near the center of the unit Poincaré ball `𝔻ᵈ = {x ∈ ℝᵈ : ‖x‖ &lt; 1}`. Random initialization with small norm works well.
2. **Set curvature parameter <code>c &lt; 0</code>** — this controls how "curved" the space is:
    * Deep ontologies (6+ levels, e.g., SNOMED) → higher `|c|` → stronger curvature → more separation between levels
    * Shallow or cross-linked subgraphs (e.g., drug-adverse-event networks) → lower `|c|`
    * Best practice: **learn <code>c</code> jointly** during training (treat it as a trainable parameter)
3. **Define training objective** — typically contrastive or ranking loss over positive (connected) and negative (unconnected) node pairs, using Poincaré distance `d(u,v)` as the similarity measure.
4. **Riemannian gradient descent** — standard gradient descent does not work in curved space. Instead:
    * Compute Euclidean gradient as usual
    * **Project** it onto the tangent space at the current point (Riemannian gradient)
    * **Retract** the update back onto the manifold (stay inside the ball)
    * This is the algorithm from Bonnabel (2013), implemented in the `geoopt` Python library
5. **Output embeddings** — each node `v` gets a position `φ(v) ∈ 𝔻ᵈ`. Parent nodes near center; leaf/specific nodes near boundary.

**Input:** Graph `G₀`, curvature `c`, dimension `d` (typically d=5–32 for hyperbolic vs. d=128–768 for Euclidean)

**Output:** Embedding table `Φ : V₀ → 𝔻ᵈ`

**Concrete illustration:**

After training on the SNOMED hierarchy:

"Disease" (root)           → φ = [0.01, 0.02, ...]   ‖φ‖ ≈ 0.05  (near center)

"Cardiovascular disease"   → φ = [0.3, 0.1, ...]    ‖φ‖ ≈ 0.35

"Hypertension"             → φ = [0.6, 0.2, ...]    ‖φ‖ ≈ 0.65

"Hypertensive CKD"         → φ = [0.85, 0.1, ...]   ‖φ‖ ≈ 0.87  (near boundary)

"Hypertensive CKD stage 5" → φ = [0.94, 0.05, ...]  ‖φ‖ ≈ 0.94

The key property: `d("Hypertension", "Hypertensive CKD stage 5")` is large (they are far apart in hierarchy), while `d("Hypertension", "Cardiovascular disease")` is moderate (nearby in tree). Euclidean embeddings would need ~10× more dimensions to achieve the same separation without crowding.


#### Stage 3 — Vector Database Integration

**Goal:** Store the hyperbolic embeddings in a queryable index that supports fast approximate nearest-neighbor (ANN) search under Poincaré distance.

**Steps:**



1. **Choose indexing strategy** — two main options:
    * **Manifold-aware indexing:** Build Voronoi cells or geodesic partitions directly in hyperbolic space. Exact but computationally complex.
    * **Euclidean approximation:** Apply a differentiable mapping (e.g., exponential map at the origin) to project Poincaré embeddings into a tangent Euclidean space, then use standard FAISS/HNSW. Faster to implement but introduces approximation error.
2. **Index construction** — store all node embeddings `{φ(v) : v ∈ V₀}` in the chosen index structure.
3. **Dynamic update support** — when new disease codes are added (e.g., a new ICD revision), embed the new node using the trained model and insert into the index without full retraining.
4. **Query interface** — expose a `search(query_embedding, k)` function that returns top-k nodes by Poincaré distance.

**Input:** Embedding table `Φ`, index type choice **Output:** Queryable ANN index `I`

**Practical note: **The `geoopt` library (PyTorch-compatible, CUDA-ready) handles Riemannian optimization. For the ANN layer, `hnswlib` with a custom distance function is the most accessible path — wrapping Poincaré distance as a custom metric.


#### Stage 4 — LLM-Driven Query

**Goal:** Allow an LLM (e.g., the CDSS inference engine) to retrieve the most relevant knowledge graph nodes for a given clinical query.

**Steps:**



1. **Query encoding** — encode the clinical query (e.g., "treatment threshold for Grade 2 hypertension with CKD") using a medical LLM encoder (BioBERT, PubMedBERT, or the embedding head of the LLM).
2. **Project to hyperbolic space** — map the query embedding into the Poincaré ball using the exponential map `expₒ(x) = tanh(‖x‖) * x/‖x‖` from the origin.
3. **Top-k retrieval** — query the ANN index `I` using Poincaré distance to retrieve top-k relevant nodes.
4. **Re-ranking** — optionally re-rank retrieved nodes using the LLM's own attention scores or cross-encoder over the retrieved context.
5. **Prompt assembly** — inject retrieved node content (recommendation text, evidence grades, source references) into the LLM prompt as RAG context.

**Input:** Natural language query, index `I`, LLM encoder **Output:** Top-k retrieved knowledge chunks, assembled prompt

**Example retrieval:**

Query: "Patient with BP 165/95, eGFR 45 — medication options?"

Hyperbolic retrieval finds:

  1. "Grade 2 Hypertension (≥160/100)" [distance 0.12]

  2. "CKD Stage 3a (eGFR 45–59)" [distance 0.18]

  3. "Hypertensive CKD — preferred agents: ACE inhibitor/ARB" [distance 0.21]

  4. "Avoid NSAIDs in CKD" [distance 0.29]

→ These are assembled into RAG prompt → LLM generates recommendation

The hyperbolic geometry ensures that "CKD" and "Hypertension" nodes (in different subtrees) are still retrieved together because their cross-link edge was preserved during training — something Euclidean embeddings would scatter in high-dim space.


#### Stage 5 — Interpretation & Visualization

**Goal:** Present the knowledge graph structure and retrieval results to clinicians in an interpretable, navigable interface.

**Steps:**



1. **Radial layout** — render the Poincaré disk with top-level concepts at center, leaf concepts at periphery. Clinicians can zoom in to a subtree (e.g., all diabetes subtypes) without losing context of where it sits in the full ontology.
2. **Subtree highlighting** — given a query, highlight the retrieved nodes and their ancestral paths back to the root, showing *why* a concept was retrieved.
3. **Small-world shortcut display** — render cross-branch edges (e.g., hypertension ↔ CKD) as arcs, making co-morbidity relationships visible.
4. **Confidence intervals** — show uncertainty around boundary-region embeddings (where rare diseases cluster) to flag low-confidence retrievals.
5. **Clinician feedback loop** — allow clinicians to flag incorrect retrievals, triggering targeted re-embedding or edge correction.

**Input:** Poincaré embeddings, query results, clinician interface **Output:** Interactive hyperbolic disk visualization


### Step-by-Step Worked Example (End-to-End)

**Clinical scenario:** A polyclinic doctor sees a patient with BP 165/95 mmHg, eGFR 48, and HbA1c 7.8%. She queries the CDSS: *"What medications should I initiate?"*


<table>
  <tr>
   <td><strong>Step</strong>
   </td>
   <td><strong>What happens</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Stage 1 output</strong>
   </td>
   <td>Graph has nodes for Hypertension (I10), CKD Stage 3a (N18.3), T2DM (E11), plus edges: HTN→complicates→CKD, DM→complicates→CKD, ACE-inhibitor→treats→HTN+CKD
   </td>
  </tr>
  <tr>
   <td><strong>Stage 2 output</strong>
   </td>
   <td>"HTN" near Poincaré boundary at ‖φ‖≈0.65; "CKD" at ‖φ‖≈0.70; "HTN+CKD" combined node at ‖φ‖≈0.82; "ACE-inhibitor for HTN+CKD" at ‖φ‖≈0.91
   </td>
  </tr>
  <tr>
   <td><strong>Stage 3</strong>
   </td>
   <td>All embeddings indexed; cross-branch HTN↔CKD edge preserved in geodesic distance
   </td>
  </tr>
  <tr>
   <td><strong>Stage 4</strong>
   </td>
   <td>Query encoded → top-k retrieval returns: Grade 2 HTN node, CKD 3a node, "Preferred antihypertensive in CKD = ACEi/ARB" node, "Avoid NSAIDs" node
   </td>
  </tr>
  <tr>
   <td><strong>Stage 5</strong>
   </td>
   <td>Clinician sees Poincaré disk with HTN and CKD subtrees highlighted, arc showing their complication link, recommendation nodes lit up near boundary
   </td>
  </tr>
  <tr>
   <td><strong>LLM output</strong>
   </td>
   <td>"For Grade 2 hypertension with CKD Stage 3a: initiate Ramipril 5mg OD (ACE inhibitor preferred per MOH ACE Guideline p.42). Avoid NSAIDs. Target BP &lt;130/80."
   </td>
  </tr>
</table>



---


# Appendix of Exp Details


## Dataset / Benchmark 

Uses SNOMED CT, ICD, UMLS as illustrative ontologies. References benchmark results from Nickel & Kiela (2017), Sala et al. (2018), Chami et al. (2019/2020). 
