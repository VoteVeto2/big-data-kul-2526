# Peer Review: Factual Accuracy of `summary/chapters`

Date: 2026-06-24

Scope: reviewed `01-introduction.html` through `09-deep-learning-graphs.html` for factual accuracy and exam-risk wording. I did not compare line-by-line against the original KU Leuven slides, so findings below are based on standard ML, data science, big data, and graph learning definitions.

## Overall Verdict

The chapters are broadly useful and mostly accurate. The main risk is not missing content, but overconfident simplifications: several statements are phrased as absolutes when the correct concept is conditional. I would fix the high-priority items before relying on these notes for exam preparation.

## High-Priority Corrections

1. `02-preprocessing.html:L627` says Lasso "does not work" when `#instances < #features`.
   This is false. Lasso is commonly used in high-dimensional `p > n` settings. Better: ordinary least squares is underdetermined when `p > n`; Lasso can still work through regularization but may be unstable with correlated predictors and can select at most about `n` active variables.

2. `05-deep-learning-images.html:L528-L530` gives softmax as `exp(-beta z_i) / sum exp(-beta z_j)` while describing it as an argmax approximation.
   With logits/scores, standard softmax is `exp(z_i) / sum exp(z_j)` or `exp(beta z_i) / sum exp(beta z_j)`. The negative sign turns it into a softmin unless `z` is defined as an energy/cost.

3. `06-unsupervised.html:L659-L687` says UMAP is "parametric by default" and uses that as the quiz answer.
   Standard UMAP is not parametric by default. It is a graph/manifold embedding method with an out-of-sample `transform` capability. Parametric UMAP is a separate neural-network variant. Update the explanation to: UMAP is often preferable to t-SNE in pipelines because standard implementations can transform new points, while parametric UMAP is optional.

4. `03-evaluation.html:L615-L619` gives a nonstandard AIC formula and an adjusted `r^2` formula with ambiguous `k`.
   For Gaussian linear regression, AIC is typically `n log(SSE/n) + 2k` plus a constant, where `k` is the number of estimated parameters. The formula shown resembles a prediction-error correction, not standard AIC. Adjusted `r^2` should be `1 - (1-r^2)(n-1)/(n-p-1)` if `p` means number of predictors excluding the intercept.

5. `08-big-data.html:L667-L672` treats hopping and tumbling windows as the same.
   They are not the same. Tumbling windows are non-overlapping consecutive windows. Hopping windows advance by a fixed hop and may overlap or have gaps depending on hop size.

6. `03-evaluation.html:L432-L440` describes specificity as "of the actual negatives, how many flagged?"
   Specificity is `TN/(TN+FP)`: of actual negatives, how many were correctly not flagged. The current wording describes false positives, not true negatives.

7. `02-preprocessing.html:L417-L435` overstates scaling.
   Standardization gives zero mean and unit variance, not necessarily a normal `N(0,1)` distribution. Also, scaling is essential for distance-based models and regularized/gradient-based optimization, but plain unregularized regression does not require scaling for correctness.

8. `08-big-data.html:L395-L399` oversimplifies Hadoop 3 erasure coding.
   The common 1.5x overhead comes from schemes such as Reed-Solomon `6 data + 3 parity`, not literally "1 parity block per 2 data blocks" in the simple replication sense. Keep the 1.5x contrast, but phrase the parity layout more carefully.

## Important Clarifications

- `02-preprocessing.html:L480-L494`: "higher WoE = less risk" depends entirely on which class is in the numerator. If `WoE = ln(good share / bad share)`, higher means lower risk; if the class coding is reversed, the interpretation reverses.

- `02-preprocessing.html:L536`: linear-regression noise is usually written as `epsilon ~ N(0, sigma^2)` if `sigma` denotes standard deviation.

- `02-preprocessing.html:L579`: logistic regression is not automatically well-calibrated. It is often better calibrated than many classifiers when correctly specified, but calibration can still fail under misspecification, regularization, imbalance, or dataset shift.

