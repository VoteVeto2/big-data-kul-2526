# Codex review of all mock-exam solutions

Reviewed files:

- `Mock-Exam-1.html` through `Mock-Exam-4.html` (80 MCQ keys and per-option explanations)
- `Mock-Exam-1-Solutions.html` through `Mock-Exam-4-Solutions.html` (16 open questions, 33 answer parts)

## Overall judgment

Opus got most of the material right and stayed very close to the course notes, but the solutions are **not fully technically reliable**.

Across 113 independently judged answer units:

| Judgment | Count |
|---|---:|
| Correct | 63 |
| Correct, but wording/qualification should improve | 33 |
| Needs correction | 17 |

The most serious errors are repeated rather than isolated:

1. **Hadoop MapReduce is described incorrectly.** A Hadoop reducer is not normally invoked on a partial result and then re-invoked when a “straggler” arrives. Reducer output also does not have to match mapper-output types. Those constraints apply to a reusable combiner/aggregation state, not to every reducer. This affects Exam 1 Q18, Exam 2 Q18, Exam 2 Open 4, and part of Exam 3 Open 4.
2. **Standard UMAP is wrongly called “parametric by default.”** Standard `umap.UMAP` can transform new points, but it is distinct from neural-network-based `ParametricUMAP`. This affects Exam 3 Open 3b.
3. **The PDP construction is wrong/internally conflated.** A standard PDP replaces the feature of interest for every observed row and averages predictions over the observed complement features. Fixing all other features at median/mode is not the same PDP. This affects Exam 4 Open 1a/1c.
4. **The diffusion answer says the network predicts noise added “from one step to the next.”** In the standard DDPM epsilon parameterization, training samples a timestep and predicts the epsilon used in the closed-form noisy sample \(x_t\), not merely the last incremental noise.
5. **Several broad family-level claims are actually algorithm-specific**, especially “boosting does not bootstrap; it reweights instances,” which describes AdaBoost rather than boosting in general.

Several of these mistakes already exist in the repository’s extracted/summary course material. Opus generally reproduced those notes faithfully; it did not independently correct their technical inaccuracies.

## Highest-priority corrections

### 1. Hadoop reducer/combiner semantics

Affected:

- `Mock-Exam-1.html:545`
- `Mock-Exam-2.html:551` (Q18)
- `Mock-Exam-2-Solutions.html:369`
- `Mock-Exam-3-Solutions.html:369`

Correct framing:

- Mapper intermediate outputs are grouped and delivered to the reducer.
- Hadoop may run an **optional combiner** zero or more times on mapper-side intermediate data.
- A reducer may emit zero or more records and its output types need not match the mapper’s intermediate types.
- For a combinable mean, carry `(sum, count)` (or a mathematically equivalent state), merge states associatively/commutatively, and divide only when producing the final result.
- A reducer does not finish a key, later receive a straggler value, and then re-reduce its previous output in the manner described.
- Map outputs are fetched from mapper nodes; they are not all “persisted to HDFS” for reducer fault tolerance.

### 2. UMAP

Affected: `Mock-Exam-3-Solutions.html:364`

Replace “UMAP is parametric by default” with:

> Standard UMAP is non-neural/non-parametric in the usual sense, but its implementation provides a `transform` operation for embedding new points into a learned space. Parametric UMAP is a separate neural-network variant that learns an explicit mapping.

The conclusion that UMAP is often more practical than t-SNE for transforming new data is defensible; the stated reason is not.

### 3. PDP/ICE

Affected: `Mock-Exam-4-Solutions.html:354`

Correct construction:

> For each grid value \(v\), replace the feature of interest by \(v\) in every row, preserve each row’s other feature values, predict all rows, and average those predictions.

Median/mode replacement creates a prediction profile at a reference point, not the standard marginal PDP. The main standard pitfall omitted from the answer is that PDP/ICE can create unrealistic combinations when the selected feature is correlated with other features.

### 4. Diffusion

Affected: `Mock-Exam-1-Solutions.html:364`

For the usual DDPM epsilon objective:

