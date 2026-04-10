//Reviewer 1: WHbG//

We thank the reviewer for insightful suggestions.

**1.Benchmark novelty.** To the best of our knowledge, we are the **first to incorporate urban spatial graphs as a first-class input modality** for evaluating multimodal embedding models.  
- we introduce a **new evaluation setting**, where models must jointly align **visual, textual, and structured spatial information**, making standard tasks **unexplored under graph-augmented inputs**.  
- Our goal is not new tasks, but to test whether embeddings can **capture structured urban spatial knowledge**, via:  
  - **(i) spatial graphs**, and  
  - **(ii) designed spatial signals (SRPs, SCCs)**, two of which are also data-level contribution.

**2. Failure cases.** We note that the variance between different models/LLMs appear to be stochastic rather than systematic effects. we also undertake an error case analysis on **Nearest POI task** in Beijing. In one example, the model predicts a POI along a nearby main road, while the ground truth is slightly closer on a parallel small pedestrian path. Although both are spatially close, the model favors the POI with stronger road connectivity, suggesting reliance on **coarse structural proximity cues**, which makes **fine-grained distance distinctions** more challenging.

**3. CityLens.** We will cite CityLens and clarify this distinction in the revision.

**4. Data leakage.** We did the followings to avoid data leakage:
- **Spatial separation:**  Anchor locations in train/test sets are **spatially separated**, so their local subgraphs (≤1000 nodes) have minimal or no overlap.
- **Mitigating overlap:**  While entity names may recur, the **spatial configurations, subgraphs, and SRP/SCC descriptions** make locations different, reducing near-duplicate leakage. We will clarify these details in the appendix.

**5. PIGEON/Geo-R.** We additionally evaluate on the IM2GPS3K benchmark by converting coordinate predictions into our ranking setting.
- **Protocol:** We convert GPS coordinates into textual labels (Mapbox), and create 20 candidates per query. 
- **Results:**UGE significantly improves over its backbone (Qwen2.5-VL-7B):
  - **Ranking:** Hit@5 (0.354 → 0.599), NDCG@5 (0.221 → 0.428)
  - **Geolocation accuracy:** Acc@1km (0.132 → 0.350), Acc@25km (0.294 → 0.566), Acc@200km (0.507 → 0.763), Acc@2500km (0.616 → 0.837)
  - **Error reduction:** mean (3467.9 km → 1595.3 km)
- **Takeaway:** The consistent improvements further suggest the usefulness of structured spatial data



//Reviewer 2: q92V//

**1.small candidate pool.** We agree that for tasks such as geolocation and image retrieval, a 20-candidate setting is limited. Following reviewer's suggestion, we extend the **Singapore geolocation test** with larger candidate pools (100 and 150) and evaluate UGE (Qwen2.5-VL-7B).
- **Results:**

| Candidates | Model     | Hit@5 | NDCG@5 |
|------------|-----------|-------|--------|
| 100        | UGE       | **43.38** | **31.22** |
|            | Baseline  | 27.15 | 19.97 |
| 150        | UGE       | **40.08** | **28.07** |
|            | Baseline  | 21.62 | 15.48 |

- **Takeaway:**  
  While performance decreases as candidate size increases , **UGE consistently outperforms its backbone**, indicating that the gains persist in more realistic, larger search spaces.


**2. Zero-shot vs. transfer learning.**  
We use *zero-shot* to indicate that **evaluation tasks are not explicitly seen during training**, as the model is trained with a general **contrastive objective** and evaluated on diverse downstream tasks.  We acknowledge that shared instruction structures make this closer to **transfer learning**, while still involving **cross-city generalization**. We are open to adjust the terminology in the final version.

//Reviewer 3: rLr4//

1.**Efficiency concern.** We agree that inference latency is important.However, we stress that the scope of the work is to validate if **urban spatial graphs can enhance multi-modal embeddings**. Accordingly, the proposed UGE model is designed with a focus on **representation quality** (alignment of visual, textual, and graph modalities), rather than being optimized for **real-time deployment**. That being said, if needed, we can apply more **aggressive or adaptive subgraph sampling**, employ **offline graph precomputation** with only retrieval + fusion at inference, and **graph caching** for overlapping neighbourhoods. We will discuss these optimizations and trade-offs in the Appendix.

