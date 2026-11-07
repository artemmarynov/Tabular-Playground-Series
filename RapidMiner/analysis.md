# Review of participant 1 results

Review date: 2026-11-07. This review uses the submitted CSV, process XML,
correlation matrix, chart configurations, and manually recorded K-Means results.
The AI Studio processes were inspected statically; the clustering experiments
were not rerun. Recorded scores are preserved at their original three-decimal
precision.

## Converted result files

| File | Contents | Rows |
| --- | --- | ---: |
| `kmeans_metrics.csv` | k, total items, average within-centroid distance, Davies-Bouldin | 6 |
| `kmeans_cluster_sizes.csv` | k, cluster identifier, item count | 34 |
| `statistics.csv` | Attribute, type, missing count, mean, sample standard deviation, minimum, maximum | 30 |

Both original TXT files were parsed completely. Every CSV field was read back
and compared with the corresponding source value before the TXT files were
removed. No metric precision, cluster sizes, or statistics were discarded.
Seed, restart count, and iteration limit were not recorded in the text results;
they have not been retroactively assigned to the six runs.

## Data understanding and preparation

- The raw file has 98,000 observations, an `id` column, and 29 numeric features.
  The submitted statistics report zero missing values for every attribute.
- `f_00`-`f_06` and `f_14`-`f_21` have means close to zero and standard
  deviations close to one. This does not establish normality or lack of
  clustering information.
- The integer features `f_07`-`f_13` have standard deviations from approximately
  3.69 to 5.90. The later continuous features have standard deviations from
  approximately 1.31 to 1.58. Scaling before distance-based modeling is justified.
- The exported correlation matrix contains exactly the 29 input features and
  excludes `id`. Its strongest absolute pairwise correlation is
  `f_08` / `f_09`: -0.36073389430928154. The next strongest pairs are
  `f_09` / `f_12` (-0.295563), `f_09` / `f_13` (-0.281970),
  `f_10` / `f_13` (-0.281057), and `f_10` / `f_11` (-0.267171).
  These findings alone do not justify removing a feature.
- The six histogram JSON files select the intended features. The four boxplot
  configurations jointly cover all 29 features. These files store visualization
  settings, not plotted observations, histogram bin counts, or quartiles.
  Add rendered PNG/SVG/PDF figures for a report that can be read outside AI Studio.

The standardized CSV was checked against the raw file by attribute name, because
its `id` column appears last. All 98,000 identifiers and their row order are
preserved. All 2,842,000 transformed feature values agree with
`(raw - mean) / sample_std` within a tolerance of 1e-10; the maximum observed
absolute difference was approximately 7.20e-14. The largest absolute feature
mean was 5.87e-16 and the largest deviation of a sample standard deviation from
one was 1.21e-14. Thus the supplied standardized artifact is consistent with the
documented Z-transformation and does not show scaling or row-alignment errors.

## Process configuration

The three process files implement data inspection, preparation, and K-Means.
`id` receives the special `id` role. Both preparation and clustering explicitly
normalize all 29 features with `Z-transformation` and exclude special attributes.
The K-Means model and clustered table are correctly connected to the matching
inputs of `Cluster Distance Performance`.

The saved `03_kmeans.rmp` snapshot specifies:

| Parameter | Value |
| --- | --- |
| k | 12 |
| Local random seed enabled | true |
| Local random seed | 1234 |
| Maximum runs | 10 |
| Maximum optimization steps | 300 |
| Good start values / k-means++ | true |
| Distance | EuclideanDistance, NumericalMeasures |
| Add cluster attribute | true |
| Add as label | false |
| Main performance criterion | Davies Bouldin |
| Main criterion only | false |
| Normalize performance by feature count | false |
| Maximize parameter | true |

The local seed overrides the process-level seed of 2001 for this learner.
The snapshot documents the current configuration, not the settings of every
historical run. Confirm those settings from the experiment history before
adding them to the metrics table.