> Sample \(x_0\), a timestep \(t\), and \(\epsilon\sim N(0,I)\); construct \(x_t=\sqrt{\bar\alpha_t}x_0+\sqrt{1-\bar\alpha_t}\epsilon\); train \(\epsilon_\theta(x_t,t)\) to predict that \(\epsilon\).

Generation then uses the prediction to parameterize/sample the reverse transition. “Remove the noise added from one step to the next” is an intuitive simplification, but not an accurate statement of the training target.

## Complete MCQ review

### Mock Exam 1

| Q | Judgment | Notes |
|---:|---|---|
| 1 | Correct | Useful means actionable. |
| 2 | Correct | Bayes formula is correct. |
| 3 | Correct | Target-aware grouping is the intended advantage. |
| 4 | Correct | L1 can create exact zero coefficients; L2 generally does not. |
| 5 | Correct | Correct lazy-learning characterization. |
| 6 | Correct with caveat | PR is usually more informative under severe imbalance. AUROC itself is prevalence-invariant as a ranking statistic; “many TNs keep FPR small” is an intuitive but incomplete explanation. |
| 7 | Correct | \(AR=2AUC-1\). |
| 8 | Correct with caveat | Say “base value/expected model output,” not always “base rate”; additivity is on the explained output scale. |
| 9 | Correct | Sequential correction is the relevant distinction. |
| 10 | Correct with caveat | The LightGBM key is sound. XGBoost defaults are usually depth-wise, but XGBoost also supports loss-guided growth, so the distractor explanation is too absolute. |
| 11 | Correct | MLP nonlinearity resolves XOR-type limitations. |
| 12 | Correct | Weight sharing is the key efficiency. |
| 13 | Correct with caveat | The key is correct. The explanation saying softmax is a smooth approximation to `argmax` is loose; soft-argmax is the closer term, while log-sum-exp approximates max. |
| 14 | Correct with caveat | Convexity is not a metric axiom. The listed identity axiom should be \(d(a,b)=0\iff a=b\), not merely \(d(a,a)=0\). |
| 15 | Correct | Lower DBI is better. |
| 16 | Correct | BM25 saturation and length normalization are correctly characterized. |
| 17 | Correct | Shared recurrent weights and BPTT are correct. |
| 18 | **Needs correction** | The averaging-state idea is useful, but the claimed Hadoop reducer lifecycle and mandatory same-structure rule are wrong. See high-priority finding 1. |
| 19 | Correct with caveat | `DataFrame = Dataset[Row]` is literal in Scala/Java Spark SQL; in Python/R it is best presented as the conceptual/internal relationship. |
| 20 | Correct | Girvan–Newman removes highest-edge-betweenness edges and recomputes. |

### Mock Exam 2

| Q | Judgment | Notes |
|---:|---|---|
| 1 | Correct | Association-rule mining is unsupervised/descriptive. |
| 2 | Correct | All learned preprocessing must be fitted on training data. |
| 3 | **Needs correction** | “20 and 15 are the closest” is false under absolute distance: 2 and 1.25 are closer. Cash/travel are closer only under a relative/log-odds criterion. The question must state the similarity criterion or use an unambiguous pair. |
| 4 | Correct | Information gain is 1 bit. |
| 5 | Correct | Recall is \(7/9\). |
| 6 | Correct | Expected-loss calculation and positive classification are correct. |
| 7 | Correct with caveat | CV is often pessimistic relative to a final model trained on all data, but bias depends on the procedure. A held-out test set estimates generalization; it does not “guarantee” it. |
| 8 | Correct | Random-feature selection is per split. |
| 9 | Correct with caveat | \(\alpha=0\) at error 0.5. In practice this is a no-progress/termination case; the Open 3 answer should consistently use \(\varepsilon\ge 0.5\), not only \(>0.5\). |
| 10 | Correct | OOB fraction is approximately \(e^{-1}\approx 37\%\). |
| 11 | Correct with caveat | Saturation and small derivatives are the core cause. “Derivatives are at most 1” alone is insufficient because tanh reaches 1 at zero. |
| 12 | Correct with caveat | Validation can beat training because dropout is disabled. Modern “inverted dropout” scales during training, so saying activations are scaled at test time is implementation-dependent and usually false. |
| 13 | Correct | Mode collapse definition is correct. |
| 14 | Correct with caveat | Hard vs soft assignment is correct. A general GMM is richer than merely “soft k-means” because it can model unequal/elliptical covariance structures. |
| 15 | Correct | Ordinary t-SNE lacks a direct transform for new data. |
| 16 | Correct | Binary BoW vector is correct. |
| 17 | Correct | LSTM has cell and hidden state; GRU has one recurrent state. |
| 18 | **Needs correction** | The arithmetic \(225\) is correct for the invented re-reduction scenario, but that scenario is not the normal Hadoop reducer lifecycle. |
| 19 | Correct with caveat | The key is correct. The wrong-option explanation should not say Structured Streaming “discards the source” or always keeps only minimal state; state size depends on the query. |
| 20 | Correct with caveat | Uniform vs biased teleport is correct. “Lazy” random walks/resting are independent of whether PageRank is personalized. |

