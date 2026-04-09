//Reviewer 1: WHbG//

1. **Benchmark is insufficiently novel**. We thank the reviewer for this suggestion. While the individual task formats are conceptually familiar, our benchmark is, to the best of our knowledge, the **first to incorporate urban spatial graphs as a first-class input modality** for evaluating multimodal embedding models.
By treating spatial graphs as an additional modality, the benchmark introduces a **new evaluation setting** where models must jointly align **visual, textual, and structured spatial information**. In this context, even conventional tasks become **previously unexplored** for embedding models with spatial inputs.

The primary aim of this work is to evaluate whether multimodal embeddings can **capture structured urban spatial knowledge**, specifically through:
- (i) **urban spatial graphs**, and
- (ii) **designed spatial signals in SRPs and SCCs**.
Rather than proposing new task types, we **reframe standard tasks** to test whether embeddings can encode **relational and spatial structure beyond visual appearance**.
We want to emphasize the **data-level contribution** of our work. Our two-stage dataset provides **complementary supervision signals**:
- **Stage 1 (SRPs/SCCs):** injects spatial awareness via **language-aligned spatial reasoning paths and contextual descriptions**, grounding images within their surrounding urban context.
- **Stage 2 (graph conditioning):** introduces **explicit spatial structure** through urban graphs, enabling the model to encode **topological and relational information**.
This design allows spatial knowledge to be **progressively aligned and injected** into the embedding space.

2. **No analysis for the failure cases**. We thank the reviewer for this insightful comment. We note that this behavior appears to be **model-specific variance**. 
Following the suggestion, we also undertake an error case analysis on Nearest POI task in Beijing.
In one representative example, the model predicts a POI A located along a nearby main road, while the ground-truth POI B lies slightly closer but along a parallel pedestrian-accessible path. Both candidates are spatially close and share similar local context, but the model favors the POI A more strongly connected to the dominant road structure. This suggests that the model may rely more on **coarse structural proximity cues (e.g., road connectivity)**, which can make **fine-grained distance distinctions between nearby candidates more challenging**.

3. **CityLens**. CityLens focuses on urban socio-economic prediction from visual data, formulated as supervised learning over predefined indicators. In contrast, our work targets representation learning and evaluation, aiming to assess whether multimodal embeddings can align visual, textual, and spatial graph information.
In particular, our benchmark differs by (1) introducing spatial graphs as an input modality, and (2) formulating tasks as retrieval and ranking, rather than prediction. We will cite CityLens and clarify this distinction in the revision.

4. **How the our work avoid leakage between training and evaluation**. We take several steps to avoid leakage between training (SRP/SCC) and evaluation:
- **Spatially separated candidate construction:**
  Candidate sets are constructed based on **spatial proximity to the anchor image**, with candidates sampled within a radius of 100–1000 meters. Importantly, anchor locations in the training and test sets are **spatially separated**, such that their corresponding local subgraphs (up to 1000 nodes) have minimal or no overlap. This reduces the risk of shared entities or graph structures.
- **Test-set-only candidate supplementation:**
  When the required number of candidates (e.g., 20) cannot be satisfied within the local neighborhood, additional candidates are sampled from **other instances within the test set**, ensuring that candidate pools remain independent of training data.
- **Mitigation of name/text overlap:**
  While entity names (e.g., common street or POI names) may naturally recur in urban environments—for example, similar street names such as *“Nan Xiaojie”* and *“Bei Xiaojie”* in areas like Dongzhimen or Xizhimen in Beijing—the **specific spatial configurations, local subgraphs, and SRP/SCC descriptions** associated with each anchor location are distinct across splits. This mitigates near-duplicate or template-level leakage.
We will clarify this data construction process more explicitly in the appendix.

