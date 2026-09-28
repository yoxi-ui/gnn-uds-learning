# Conditioned S06 screening comparison

This is a single-seed screening run configured for 5000 epochs (completed). The comparison uses checkpoint `best` from `s06_conditioned_5000_b8_seed42_full`, whose run summary records `best_epoch=4876` and `best_validation_loss=0.0281887360`. It uses the frozen 118/27/3 split and the same teacher-forced non-overlapping 60->60 protocol; it is not a final effectiveness claim.

- Condition features: six rainfall-forcing variables from `rains.npy` and `event_manifest.csv`.
- Normalization: training events only.
- The condition is a full-event static rainfall-forcing summary. It includes rainfall information after each prediction window, so it is suitable for scenario simulation under known event forcing, not a history-only causal online predictor.
- Relative-change values in the group tables are arithmetic means of event-level relative changes, not a pooled prediction-step/node ratio.
- Positive relative change means the conditioned error is lower than the frozen baseline; negative values indicate degradation.

## Group summary

| grouping | split | group | event_count | node_rmse_mean | node_rmse_median | node_rmse_std | node_rmse_relative_change_mean_pct | pipe_rmse_mean | pipe_rmse_median | pipe_rmse_std | pipe_rmse_relative_change_mean_pct | flood_volume_rmse_mean | flood_volume_rmse_median | flood_volume_rmse_std | flood_volume_rmse_relative_change_mean_pct | node_mae_mean | node_mae_median | node_mae_std | node_mae_relative_change_mean_pct | pipe_mae_mean | pipe_mae_median | pipe_mae_std | pipe_mae_relative_change_mean_pct | flood_volume_mae_mean | flood_volume_mae_median | flood_volume_mae_std | flood_volume_mae_relative_change_mean_pct |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| rainfall_group | validation | large | 11 | 4.5330 | 3.6251 | 2.5851 | -5.1649 | 3.3877 | 2.7856 | 1.7491 | -3.9511 | 7.3802 | 7.9063 | 3.7592 | -52.1992 | 1.4817 | 1.3465 | 0.5213 | -20.4473 | 1.0924 | 0.9815 | 0.3633 | -23.1414 | 1.5045 | 1.2747 | 0.7456 | -22.5727 |
| rainfall_group | validation | medium | 13 | 2.3009 | 2.2009 | 0.7253 | -25.2402 | 1.8234 | 1.7531 | 0.5160 | -29.0604 | 3.7231 | 2.5445 | 2.8105 | -62.9541 | 0.9488 | 0.8481 | 0.2984 | -40.5302 | 0.7207 | 0.6769 | 0.1880 | -47.7124 | 0.7130 | 0.6056 | 0.3247 | -38.7276 |
| rainfall_group | validation | small | 3 | 1.2990 | 1.1746 | 0.2014 | -23.8453 | 1.0065 | 0.9517 | 0.1058 | -27.9320 | 1.5723 | 1.6595 | 0.3054 | -47.9352 | 0.6060 | 0.5220 | 0.1209 | -43.0838 | 0.4450 | 0.4093 | 0.0665 | -46.3718 | 0.3859 | 0.3575 | 0.0766 | -43.9905 |
| rainfall_group | heldout_test | large | 3 | 2.3723 | 2.0314 | 0.7975 | -19.7975 | 1.8567 | 1.6449 | 0.5534 | -20.8388 | 3.9505 | 2.0897 | 3.0069 | -60.1828 | 1.0210 | 0.8421 | 0.3996 | -40.2730 | 0.7464 | 0.6441 | 0.2541 | -40.4313 | 0.8083 | 0.6018 | 0.4209 | -34.6842 |
| max_intensity_group | validation | high_peak | 17 | 3.9400 | 3.2889 | 2.2548 | -15.4358 | 2.9713 | 2.5251 | 1.5333 | -13.5307 | 6.6009 | 5.8167 | 3.7531 | -64.9749 | 1.3628 | 1.3411 | 0.4807 | -31.8711 | 1.0022 | 0.9520 | 0.3319 | -33.4141 | 1.3016 | 1.1541 | 0.6853 | -35.0279 |
| max_intensity_group | validation | low_peak | 5 | 1.2756 | 1.2298 | 0.1588 | -22.6501 | 1.0072 | 0.9786 | 0.0840 | -29.8866 | 1.4088 | 1.2404 | 0.3137 | -35.3730 | 0.5875 | 0.5462 | 0.0968 | -39.3293 | 0.4439 | 0.4176 | 0.0539 | -45.9235 | 0.3666 | 0.3574 | 0.0650 | -35.0023 |
| max_intensity_group | validation | medium_peak | 5 | 2.0627 | 1.9349 | 0.3222 | -16.1628 | 1.6880 | 1.6899 | 0.2202 | -25.1174 | 3.0081 | 2.2494 | 1.3665 | -50.9925 | 0.8691 | 0.8481 | 0.1368 | -28.5220 | 0.6925 | 0.6769 | 0.1009 | -43.2550 | 0.6030 | 0.5864 | 0.1784 | -22.6488 |
| max_intensity_group | heldout_test | high_peak | 1 | 3.4738 | 3.4738 | 0.0000 | -30.4300 | 2.6150 | 2.6150 | 0.0000 | -22.1923 | 8.1923 | 8.1923 | 0.0000 | -156.2477 | 1.5747 | 1.5747 | 0.0000 | -82.2014 | 1.0960 | 1.0960 | 0.0000 | -71.3100 | 1.3951 | 1.3951 | 0.0000 | -78.7943 |
| max_intensity_group | heldout_test | medium_peak | 2 | 1.8216 | 1.8216 | 0.2098 | -14.4812 | 1.4775 | 1.4775 | 0.1674 | -20.1621 | 1.8296 | 1.8296 | 0.2601 | -12.1503 | 0.7441 | 0.7441 | 0.0979 | -19.3087 | 0.5717 | 0.5717 | 0.0724 | -24.9919 | 0.5149 | 0.5149 | 0.0868 | -12.6291 |

## Screening decision

Validation mean relative changes for the peak-intensity groups (Node/Pipe/Flood RMSE):
| group | node_rmse_relative_change_pct | pipe_rmse_relative_change_pct | flood_volume_rmse_relative_change_pct |
| --- | --- | --- | --- |
| high_peak | -15.4358 | -13.5307 | -64.9749 |
| low_peak | -22.6501 | -29.8866 | -35.3730 |
| medium_peak | -16.1628 | -25.1174 | -50.9925 |

Result: the predefined screening criterion is not met; high-peak errors did not all decrease and/or low/medium groups degraded. Do not proceed to multi-seed or pressure-test experiments from this checkpoint.
