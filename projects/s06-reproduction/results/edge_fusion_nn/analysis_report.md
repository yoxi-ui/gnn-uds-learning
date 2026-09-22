# GNN-UDS 测试结果分析

- 事件数：123
- 总时间步：92997
- 事件宏平均 Node loss：0.010385
- 事件宏平均 Edge loss：0.001185
- 按全部时间步汇总 Node loss：0.009912
- 按全部时间步汇总 Edge loss：0.001115
- 误差最大事件：`bpswmm_113`（总 loss=0.049194）
- 误差最小事件：`bpswmm_65`（总 loss=0.001785）

## 原始物理量指标

详见 `raw_metrics.csv`。Node 的前三个输出为 `depthN`、`q_us`、`q_ds`；Edge 的前三个输出为 `depthL`、`volumeL`、`flow_vol`。

## 图表

- `top20_event_loss.png`：误差最大的 20 个事件。
- `curves_worst_*.png`：最差事件真实值/预测值曲线。
- `curves_median_*.png`：中位事件真实值/预测值曲线。
- `curves_best_*.png`：最佳事件真实值/预测值曲线。