5. **Test UGE on coordinates-based geolocation test set**. We thank the reviewer for this suggestion. To better contextualize geolocation ranking, we additionally evaluate on the IM2GPS3K benchmark (used by PIGEON/Geo-R) by converting coordinate predictions into our ranking setting.
- **Protocol:**
  We convert GPS coordinates into textual labels via reverse geocoding (Mapbox), and construct 20-candidate sets per query. This allows evaluation using both:
  - ranking metrics (Hit@K, NDCG@K), and
  - standard distance-based metrics (Acc@1km, 25km, 200km, 750km, 2500km).
- **Results:**
  UGE significantly improves over its backbone (Qwen2.5-VL-7B):
  - **Ranking:** Hit@5 (0.354 → 0.599), NDCG@5 (0.221 → 0.428)
  - **Geolocation accuracy:** Acc@1km (0.132 → 0.350), Acc@25km (0.294 → 0.566), Acc@200km (0.507 → 0.763), Acc@2500km (0.616 → 0.837)
  - **Error reduction:** mean (3467.9 km → 1595.3 km), median (196.3 km → 8.4 km)
- **Takeaway:**
  The improvements are consistent across both **ranking quality** and **coordinate-level accuracy**, aligning with standard evaluation protocols used by coordinate-prediction methods.



//Reviewer 2: q92V//

1. **small candidate pool**


2. **Zero-shot VS transfer learning**. We would like to clarify the use of the term. We used the term to indicate that the **evaluation tasks themselves are not explicitly seen during training**. The model is trained with a general **contrastive objective** on UGData, while evaluation is conducted on four specific downstream tasks (geolocation ranking, image retrieval, urban perception, spatial grounding) with distinct queries and instructions.
That said, we acknowledge that the instruction templates share structural similarity between training and evaluation, which aligns more closely with **format-consistent transfer learning**. At the same time, our evaluation involves **cross-city generalization**. We are open to revise on this issue.

//Reviewer 3: rLr4//

1. **Efficiency concern**. We thank the reviewer for highlighting the efficiency concern. We agree that inference latency is important for real-world deployment.
- **Scope clarification:**
  The primary focus of this work is to study and test **spatially grounded multimodal embeddings**, i.e., whether incorporating spatial graphs and the designed spatial signals (SRPs, SCCs) helps models better understand **spatial structure** and align **visual, textual, and topological information** across urban tasks (e.g., geolocation ranking, urban perception). As such, the current implementation prioritizes **representational effectiveness** over efficiency.
- **Current design:**
  We already apply **subgraph sampling**, truncating the local graph to at most 1000 nodes based on spatial proximity to the anchor image, in both training and inference. This can be further improved via **adaptive sampling**, e.g., using smaller subgraphs in low-density regions.
- **Optimization strategies (future work):**
  (1) **Offline graph precomputation:** Precompute subgraph embeddings so inference reduces to retrieval + fusion (similar to RAG pipelines).
  (2) **Graph caching:** Reuse embeddings for overlapping spatial neighborhoods across queries.
We will include a discussion of these optimization directions and trade-offs in the Appendix.

2. **Limited statistical robustness analysis**.  We thank the reviewer for this suggestion. Due to time constraints, we conduct multi-seed analysis on **UGE (Qwen2.5-VL-7B)** using **five training seeds** (42, 123, 3407, 2025, 7), evaluated on the **Singapore geolocation task**.

Across 5 seeds, UGE achieves:
- **Hit@5: 0.6314 ± 0.0210**

**Per-seed results:**

| Seed | Hit@5 |
|------|------|
| 42   | 0.6030 |
| 123  | 0.6210 |
| 3407 | 0.6320 |
| 2025 | 0.6460 |
| 7    | 0.6550 |

These results indicate **stable performance across different random initializations**, with relatively low variance compared to the overall performance gains.

3. **small test sets**.

4.We agree that fine-grained metric spatial understanding (e.g., distance and distance–direction) remains challenging. Following this suggestion, we include **error case analysis** for distance-based queries.
- **Error case (distance confusion):**
  For the query *“Based on the spatial graph, how far is the nearest amenity from the closest amusement_park?”*, the ground truth is *Cheng San Public Library* (~100m), while UGE predicts *LiHO Tea*. We observe that *LiHO Tea* is also located near the same road (*Hougang Avenue 10*) as the amusement park, which may lead the model to **over-rely on local proximity cues** and fail to correctly compare **fine-grained distances**.