**2. Limited statistical robustness analysis.** Due to time constraints, we conduct multi-seed runs on **UGE (Qwen2.5-VL-7B)** by fine-tuning **Stage 2 from Stage 1 checkpoints** using **five training seeds** (42, 123, 3407, 2025, 7), evaluated on the **Singapore geolocation task**.
Across 5 seeds, UGE achieves: **Hit@5: 0.6333 ± 0.0142**
**Per-seed results:**
| Seed | Hit@5 |
|------|------|
| 42   | 64.23(reported in paper) |
| 123  | 63.20 |
| 3407 | 62.10 |
| 2025 | 61.60 |
| 7    | 65.50 |

These results indicate **stable performance across random initializations**.

**3. Small test sets.**  Our benchmark covers **4 core tasks**, with two tasks expanded into **6 and 4 subtasks across 4 cities**, emphasizing **diversity and structured evaluation**. The test sets are **carefully curated** to ensure high quality and include **challenging samples**, rather than simply increasing scale. We observe similar patterns in prior work. For example, **CityEval** contains 100–300 samples per task per city, reflecting a similar trade-off between task diversity and per-task scale.

**4. Error case.**  For a distance query, the model predicts a POI near the same road as the reference location, while the ground truth is slightly closer. This suggests reliance on **proximity cues**, making **fine-grained distance comparison** difficult.
- **Analysis:**  
  (1) **Weak metric grounding:** SRPs express distance/direction in language but are not explicitly aligned with graph-level metrics.  
  (2) **Objective limitation:** Contrastive learning captures semantic relations but struggles with **continuous distance values**.
- **Future work:**  
  (1) **Metric-aware supervision** (e.g., distance regression/bins),  
  (2) **Stronger edge encoding** (distance + direction aligned with SRPs).

  
//Reviewer 4: udFw//

We thank the reviewer for the suggestions. 

**1. Generation.** 
- **Scope:**  Our work focuses on **multimodal embeddings** and **representation learning**, evaluated via **retrieval and ranking** tasks for aligning visual, textual, and spatial graph information.
- **Future direction (generation):**
  We view generation as a complementary extension, using UGE as a **spatially grounded embedding module** within a **RAG** pipeline:
  (1) retrieve relevant urban locations or neighborhoods using UGE embeddings, and
  (2) condition a generative model to produce **spatial QA or navigation instructions** grounded in retrieved evidence.

**2.Efficiency concern.**  
- **Scope:** This work focuses on **spatially grounded multimodal embeddings**, prioritizing **representation quality** (alignment of visual, textual, and graph information) over efficiency.
- **Current design:** We apply **subgraph sampling** (≤1000 nodes), which can be further improved via **adaptive sampling**.
- **Future optimization:**  
  (1) **Offline graph precomputation** (retrieval + fusion at inference),  
  (2) **Graph caching** for overlapping neighborhoods.
We will discuss these optimizations and trade-offs in the Appendix.

**3. Joint training baseline.** 

Following this comment, we added a **joint training baseline (“Joint”)**, where Stage 1 (SRP/SCC) and Stage 2 (graph-conditioned) data are naively combined and trained simultaneously.
**Ablation results (H@5 / N@5):**
| Task                | Method        | NY | SG | BJ |
|---------------------|--------------|----|----|----|
| Geolocation Ranking | **UGE**      | **58.85/47.94** | **64.23/54.65** | **71.92/63.38** |
|                     | Stage1-only  | 50.38/34.76 | 55.77/43.70 | 53.08/42.26 |
|                     | Joint        | 39.84/25.94 | 38.25/29.87 | 15.96/22.60 |
|                     | Stage2-only  | 29.23/18.02 | 25.77/17.23 | 2.31/1.42   |
| Image Retrieval     | **UGE**      | **58.86/43.24** | **71.10/52.22** | 57.43/40.44 |
|                     | Stage1-only  | 58.71/42.07 | 69.95/53.14 | **60.14/43.00** |
|                     | Joint        | 32.86/28.65 | 48.78/34.77 | 30.92/30.56 |
|                     | Stage2-only  | 27.00/15.22 | 27.61/16.40 | 27.71/17.11 |

As shown above, Joint training underperforms Stage1-only and the full two-stage UGE, indicating gains are **not due to data volume** but the **progressive training strategy**. We will clarify this in the revision.