- `04-trees-ensembles.html:L402-L405`: averaging probabilities and majority voting are not "virtually guaranteed" to agree, even with many models. They often agree when base probabilities are similarly calibrated, but can diverge.

- `04-trees-ensembles.html:L485-L488`: the statement about squared-error loss penalizing predictions that are "too correct" only makes sense in a binary margin setup with labels like `y in {-1,+1}`. It is not true for ordinary regression MSE in general.

- `05-deep-learning-images.html:L421-L430`: folding the sigmoid derivative into the learning rate is only a rough simplification. The derivative changes per instance and per step, so it is not a constant learning-rate factor.

- `05-deep-learning-images.html:L570-L572`: dropout scaling is implementation-dependent. Original dropout scales activations at test time; modern "inverted dropout" scales during training and does no test-time scaling.

- `05-deep-learning-images.html:L675-L680`: YOLO is described as using IoU "as the loss metric." IoU is central for evaluation, confidence targets, matching/NMS, and newer IoU-style losses, but original YOLO used a composite loss, not pure IoU loss.

- `07-deep-learning-text.html:L598-L601`: scaled dot-product attention is usually written `softmax(QK^T / sqrt(d_k))V`. The earlier `Q^T K` notation is dimensionally confusing unless a column-vector convention is explicitly stated.

- `07-deep-learning-text.html:L630-L631`: "GPT uses sparse self-attention" is not generally true across GPT-style models. Decoder-only masked self-attention is the core fact; sparsity is model-specific.

- `07-deep-learning-text.html:L644-L655`: Transformer generation does not necessarily recompute all attention from scratch in production. With KV caching, each next token attends over previous keys/values, so total generation is still roughly quadratic over the final sequence length, but the "recompute entire sequence" wording is too strong.

- `08-big-data.html:L384-L391`: HDFS block size of 64 MB is historically correct for older Hadoop defaults, but modern defaults commonly use 128 MB or more. Phrase as "historically 64 MB; often 128 MB+ today" unless the course slide requires 64 MB.

- `08-big-data.html:L501-L507`: "DAG engine supporting cyclic data flow" is conceptually confusing because DAG means acyclic. Spark supports iterative algorithms by building repeated acyclic execution graphs, not by making one cyclic DAG.

- `08-big-data.html:L597-L601`: the MLlib sentence appears to assign "feature parity in 2.3" to the older RDD API. The usual story is: `spark.ml` DataFrame API became the primary API; the older `spark.mllib` RDD API moved to maintenance mode.

## Chapter-Level Notes

- `01-introduction.html`: no major conceptual mistakes found. The big-data volume/traffic/quality statistics and AI milestone dates are time-sensitive; avoid treating them as durable facts unless they match the lecture slides.

- `02-preprocessing.html`: strong chapter, but fix Lasso, standardization, WoE interpretation, and calibration wording.

- `03-evaluation.html`: good structure, but fix specificity wording, AIC/adjusted `r^2`, and avoid saying traditional models inherently optimize accuracy. Many optimize log-loss, hinge loss, impurity, or squared error instead.

- `04-trees-ensembles.html`: mostly accurate. Main issue is overclaiming that probability averaging and majority voting converge to the same result.

- `05-deep-learning-images.html`: several high-yield details need tightening: softmax sign, dropout scaling, YOLO loss wording, and the simplified perceptron gradient.

- `06-unsupervised.html`: main factual issue is UMAP being called parametric by default. Fix the quiz because it currently reinforces the mistake.

- `07-deep-learning-text.html`: core concepts are solid. Clarify attention notation, GPT sparsity, and KV-cache generation costs.

- `08-big-data.html`: useful and mostly correct, but fix window terminology, erasure-coding wording, Spark DAG/cyclic phrasing, and modern HDFS block-size nuance.

- `09-deep-learning-graphs.html`: no major factual mistakes found. The node2vec BFS/DFS description is consistent with the original paper: BFS-like walks align with structural equivalence; DFS-like walks align with homophily/community exploration.
