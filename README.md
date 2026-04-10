//Reviewer 1: WHbG//

We thank the reviewer for insightful suggestions.

**1.Benchmark novelty.** Our benchmark is, to the best of our knowledge, the **first to incorporate urban spatial graphs as a first-class input modality** for evaluating multimodal embedding models.  
- This introduces a **new evaluation setting**, where models must jointly align **visual, textual, and structured spatial information**, making standard tasks **unexplored under graph-augmented inputs**.  
- The goal is not new tasks, but to test whether embeddings can **capture structured urban spatial knowledge**, via:  
  - **(i) spatial graphs**, and  
  - **(ii) designed spatial signals (SRPs, SCCs)**.  
- We also want to emphasize the **data-level contribution**:  
  - **Stage 1 (SRPs/SCCs):** injects spatial awareness via language-aligned reasoning cues.  
  - **Stage 2 (graph conditioning):** introduces explicit **topological and relational structure**.  

This design enables **progressive injection of spatial knowledge** into the embedding space.

**2. No analysis for the failure cases.** We note that this behavior appears to be **model-specific variance**. Following the suggestion, we also undertake an error case analysis on Nearest POI task in Beijing.In one example, the model predicts a POI along a nearby main road, while the ground truth is slightly closer on a parallel pedestrian path. Although both are spatially close, the model favors the POI with stronger road connectivity, suggesting reliance on **coarse structural proximity cues**, which makes **fine-grained distance distinctions** more challenging.

**3. CityLens.** CityLens focuses on urban socio-economic prediction from visual data, while our work targets **representation learning and evaluation** for aligning visual, textual, and spatial graph information. We will cite CityLens and clarify this distinction in the revision.

**4. Data leakage.** We take several steps to avoid leakage between training (SRP/SCC) and evaluation:
- **Spatial separation:**  Anchor locations in train/test sets are **spatially separated**, so their local subgraphs (≤1000 nodes) have minimal or no overlap.
- **Independent candidate pools:**  When local candidates are insufficient, additional candidates are sampled **only from the test set**, ensuring no training data is used.
- **Mitigating overlap:**  While entity names may recur, the **spatial configurations, subgraphs, and SRP/SCC descriptions** are distinct across splits, reducing near-duplicate leakage.
We will clarify this process in the appendix.

**5. PIGEON/Geo-R.** We additionally evaluate on the IM2GPS3K benchmark (used by PIGEON/Geo-R) by converting coordinate predictions into our ranking setting.
- **Protocol:** We convert GPS coordinates into textual labels via reverse geocoding (Mapbox), and construct 20-candidate sets per query. This allows evaluation using both:
  - ranking metrics (Hit@K, NDCG@K), and
  - standard distance-based metrics (Acc@1km, 25km, 200km, 750km, 2500km).
- **Results:**
  UGE significantly improves over its backbone (Qwen2.5-VL-7B):
  - **Ranking:** Hit@5 (0.354 → 0.599), NDCG@5 (0.221 → 0.428)
  - **Geolocation accuracy:** Acc@1km (0.132 → 0.350), Acc@25km (0.294 → 0.566), Acc@200km (0.507 → 0.763), Acc@2500km (0.616 → 0.837)
  - **Error reduction:** mean (3467.9 km → 1595.3 km)
- **Takeaway:** The improvements are consistent across both **ranking quality** and **coordinate-level accuracy**, aligning with standard evaluation protocols used by coordinate-prediction methods.



//Reviewer 2: q92V//

**1.small candidate pool.** We agree that for tasks such as geolocation and image retrieval, a 20-candidate setting is limited. Following reviewer's suggestion. We extend the **Singapore geolocation test** with larger candidate pools (100 and 150) and evaluate UGE (Qwen2.5-VL-7B).
- **Results:**

| Candidates | Model     | Hit@5 | NDCG@5 |
|------------|-----------|-------|--------|
| 100        | UGE       | **43.38** | **31.22** |
|            | Baseline  | 27.15 | 19.97 |
| 150        | UGE       | **40.08** | **28.07** |
|            | Baseline  | 21.62 | 15.48 |
- **Takeaway:**  
  While performance decreases as candidate size increases (expected), **UGE consistently outperforms its backbone**, indicating that the gains persist in more realistic, larger search spaces.


**2. Zero-shot vs. transfer learning.**  
We use *zero-shot* to indicate that **evaluation tasks are not explicitly seen during training**, as the model is trained with a general **contrastive objective** and evaluated on distinct downstream tasks.  We acknowledge that shared instruction structures make this closer to **format-consistent transfer learning**, while still involving **cross-city generalization**. We will revise the terminology accordingly.

//Reviewer 3: rLr4//

1.**Efficiency concern.** 
We agree that inference latency is important.
- **Scope:** This work focuses on **spatially grounded multimodal embeddings**, prioritizing **representation quality** (alignment of visual, textual, and graph information) over efficiency.
- **Current design:** We apply **subgraph sampling** (≤1000 nodes), which can be further improved via **adaptive sampling**.
- **Future optimization:**  
  (1) **Offline graph precomputation** (retrieval + fusion at inference),  
  (2) **Graph caching** for overlapping neighborhoods.