`maximize=true` preserves the reported metric sign according to the
[operator documentation](https://docs.rapidminer.com/2026.1/studio/operators/validation/performance/segmentation/cluster_distance_performance.html).
For manual comparison, a lower positive Davies-Bouldin value is preferable.
Do not reuse this setting uncritically in a parameter optimizer that maximizes
the returned criterion. The average-distance column retains the operator's
original name and is not relabeled as SSE or inertia without checking its
precise definition.

Reproducibility limitations:

- Every process reads `C:\Users\admin\Downloads\data.csv`. The path must be
  adjusted on another computer, including the current repository checkout.
- The preparation process produces a result table but contains no `Write CSV`
  operator. The provided standardized CSV was exported separately; regeneration
  and export are therefore not yet a single saved workflow.
- `return_preprocessing_model=false` means a fitted normalization model is not
  returned. Preserve it when applying the workflow to genuinely new observations.
- No exported fitted K-Means model, centroid table, or per-row cluster assignments
  are present. Cluster sizes alone do not allow cluster profiles or independent
  recalculation of clustering scores.
- The CSV contains numeric ASCII content, so the configured windows-1252 reading
  does not explain a data error here. UTF-8 remains preferable for future text data.

## K-Means results and interpretation

| k | Avg. within centroid distance | Davies-Bouldin | Smallest cluster | Largest cluster |
| ---: | ---: | ---: | ---: | ---: |
| 2 | 27.828 | 4.706 | 38,311 | 59,689 |
| 3 | 27.042 | 4.544 | 18,701 | 40,562 |
| 4 | 26.447 | 4.268 | 16,786 | 30,695 |
| 5 | 25.985 | 4.123 | 15,504 | 21,569 |
| 8 | 25.174 | 4.021 | 11,083 | 13,808 |
| 12 | 24.506 | 3.923 | 5,802 | 10,268 |

For every recorded run, the number of clusters equals k, their identifiers are
contiguous, and their sizes sum to 98,000. There are no empty clusters in these
records. At k=12, individual cluster sizes range from about 5.92% to 10.48% of
the data. Balanced sizes do not establish that the recovered groups are real
operating states.

Both recorded metrics decrease over the tested values of k. From k=2 to k=12,
the recorded average-distance criterion falls by about 11.94% and Davies-Bouldin
by about 16.64%. **k=12 is the best recorded candidate by Davies-Bouldin.**
It is also the upper boundary of the explored range. These six runs do not
establish an optimal or true number of clusters, a reliable elbow, stability
across initializations, or superiority over GMM and DBSCAN. There is no universal
Davies-Bouldin cutoff that turns these scores into a claim of good separation.
True-label accuracy and ARI cannot be obtained from this unlabeled CSV.

## Remaining participant 1 work and handoff

1. Record actual settings for all historical runs when known; otherwise leave
   them explicitly unknown. Use consistent preprocessing and record the seed
   for every new run.
2. Complete the missing k values 6, 7, 9, 10, and 11 under one documented
   configuration. If the best result remains at k=12, examine 13-20 before
   describing the range as adequately explored.
3. Repeat the three best candidate k values with max runs 30 and seeds
   1234, 42, and 43. Compare score variability and cluster-size consistency.
   Agreement of per-row assignments, with cluster-label permutations accounted
   for, provides a stronger stability check when assignments are available.
4. Export per-row `id` / `cluster` assignments, the centroid table, rendered EDA
   figures, and the chosen model. Record the exact configuration that generated
   each artifact.
5. Give participant 2 the standardized CSV, preparation process, metrics, and
   assignments. Reassign the special `id` role after CSV import. Do not feed
   the K-Means cluster labels into GMM or DBSCAN as ordinary input features.
   For comparable evaluation, agree on the same feature space and the same
   fixed observation IDs for any sampled scores.
6. Write the participant 1 CRISP-DM report sections, distinguishing verified
   data properties, recorded model scores, hypotheses, and pending validation.

The instructor's supervised-versus-unsupervised rubric adaptation remains an
external requirement to resolve. Additional clustering experiments alone do
not establish compliance with the supervised-model requirements.
