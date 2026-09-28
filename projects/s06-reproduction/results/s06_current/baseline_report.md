# S06 frozen baseline analysis

This report uses the reconstructed 148-event dataset and the frozen 118/27/3 split. The checkpoint is the 10,000-update `best` model; no training was run.

## Frozen protocol

- checkpoint: `D:\论文\GNN-UDS\surrogate\model\shunqing\s06_formal_10000_b8_seed42\best`
- best update: `9098`; best validation loss: `0.025691155`
- evaluation: true 60-step history for every non-overlapping 60-step forecast window (`stride=60`)
- rainfall grouping: total event rainfall; cut points computed only from the 118 training events
- training cut points: q1=26.600, q2=42.200
- rainfall alignment: nominal event samples use `rains.npy[59:59+duration]`; the integrated totals agree with `event_manifest.csv` within 1.7e-5 mm

## Event and split counts

- all events: 148; train=118, validation=27, held-out test=3
- held-out test events: `test_bpswmm_118`, `test_bpswmm_133`, `test_bpswmm_1612`

## Group summaries

Pooled metrics:
| split | rainfall_group | event_count | window_count | node_rmse_pooled | node_mae_pooled | pipe_rmse_pooled | pipe_mae_pooled | flood_volume_rmse_pooled | flood_volume_mae_pooled | flood_f1_pooled | flood_precision_pooled | flood_recall_pooled |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| heldout_test | large | 3 | 43 | 1.9949 | 0.6923 | 1.5784 | 0.5099 | 2.2512 | 0.5535 | 0.9920 | 0.9936 | 0.9903 |
| validation | large | 11 | 124 | 5.4843 | 1.2659 | 4.0778 | 0.9106 | 6.9757 | 1.2847 | 0.9859 | 0.9878 | 0.9839 |
| validation | medium | 13 | 157 | 1.7004 | 0.6050 | 1.3247 | 0.4435 | 1.9397 | 0.4362 | 0.9902 | 0.9921 | 0.9884 |
| validation | small | 3 | 26 | 1.0455 | 0.4219 | 0.7849 | 0.3042 | 1.0440 | 0.2648 | 0.9901 | 0.9901 | 0.9901 |

Event-level distribution metrics (mean / median / standard deviation):
| split | rainfall_group | event_count | node_rmse_event_mean | node_rmse_event_median | node_rmse_event_std | node_mae_event_mean | node_mae_event_median | node_mae_event_std | pipe_rmse_event_mean | pipe_rmse_event_median | pipe_rmse_event_std | pipe_mae_event_mean | pipe_mae_event_median | pipe_mae_event_std | flood_volume_rmse_event_mean | flood_volume_rmse_event_median | flood_volume_rmse_event_std | flood_volume_mae_event_mean | flood_volume_mae_event_median | flood_volume_mae_event_std | flood_f1_event_mean | flood_f1_event_median | flood_f1_event_std |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| heldout_test | large | 3 | 1.9522 | 1.8196 | 0.5347 | 0.7047 | 0.7151 | 0.1348 | 1.5374 | 1.4186 | 0.4515 | 0.5193 | 0.5283 | 0.1022 | 2.1520 | 1.8468 | 0.7600 | 0.5643 | 0.5275 | 0.1635 | 0.9919 | 0.9918 | 0.0003 |
| validation | large | 11 | 4.5682 | 3.4650 | 2.9840 | 1.2807 | 1.0159 | 0.5680 | 3.4793 | 2.7542 | 2.1134 | 0.9215 | 0.7456 | 0.3822 | 5.6839 | 4.3618 | 4.0087 | 1.2970 | 0.9628 | 0.7923 | 0.9860 | 0.9882 | 0.0068 |
| validation | medium | 13 | 1.8311 | 1.8355 | 0.5113 | 0.6749 | 0.6967 | 0.1699 | 1.4283 | 1.4529 | 0.4243 | 0.4954 | 0.5130 | 0.1255 | 2.0630 | 2.0174 | 0.8858 | 0.5002 | 0.5136 | 0.1634 | 0.9900 | 0.9904 | 0.0026 |
| validation | small | 3 | 1.0446 | 1.0252 | 0.0367 | 0.4203 | 0.4118 | 0.0251 | 0.7849 | 0.7771 | 0.0276 | 0.3025 | 0.3002 | 0.0194 | 1.0534 | 1.0423 | 0.0929 | 0.2659 | 0.2688 | 0.0174 | 0.9901 | 0.9900 | 0.0004 |

## Interpretation checks

The validation rows, rather than a single held-out event, should determine whether error increases consistently with rainfall group. `group_dominance.csv` gives a leave-one-worst-event-out check; a large change in `mean_without_worst` indicates that a group contrast is not stable.

On validation, node, pipe, and flood-volume RMSE all increase from small to medium to large rainfall groups. The large-group contrast is not caused solely by `bpswmm_1011`: after removing that event, the large-group event-level means remain node=3.8299, pipe=2.9637, and flood-volume=4.6608 RMSE.

The maximum one-minute rainfall rate is more strongly associated with error than total rainfall in validation (Pearson r about 0.93-0.95 versus 0.69-0.71 for the three RMSE measures). This is an association, not a causal claim.

The three held-out events are reported for final confirmation only. They do not provide enough coverage to establish a three-level rainfall trend by themselves.

All three held-out events fall in the large-total-rainfall bin, while their totals are 54.4, 80.7, and 51.9 mm and their peak one-minute rates are 12.8, 24.0, and 9.9 mm/h. The training maxima are 156.0 mm total and 36.6 mm/h peak, so `test_bpswmm_133` is not a clear high-rainfall extrapolation case.

The complete event-level rainfall feature table is `event_rainfall_features.csv`; event-level teacher-forced node, pipe, flood-volume, and flood-classification metrics are in `event_teacher_forced_metrics.csv`, and the joined table is `event_rainfall_error_merged.csv`.

Rainfall units: `rains.npy` is a one-minute rate in mm/h; integrated totals and peak-split amounts in the feature table are in mm. Integrated totals were checked against the event manifest.

Selected validation correlations (the complete matrix is in `rainfall_error_correlations.csv`):
| split | feature | error | event_count | pearson_r | spearman_r |
| --- | --- | --- | --- | --- | --- |
| validation | total_rainfall | node_rmse | 27 | 0.7012 | 0.8223 |
| validation | total_rainfall | pipe_rmse | 27 | 0.7083 | 0.8236 |
| validation | total_rainfall | flood_volume_rmse | 27 | 0.6855 | 0.8065 |
| validation | max_step_intensity | node_rmse | 27 | 0.9367 | 0.9327 |
| validation | max_step_intensity | pipe_rmse | 27 | 0.9494 | 0.9400 |
| validation | max_step_intensity | flood_volume_rmse | 27 | 0.9311 | 0.9324 |