### Mock Exam 3

| Q | Judgment | Notes |
|---:|---|---|
| 1 | Correct | Intended non-parametric/data-driven answer. |
| 2 | Correct | Standardization does not change tree split ordering. |
| 3 | Correct with caveat | WoE is monotonic in the category’s class-odds ratio by construction; it is not necessarily monotonic in an original numeric/ordinal feature unless bins are constrained/ordered accordingly. |
| 4 | Correct with caveat | Fully grown trees are unstable and poorly calibrated. Pure leaves still provide empirical 0/1 probabilities; the issue is that those probabilities are unreliable, not impossible. |
| 5 | Correct | AUROC is threshold-independent. |
| 6 | Correct | Never resample the test set. |
| 7 | Correct | Four classes give six one-vs-one classifiers and voting. |
| 8 | Correct with caveat | Useful teaching shorthand, but boosting can affect both bias and variance; the split is not a theorem applying identically to every boosting method. |
| 9 | Correct | Negative-gradient formulation is correct. |
| 10 | Correct | Defensible “usually” statement for ordinary tabular settings. |
| 11 | Correct | He initialization mainly targets signal/gradient propagation. |
| 12 | Correct with caveat | Same principal subspace holds under the usual linear/MSE/bottleneck assumptions. PCA is not strictly unique because signs and degenerate eigenspaces can vary. |
| 13 | **Needs correction** | Divisive clustering need not consider “all possible” splits. Some divisive methods are expensive, but the causal claim and universality are false. |
| 14 | Correct | Correct \(D^2\)-weighted k-means++ initialization. |
| 15 | Correct | Lift greater than 1 means positive association. |
| 16 | Correct with caveat | This is a common empirical rule of thumb, not a universal quality/data requirement. |
| 17 | Correct | Causal masking explanation is correct. |
| 18 | Correct with caveat | Avro is row-oriented and supports schema evolution, but field rename/removal compatibility depends on reader/writer schemas, defaults, and aliases. |
| 19 | Correct | Offline training plus online prediction is a valid/common pattern. |
| 20 | Correct | A plain DNN over a flattened adjacency matrix is not permutation invariant. |

### Mock Exam 4

| Q | Judgment | Notes |
|---:|---|---|
| 1 | Correct | “Analyze the Data” is the stated analytics phase. |
| 2 | **Needs correction** | `high/medium/low` has a natural order, so calling the ordering false contradicts the example. Integer coding still imposes unjustified equal spacing. Use truly nominal labels or change the key to focus only on spacing. |
| 3 | Correct with caveat | Correct odds-ratio interpretation, conditional on other features being held fixed. |
| 4 | **Needs correction** | Binary splits mitigate the direct preference for many branches but do not remove split-selection bias: high-cardinality variables still offer many candidate binary partitions/cut points. |
| 5 | Correct | Accuracy failure under 1% prevalence is correctly explained. |
| 6 | Correct | Basic SMOTE interpolation description is correct. |
| 7 | Correct | MDI high-cardinality/overfitting bias is correctly identified. |
| 8 | Correct | \(\sqrt{25}=5\) is the common classification heuristic. |
| 9 | Correct with caveat | XGBoost uses first- and second-order statistics, but this is not its only computational distinction and second-order boosting is not conceptually exclusive to XGBoost. |
| 10 | Correct | Output is \(8\times28\times28\). |
| 11 | Correct | Pooling result is correct. |
| 12 | Correct | Residual form \(F(x)+x\) is correct. |
| 13 | Correct with caveat | Single linkage chains. The explanation that average linkage is “robust to noise” is too strong. |
| 14 | Correct | DBSCAN noise/non-convex statement is correct. |
| 15 | Correct with caveat | Correct for the course’s normalized-TF definition; TF-IDF implementations also commonly use raw or log-scaled TF. |
| 16 | Correct | fastText subword explanation is correct. |
| 17 | Correct | RDD transformations are lazy; actions trigger evaluation. |
| 18 | **Needs correction** | Spark’s execution plan is a DAG, hence acyclic. Spark supports iterative workflows, but “a DAG supporting cyclic data flow” is contradictory wording. |
| 19 | Correct | Relational-neighbor probability is \(2/5\). |
| 20 | **Needs correction/qualification** | In a deliberately transductive task, unlabeled test-node features/structure being visible is part of the protocol, not automatically “data leakage.” It is leakage when that visibility violates the intended deployment/temporal split. The answer presents a debated evaluation critique as a universal definition. |