- **Failure analysis (distance-specific):**
  (1) **Weak metric grounding in SRPs:** SRPs introduce distance/direction through language, but these cues are not explicitly aligned with graph-level metric signals (e.g., exact edge distances), limiting precise distance comparison.
  (2) **Contrastive objective limitation:** The current framework is effective for semantic and relational alignment, but is less suited for encoding **continuous metric quantities** (e.g., exact distances), leading to ambiguity when multiple candidates are spatially close.
- **Future improvements (targeted for metric reasoning):**
  (1) **Metric-aware supervision:** Incorporate auxiliary objectives such as distance regression or distance-bin classification to explicitly supervise metric understanding.
  (2) **Stronger edge encoding:** Explicitly encode and normalize edge distance and bearing, and align them with SRP signals to improve data-level grounding.

//Reviewer 4: udFw//

1. We thank the reviewer for this valuable suggestion. We agree that evaluating open-ended generation tasks (e.g., spatial QA) would provide additional insights into spatial capabilities.
- **Scope of this work:**
  Our method is a **multimodal embedding framework** focused on **representation learning**, not generative reasoning. Accordingly, we adopt embedding-based **retrieval and ranking** tasks, which are standard for evaluating whether representations effectively align **visual, textual, and spatial graph information**. Our implementation (using Swift) is also tailored for **efficient embedding fine-tuning** across VLM backbones, rather than generation-oriented setups.
- **Future direction (generation):**
  We view generation as a complementary extension. A natural next step is to use UGE as a **spatially grounded embedding module** within a **SpatialRAG** pipeline:
  (1) retrieve relevant urban locations or neighborhoods using UGE embeddings, and
  (2) condition a generative model to produce **spatial QA or navigation instructions** grounded in retrieved evidence.
This setup can enhance **spatial faithfulness** and support real-world applications (e.g., travel assistance).

2. We thank the reviewer for highlighting the efficiency concern. We agree that inference latency is important for real-world deployment.
- **Scope clarification:**
  The primary focus of this work is to study and test **spatially grounded multimodal embeddings**, i.e., whether incorporating spatial graphs and the designed spatial signals (SRPs, SCCs) helps models better understand **spatial structure** and align **visual, textual, and topological information** across urban tasks (e.g., geolocation ranking, urban perception). As such, the current implementation prioritizes **representational effectiveness** over efficiency.
- **Current design:**
  We already apply **subgraph sampling**, truncating the local graph to at most 1000 nodes based on spatial proximity to the anchor image, in both training and inference. This can be further improved via **adaptive sampling**, e.g., using smaller subgraphs in low-density regions.
- **Optimization strategies (future work):**
  (1) **Offline graph precomputation:** Precompute subgraph embeddings so inference reduces to retrieval + fusion (similar to RAG pipelines).
  (2) **Graph caching:** Reuse embeddings for overlapping spatial neighborhoods across queries.
We will include a discussion of these optimization directions and trade-offs in the Appendix.

3.We thank the reviewer for this important suggestion. Following this comment, we added a **joint training baseline (“Joint”)**, where Stage 1 (SRP/SCC) and Stage 2 (graph-conditioned) data are naively combined and trained simultaneously.
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

As shown above, Joint training consistently underperforms Stage1-only, and is substantially worse than the full two-stage UGE model (reported in the main paper). This indicates that the gains are **not simply due to increased data volume or diversity**.
Instead, the results support the importance of the **progressive training strategy**:
Stage 1 injects spatial awareness through textual spatial reasoning cues while **preserving the pretrained visual–language alignment**, and Stage 2 incrementally introduces **explicit spatial structure via graph conditioning**. When trained jointly from scratch, graph-conditioned signals interfere with this gradual knowledge injection process, leading to weaker optimization.
We will clarify this comparison and emphasize the role of training curriculum more explicitly in the revision.

