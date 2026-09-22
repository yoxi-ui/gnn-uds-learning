# NN/GNN 与 edge-flow fusion 消融实验

更新时间：2026-09-22

## 1. 实验目的与口径

在已有 `GATconv + Conv1D` 与 `NN + Conv1D` 对比基础上，增加 `edge_fusion=True` 的结果，形成 `NN/GAT × edge_fusion` 的四组消融。

当前代码中的 `edge_fusion` 主要表示输出层的边流量融合：模型预测节点水深和边状态，再通过管网关联矩阵 `node_edge` 从边流量重建节点 `q_us`、`q_ds`。因此这里的对照应准确称为“edge-flow fusion / 输出级节点-边流量一致性融合”。GAT 分支内部的 `NodeEdge` 空间交互层在两种 flag 设置下仍然存在，所以这不是完全关闭所有隐藏层节点-边交互的严格架构消融。

所有模型均使用相同的 `shunqing/paper_like` 数据、98 个训练事件、相同归一化文件、`seq_in=6 -> seq_out=1`、`n_tp_layer=1`、`batch_size=64`、500 epoch 和相同测试事件。测试阶段复用了已有 `test_gpu_500` 中保存的 123 个 SWMM 事件状态，没有重新运行 SWMM。

## 2. 训练/验证阶段

下表的最佳轮次按每轮 Node loss + Edge loss 最小记录；训练耗时取 `time.npy` 最后一项。源码中的 `test_loss.npy` 在训练过程中记录的是留出集验证损失。

| 模型 | edge_fusion | 最佳 epoch | Node loss | Edge loss | 总 loss | 参数量 | 训练时间（秒） |
|---|---:|---:|---:|---:|---:|---:|---:|
| `NN + Conv1D` | False | 436 | 0.013317 | 0.000588 | 0.013905 | 588,892 | 30.07 |
| `NN + Conv1D` | True | 436 | 0.004451 | 0.000601 | 0.005053 | 559,738 | 83.65 |
| `GATconv + Conv1D` | False | 475 | 0.004167 | 0.001304 | 0.005471 | 607,670 | 98.04 |
| `GATconv + Conv1D` | True | 463 | 0.003081 | 0.002855 | 0.005936 | 607,412 | 132.02 |

## 3. 123 事件离线测试

测试覆盖 123 个事件和 92,997 个时间步。`pooled` 指按全部时间步汇总的归一化损失。

| 模型 | edge_fusion | Pooled Node loss | Pooled Edge loss | Pooled total loss |
|---|---:|---:|---:|---:|
| `NN + Conv1D` | False | 0.030643 | 0.001204 | 0.031848 |
| `NN + Conv1D` | True | 0.009912 | 0.001115 | 0.011027 |
| `GATconv + Conv1D` | False | 0.006356 | 0.001850 | 0.008206 |
| `GATconv + Conv1D` | True | 0.005461 | 0.003876 | 0.009337 |

按 pooled total loss，加入当前 edge-flow fusion 后：

- NN 从 `0.031848` 降至 `0.011027`，约下降 65.4%；节点 loss 从 `0.030643` 降至 `0.009912`。
- GAT 的节点 loss 从 `0.006356` 降至 `0.005461`，但边 loss 从 `0.001850` 升至 `0.003876`，总 loss 从 `0.008206` 升至 `0.009337`。
- 在这组数据和当前实现下，GAT 无融合的总 pooled loss 仍然最低；GAT 融合的节点预测最好，但边侧代价增加。

## 4. 反归一化物理量指标

指标顺序为 RMSE / MAE / R²。

| 类型 | 变量 | NN 无融合 | NN edge-flow fusion | GAT 无融合 | GAT edge-flow fusion |
|---|---|---:|---:|---:|---:|
| Node | `depthN` | 0.4741 / 0.1651 / 0.8467 | 0.4202 / 0.1480 / 0.8796 | 0.2960 / 0.1163 / 0.9403 | 0.2802 / 0.0949 / 0.9465 |
| Node | `q_us` | 30.5579 / 10.7296 / 0.0426 | 3.5817 / 1.3110 / 0.9868 | 7.9237 / 2.9699 / 0.9356 | 5.6984 / 2.1087 / 0.9667 |
| Node | `q_ds` | 27.2278 / 9.9126 / 0.1764 | 3.8727 / 1.4210 / 0.9833 | 8.0505 / 2.9712 / 0.9280 | 5.7255 / 2.1847 / 0.9636 |
| Edge | `depthL` | 0.0269 / 0.0145 / 0.9888 | 0.0255 / 0.0121 / 0.9899 | 0.0310 / 0.0171 / 0.9852 | 0.0424 / 0.0242 / 0.9721 |
| Edge | `volumeL` | 13.0811 / 5.3554 / 0.9929 | 13.1571 / 4.9780 / 0.9929 | 14.3306 / 6.0783 / 0.9915 | 16.5011 / 7.9908 / 0.9888 |
| Edge | `flow_vol` | 3.1893 / 1.3675 / 0.9793 | 3.2284 / 1.3507 / 0.9788 | 4.8364 / 1.9619 / 0.9524 | 5.3094 / 2.1891 / 0.9426 |

## 5. 结果解释

1. 对 NN 分支，edge-flow fusion 对节点流量输出帮助明显：节点 `q_us`、`q_ds` 的 R² 从 0.0426/0.1764 提高到 0.9868/0.9833，说明仅靠独立节点预测很难恢复流量守恒，而利用边流量和关联矩阵能显著改善物理一致性。
2. 对 GAT 分支，融合后节点水深和节点流量均改善，但边变量本身变差，说明“由边流量重建节点流量”会把边预测误差传递到节点，同时可能改变优化重点。
3. 因此当前结果支持两个结论：图拓扑对节点响应仍然重要；输出级节点-边流量一致性对 NN 尤其有效，但不应直接声称融合在所有指标上都优于无融合模型。

## 6. 结果文件

- NN 无融合模型：`model/shunqing/nn_gpu_500`；测试分析：`results/shunqing/offline_eval_nn_b32/analysis`
- NN 融合模型：`model/shunqing/nn_fusion_gpu_500`；测试分析：`results/shunqing/offline_eval_nn_fusion_b32/analysis`
- GAT 无融合模型：`model/shunqing/train_gpu_500`；测试分析：`results/shunqing/test_gpu_500/analysis`
- GAT 融合模型：`model/shunqing/gat_fusion_gpu_500`；测试分析：`results/shunqing/offline_eval_gat_fusion_b32/analysis`

每个分析目录包含 `summary.json`、`raw_metrics.csv`、`event_metrics.csv` 和代表性曲线图。模型权重、原始 `.npy` 数据和完整训练目录不应上传 GitHub。