## Complete open-question review

### Mock Exam 1

| Part | Judgment | Notes |
|---|---|---|
| Open 1a | Correct | Automation/business-purpose explanation is coherent and course-aligned. |
| Open 1b | Correct | Orthogonality and the V’s are correctly stated. |
| Open 2a | Correct with caveat | AUC-PR is the strongest answer. AUROC is threshold-independent but often less revealing under heavy imbalance, which the answer does acknowledge. |
| Open 2b | Correct with caveat | Train-only resampling and the accuracy trade-off are right. Sampling and class weighting are only exactly equivalent under particular learners/losses; SMOTE and regularization can break the equivalence. |
| Open 3a | **Needs correction** | Standard DDPM epsilon prediction is described incorrectly as one-step incremental-noise prediction. See high-priority finding 4. |
| Open 3b | Correct | GAN instability/mode-collapse comparison is sound. |
| Open 4a | Correct | Fixed context bottleneck vs per-step attention is accurately explained. |
| Open 4b | Correct | Q/K/V, scaling, token mixing, and position-wise FFN distinction are correct. |

### Mock Exam 2

| Part | Judgment | Notes |
|---|---|---|
| Open 1a | Correct with caveat | Formula/sign are right. “Monotonic with target” should be phrased as monotonic in the category class-odds ratio, not automatically in the feature’s natural order. |
| Open 1b | Correct with caveat | Leakage explanation is correct. The \(IV>0.1\) cutoff is a domain heuristic, not a universal definition of importance. |
| Open 2a | **Needs correction** | “Boosting does not bootstrap; it reweights misclassified instances” describes AdaBoost. Gradient boosting fits gradients/residuals and stochastic boosting can subsample rows. |
| Open 2b | Correct with caveat | Bagging-as-variance-reduction and boosting-as-bias-reduction are useful defaults, but both methods can affect both terms. |
| Open 3a | Correct | Apriori’s support-then-rule-generation structure and power-set argument are correct. |
| Open 3b | Correct | Downward closure and join/prune are correct. |
| Open 4a | **Needs correction** | Hadoop reducers may emit zero or more records, not necessarily one pair per key, and official Hadoop documentation says reducer output is not sorted. |
| Open 4b | **Needs correction** | The answer incorrectly turns combiner/merge-state requirements into mandatory reducer semantics and uses a nonexistent “late straggler re-reduce” lifecycle. |

Metadata issue: Exam 2 Open 3 is tagged `Ch 1 · Introduction` in `Mock-Exam-2.html`, but `Ch. 06 · Unsupervised` in the solution page. Chapter 6 is the plausible classification.

### Mock Exam 3

