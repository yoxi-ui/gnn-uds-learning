# S06 复现实验结果

更新时间：2026-09-21

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

完整分析结果保存在本地源码目录：

```text
D:\论文\GNN-UDS\surrogate\results\shunqing\test_gpu_500\analysis
D:\论文\GNN-UDS\surrogate\results\shunqing\model_comparison
```

其中包括：

- 事件级损失表和汇总 JSON；
- 原始物理量 RMSE/MAE/R² 表；
- 最大、典型和最小误差事件的真实值/预测值曲线；
- 训练/验证损失对照曲线；
- `GATconv + Conv1D` 与 `GATconv + GRU` 的对照表。

为避免泄露数据和造成仓库过大，原始 `.npy` 数据、模型权重和大规模运行目录不纳入学习记录仓库。

## 6. 使用的分析脚本

脚本位于本地源码目录 `D:\论文\GNN-UDS\surrogate`：

- `analyze_test_results.py`
- `compare_training_runs.py`
- `evaluate_saved_events.py`
