# S06 复现实验结果

更新时间：2026-09-21

> 本文件保留早期 98/25 划分、6→1 短序列的模型消融结果，作为历史阶段记录。最新的 148 场、118/27/3、60→60 正式基线与降雨工况分析见 [`status.md`](status.md) 和 [`../../learning_logs/s06/2026-09-24_to_2026-09-28.md`](../../learning_logs/s06/2026-09-24_to_2026-09-28.md)。两套协议的绝对指标不直接比较。

## 1. 测试规模

- 环境：`shunqing`
- 测试事件数：123
- 总时间步：92,997
- 主模型：`GATconv + Conv1D`
- 训练轮数：500 epoch

## 2. 事件级损失

| 指标 | 数值 |
|---|---:|
| 事件宏平均 Node loss | 0.006468 |
| 事件宏平均 Edge loss | 0.001940 |
| 按全部时间步汇总 Node loss | 0.006356 |
| 按全部时间步汇总 Edge loss | 0.001850 |
| 总 pooled loss | 0.008206 |

误差最大的事件为 `bpswmm_1011`：Node loss 为 `0.015728`，Edge loss 为 `0.005430`，总 loss 为 `0.021158`。

## 3. 原始物理量指标

以下指标是在反归一化后的物理量上计算得到的。

| 类型 | 物理量 | RMSE | MAE | R² |
|---|---|---:|---:|---:|
| Node | `depthN` | 0.2960 | 0.1163 | 0.9403 |
| Node | `q_us` | 7.9237 | 2.9699 | 0.9356 |
| Node | `q_ds` | 8.0505 | 2.9712 | 0.9280 |
| Edge | `depthL` | 0.0310 | 0.0171 | 0.9852 |
| Edge | `volumeL` | 14.3306 | 6.0783 | 0.9915 |
| Edge | `flow_vol` | 4.8364 | 1.9619 | 0.9524 |

## 4. 时间模块对照实验

两组实验使用相同的环境、数据划分、随机种子、空间 GNN、序列长度、batch size 和 epoch 数，仅改变时间模块。

| 模型 | 最佳 epoch | 最佳验证总 loss | Node loss | Edge loss | 训练时间（秒） |
|---|---:|---:|---:|---:|---:|
| `GATconv + Conv1D` | 474 | 0.005471 | 0.004167 | 0.001304 | 98.04 |
| `GATconv + GRU` | 483 | 0.008395 | 0.005773 | 0.002622 | 170.11 |

在当前参数设置下，`GATconv + Conv1D` 的验证误差更低，训练时间也更短。这里的对照结论属于训练/验证阶段结论；`GRU` 的 123 个测试事件原始物理量指标仍需单独补齐后再做最终测试集比较。

## 5. 本地结果文件

完整分析结果来源于本地源码目录：

```text
D:\论文\GNN-UDS\surrogate\results\shunqing\test_gpu_500\analysis
D:\论文\GNN-UDS\surrogate\results\shunqing\model_comparison
```

适合纳入 GitHub 的轻量结果已经复制到本项目：

- 测试分析：[results/test_gpu_500/analysis](results/test_gpu_500/analysis)
- 模型对照：[results/model_comparison](results/model_comparison)

其中包括：

- 事件级损失表和汇总 JSON；
- 原始物理量 RMSE/MAE/R² 表；
- 最大、典型和最小误差事件的真实值/预测值曲线；
- 训练/验证损失对照曲线；
- `GATconv + Conv1D` 与 `GATconv + GRU` 的对照表。

为避免泄露数据和造成仓库过大，原始 `.npy` 数据、模型权重和大规模运行目录不纳入学习记录仓库；当前 `results/` 目录只包含汇总表、报告和绘图文件。

## 6. 使用的分析脚本

脚本位于本地源码目录 `D:\论文\GNN-UDS\surrogate`：

- `analyze_test_results.py`
- `compare_training_runs.py`
- `evaluate_saved_events.py`

## 7. NN/GAT 与 edge-flow fusion 消融

为分析空间拓扑和节点-边流量一致性的作用，补充了四组 `Conv1D` 模型。除 `conv` 和 `edge_fusion` 外，数据、事件划分、归一化、序列窗口、batch size 和训练轮数均保持一致。

| 模型 | edge_fusion | 最佳 epoch | 验证 Node loss | 验证 Edge loss | 验证总 loss | 参数量 | 训练时间（秒） |
|---|---:|---:|---:|---:|---:|---:|---:|
| `NN + Conv1D` | False | 436 | 0.013317 | 0.000588 | 0.013905 | 588,892 | 30.07 |
| `NN + Conv1D` | True | 436 | 0.004451 | 0.000601 | 0.005053 | 559,738 | 83.65 |
| `GATconv + Conv1D` | False | 475 | 0.004167 | 0.001304 | 0.005471 | 607,670 | 98.04 |
| `GATconv + Conv1D` | True | 463 | 0.003081 | 0.002855 | 0.005936 | 607,412 | 132.02 |

123 事件离线测试的 pooled loss：

| 模型 | edge_fusion | Node loss | Edge loss | Total loss |
|---|---:|---:|---:|---:|
| `NN + Conv1D` | False | 0.030643 | 0.001204 | 0.031848 |
| `NN + Conv1D` | True | 0.009912 | 0.001115 | 0.011027 |
| `GATconv + Conv1D` | False | 0.006356 | 0.001850 | 0.008206 |
| `GATconv + Conv1D` | True | 0.005461 | 0.003876 | 0.009337 |

其中 `edge_fusion=True` 在当前源码中表示输出级 edge-flow fusion：用预测的边流量和管网关联矩阵重建节点 `q_us`、`q_ds`。它不是完全关闭或打开所有隐藏层的 `NodeEdge` 交互，因此报告中使用“edge-flow fusion / 输出级节点-边流量一致性融合”的表述。

详细四模型对照见 [`results/model_comparison/edge_fusion_comparison.md`](results/model_comparison/edge_fusion_comparison.md)，测试汇总见 [`results/model_comparison/edge_fusion_test_metrics.csv`](results/model_comparison/edge_fusion_test_metrics.csv)。融合模型的原始物理量指标和曲线位于：

- [`results/edge_fusion_nn`](results/edge_fusion_nn)
- [`results/edge_fusion_gat`](results/edge_fusion_gat)
