# GNN-UDS 模型对照实验

实验保持 `env`、数据目录、随机种子、训练事件划分、GAT 空间层、序列长度、batch size 和 epoch 数一致，只改变时间模块。

| 模型 | 最佳 epoch | 最佳验证总 loss | Node | Edge | 训练时间(s) |
|---|---:|---:|---:|---:|---:|
| GATconv+Conv1D | 474 | 0.005471 | 0.004167 | 0.001304 | 98.04 |
| GATconv+GRU | 483 | 0.008395 | 0.005773 | 0.002622 | 170.11 |

图文件：`validation_total_loss.png`、`validation_node_loss.png`、`validation_edge_loss.png`。
完整数值见 `training_comparison.csv`。