| Part | Judgment | Notes |
|---|---|---|
| Open 1a | Correct | Parametric/data-driven spectrum and boundary shapes are correct. |
| Open 1b | Correct with caveat | Outlier statements are course-aligned. Unregularized logistic regression does not mathematically require scaling, though scaling is often useful for optimization and is required for fair regularization. |
| Open 2a | **Needs correction** | At \(\varepsilon=0.5\), \(\alpha=0\) and the round makes no progress; the stopping condition should normally be \(\varepsilon\ge0.5\), not only \(>0.5\). This also conflicts with Exam 2 Q9’s explanation. |
| Open 2b | Correct | \(\frac12\ln 3\approx0.5493\) and the outlier sensitivity are correct. |
| Open 3a | Correct | PCA/t-SNE comparison is sound. |
| Open 3b | **Needs correction** | Standard UMAP is not “parametric by default.” It does provide a transform for new data; Parametric UMAP is a separate neural variant. |
| Open 4a | **Needs correction** | MapReduce intermediate/fault-tolerance details are wrong: map output is not generally persisted to HDFS, reducer fetches mapper partitions, and failed map work can be recomputed. “DAG supporting cyclic data flow” is also poor wording. |
| Open 4b | Correct | DataFrame/Dataset/MLlib accessibility explanation is broadly accurate. |

### Mock Exam 4

| Part | Judgment | Notes |
|---|---|---|
| Open 1a | **Needs correction** | Standard PDP construction is conflated with fixing complement features at median/mode. See high-priority finding 3. |
| Open 1b | Correct | ICE as one line per instance and PDP as their average is correct. |
| Open 1c | **Needs correction** | Interactions can be hidden by averaging, but a PDP marginalizes over complement features rather than simply fixing all of them. The more standard pitfall—unrealistic combinations under correlated features—is omitted. |
| Open 2a | Correct with caveat | Point-vs-distribution and reconstruction/KL terms are correct. A complete encoding explanation should include the reparameterization \(z=\mu+\sigma\odot\epsilon\), which enables backpropagation; practical VAEs usually output diagonal log-variance, not a full covariance matrix. |
| Open 2b | Correct | KL regularization/latent-space explanation is sound. |
| Open 3a | Correct with caveat | Main comparison is correct. Sparse vectors are not inherently unusable by ML algorithms; many linear/text methods handle them efficiently. |
| Open 3b | Correct with caveat | CBOW/skip-gram direction and hidden dimension are correct. Word2vec learns input and output/context matrices; retaining only the input matrix \(W\) is a convention/simplification, not the only possible representation. |
| Open 4a | Correct | DeepWalk and node2vec’s biased second-order walk are correctly described. |
| Open 4b | Correct with caveat | GraphSAGE is inductive; not every GNN architecture/training setup is automatically inductive. |

## Repository/format checks

- All four exam files contain exactly 20 MCQs and 4 open questions.
- Every MCQ has exactly one `data-correct="1"` option.
- Every solution page contains all four corresponding open questions.
- Open-question prompt and part text match between each exam and solution page.
- All local exam/solution links resolve.
- Inline JavaScript passes `node --check`.
- Two solution pages contain unescaped raw ampersands in visible metadata:
  - `Mock-Exam-2-Solutions.html:358`
  - `Mock-Exam-3-Solutions.html:358`
  Browsers tolerate this, but `&amp;` is valid HTML.

## Primary references used for external verification

- [Apache Hadoop MapReduce Tutorial](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) — mapper/reducer/combiner lifecycle, reducer output, shuffle.
- [Apache Spark RDD Programming Guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html) — caching, recomputation, lineage, shuffle disk behavior.
- [UMAP: Transforming New Data](https://umap-learn.readthedocs.io/en/latest/transform.html) — standard UMAP’s transform support.
- [UMAP: Parametric UMAP](https://umap-learn.readthedocs.io/en/latest/parametric_umap.html) — separate neural parametric variant.
- [scikit-learn PDP/ICE documentation](https://scikit-learn.org/stable/modules/partial_dependence.html) and [implementation](https://github.com/scikit-learn/scikit-learn/blob/main/sklearn/inspection/_partial_dependence.py) — marginal definition and brute-force construction.
- [Ho et al., Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239) — epsilon-prediction training objective.
- [Kingma & Welling, Auto-Encoding Variational Bayes](https://arxiv.org/abs/1312.6114) — reparameterized variational training.

## Bottom line

The answer set is useful for revision and mostly matches the lecture material. It should not be published as an authoritative technical solution set until the 17 “Needs correction” entries above are fixed, especially the repeated MapReduce, UMAP, PDP, and diffusion explanations.
