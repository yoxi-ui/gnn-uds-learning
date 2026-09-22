# S06 复现状态

## 环境

- 源码目录：`GNN-UDS/surrogate`
- Conda 环境：`gnn_uds`
- Python：3.10
- TensorFlow：2.10.0
- Spektral：1.3.1
- NumPy：1.23.5
- SciPy：1.15.0
- 关键库已完成导入检查。

## 当前阶段

已完成 `shunqing` 数据生成、GPU 训练、测试分析、时间模块对照实验，以及 NN/GAT 与 edge-flow fusion 消融实验。

- 123 个降雨事件已完成测试，共 92,997 个时间步；
- 主模型为 `GATconv + Conv1D`，已完成 500 epoch 训练和测试；
- 已汇总事件级 Node/Edge loss，并定位最大误差事件；
- 已在原始物理量上计算 RMSE、MAE 和 R²；
- 已完成 `GATconv + Conv1D` 与 `GATconv + GRU` 的训练阶段对照；
- 已完成 `NN/GAT × edge_fusion` 四组对照：`NN + Conv1D`、`NN + Conv1D + edge_fusion`、`GATconv + Conv1D`、`GATconv + Conv1D + edge_fusion`；
- 四组模型均在相同 123 个事件和 92,997 个时间步上完成离线测试，未重新运行 SWMM；
- 已将汇总表、报告和曲线复制到同目录的 `results/`，便于后续提交到 GitHub；
- 详细数值、指标和下一步建议见同目录的 [`experiment_results.md`](experiment_results.md)。

## 当前实验结果摘要

| 项目 | 结果 |
|---|---:|
| 事件宏平均 Node loss | 0.006468 |
| 事件宏平均 Edge loss | 0.001940 |
| 最大误差事件 | `bpswmm_1011` |
| 最大事件总 loss | 0.021158 |
| 最佳验证模型 | `GATconv + Conv1D` |

edge-flow fusion 消融的 pooled total loss：

| 模型 | edge_fusion | pooled total loss |
|---|---:|---:|
| `NN + Conv1D` | False | 0.031848 |
| `NN + Conv1D` | True | 0.011027 |
| `GATconv + Conv1D` | False | 0.008206 |
| `GATconv + Conv1D` | True | 0.009337 |

主模型的原始物理量指标和对照实验完整表格已记录在 [`experiment_results.md`](experiment_results.md)。

## 下一步顺序

1. 将测试曲线和事件级 CSV 作为实验附件整理保存；
2. 补充 `GRU` 模型在 123 个测试事件上的完整原始物理量指标；
3. 记录单样本推理延迟、吞吐量和显存；参数量已在 edge-flow fusion 报告中记录；
4. 再尝试 60→60、3 层空间 GNN 和 flooding 配置；
5. 若要严格复现论文的隐藏层 node-edge fusion 定义，增加真正关闭 `NodeEdge` 隐藏交互的架构开关；当前 `edge_fusion` 结果属于输出级边流量融合；
6. 保持原始 `.npy` 数据、模型权重和其他大文件留在本地，不直接上传仓库。

## 重要提醒

- CUDA 缺失提示不等于程序失败，当前可以使用 CPU；
- 第一次不要直接从 20,000 epoch 开始；
- 论文配置和仓库默认配置可能不同，实验时要记录实际参数。