We will discuss these optimizations and trade-offs in the Appendix.

**2. Limited statistical robustness analysis.** 
Due to time constraints, we conduct multi-seed analysis on **UGE (Qwen2.5-VL-7B)** by fine-tuning **Stage 2 from Stage 1 checkpoints** using **five training seeds** (42, 123, 3407, 2025, 7), evaluated on the **Singapore geolocation task**.
Across 5 seeds, UGE achieves: **Hit@5: 0.6314 ± 0.0210**
**Per-seed results:**

| Seed | Hit@5 |
|------|------|
| 42   | 60.30 |
| 123  | 63.20 |
| 3407 | 62.10 |
| 2025 | 64.60 |
| 7    | 65.50 |

These results indicate **stable performance across different random initializations**, with relatively low variance compared to the overall performance gains.

**3. Small test sets.**  
Our benchmark covers **4 core tasks**, with perception and grounding expanded into **6 and 4 subtasks across 4 cities**, emphasizing **diversity and structured evaluation**. The test sets are **carefully curated** to ensure high quality and include **challenging samples**, rather than simply increasing scale.  
We observe similar designs in prior work; for example, CityEval (CityGPT) has ~6,000 samples across 41 tasks, i.e., **~100–200 instances per task**.

**4. Error case.**  
We include **error analysis** for distance-based queries.
- **Example:**  For a distance query, the model predicts a POI near the same road as the reference location, while the ground truth is slightly closer. This suggests reliance on **local proximity cues**, making **fine-grained distance comparison** difficult.
- **Analysis:**  
  (1) **Weak metric grounding:** SRPs express distance/direction in language but are not explicitly aligned with graph-level metrics.  
  (2) **Objective limitation:** Contrastive learning captures semantic relations but struggles with **continuous distance values**.
- **Future work:**  
  (1) **Metric-aware supervision** (e.g., distance regression/bins),  
  (2) **Stronger edge encoding** (distance + direction aligned with SRPs).

  
//Reviewer 4: udFw//

**1. Generation.** We thank the reviewer for this suggestion.

- **Scope:**  Our work focuses on **multimodal embeddings** and **representation learning**, evaluated via **retrieval and ranking** tasks for aligning visual, textual, and spatial graph information.
- **Future direction (generation):**
  We view generation as a complementary extension, using UGE as a **spatially grounded embedding module** within a **SpatialRAG** pipeline:
  (1) retrieve relevant urban locations or neighborhoods using UGE embeddings, and
  (2) condition a generative model to produce **spatial QA or navigation instructions** grounded in retrieved evidence.
This setup can enhance **spatial faithfulness** and support real-world applications (e.g., travel assistance).

**2.Efficiency concern.** 
We agree that inference latency is important.
- **Scope:** This work focuses on **spatially grounded multimodal embeddings**, prioritizing **representation quality** (alignment of visual, textual, and graph information) over efficiency.
- **Current design:** We apply **subgraph sampling** (≤1000 nodes), which can be further improved via **adaptive sampling**.
- **Future optimization:**  
  (1) **Offline graph precomputation** (retrieval + fusion at inference),  
  (2) **Graph caching** for overlapping neighborhoods.
We will discuss these optimizations and trade-offs in the Appendix.

**3.joint training baseline.**
We thank the reviewer for this important suggestion. Following this comment, we added a **joint training baseline (“Joint”)**, where Stage 1 (SRP/SCC) and Stage 2 (graph-conditioned) data are naively combined and trained simultaneously.
**Ablation results (H@5 / N@5):**
| Task                | Method        | NY | SG | BJ | PA |
|---------------------|--------------|----|----|----|----|
| Geolocation Ranking | **UGE**      | **58.85/47.94** | **64.23/54.65** | **71.92/63.38** | **65.77/52.88** |
|                     | Stage1-only  | 50.38/34.76 | 55.77/43.70 | 53.08/42.26 | 53.85/39.66 |
|                     | Joint        | 39.84/25.94 | 38.25/29.87 | 15.96/22.60 | 10.54/20.08 |
|                     | Stage2-only  | 29.23/18.02 | 25.77/17.23 | 2.31/1.42   | 1.00/0.50   |
| Image Retrieval     | **UGE**      | **58.86/43.24** | **71.10/52.22** | 57.43/40.44 | **57.22/41.50** |
|                     | Stage1-only  | 58.71/42.07 | 69.95/53.14 | **60.14/43.00** | 56.43/42.16 |
|                     | Joint        | 32.86/28.65 | 48.78/34.77 | 30.92/30.56 | 20.95/28.94 |
|                     | Stage2-only  | 27.00/15.22 | 27.61/16.40 | 27.71/17.11 | 25.46/15.72 |

As shown above, Joint training consistently underperforms Stage1-only, and is substantially worse than the full two-stage UGE model (reported in the main paper). This indicates that the gains are **not simply due to increased data volume or diversity**. Instead, the results support the importance of the **progressive training strategy**
We will clarify this comparison and emphasize the role of training curriculum more explicitly in the revision.

