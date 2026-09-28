# GNN-UDS Learning

我的深度学习、图神经网络和城市排水管网替代模型学习记录。

## 学习路线

- CS231n：深度学习与计算机视觉基础
- CS224W：图机器学习与 GNN
- S06：排水管网 GNN 替代模型论文精读与复现
- 后续：强化学习、推理加速与模型部署

## 快速回顾

- [学习总总结](notes/learning_summary.md)
- [CS231n 总结](notes/cs231n/summary.md)
- [CS224W 总结](notes/cs224w/summary.md)
- [S06 论文笔记](papers/S06_GNN_surrogate/reading_notes.md)
- [复现状态](projects/s06-reproduction/status.md)
- [2026-09-24 至 2026-09-28 实验日志](learning_logs/s06/2026-09-24_to_2026-09-28.md)
- [下一阶段路线图](ROADMAP.md)

## 当前方向

以 **GNN surrogate modelling for urban drainage networks** 为科研主线，逐步连接到强化学习控制和 AI inference。

核心问题是：用图神经网络学习排水管网的时空水力响应，在保证预测误差可接受的同时，降低传统 SWMM 模拟的计算成本，并为后续 DRL/MPC 提供快速环境。

## 当前研究状态（2026-09-28）

当前项目已经从早期最小复现和模型消融，推进到固定数据划分、冻结基线和降雨工况误差诊断阶段。

- 早期消融以 `GATConv + Conv1D` 为主线，完成 NN/GAT、输出级 `edge_fusion` 和 Conv1D/GRU 对照；这些实验使用 98/25 划分与 6→1 短序列。
- S06 风格正式基线重建 148 场事件，固定为 118/27/3，使用 60→60、`stride=60` 的 teacher-forced 测试；10,000 次更新的 `best` checkpoint 已冻结。
- 验证集显示最大逐步雨强与节点、管道和洪涝体积 RMSE 的相关性高于总雨量。
- 六维静态降雨条件拼接模型没有达到“高雨强改善且普通降雨不退化”的筛选标准，暂不作为已验证有效方法。
- 当前结论、协议限制和未完成事项见 [`projects/s06-reproduction/status.md`](projects/s06-reproduction/status.md)；按日期的过程记录见 [`learning_logs/s06/2026-09-24_to_2026-09-28.md`](learning_logs/s06/2026-09-24_to_2026-09-28.md)。
- 可上传的报告、CSV、JSON 和可视化图见 [`projects/s06-reproduction/results/s06_current`](projects/s06-reproduction/results/s06_current)。

本仓库保存学习记录、实验过程和轻量结果，不包含原始 `.npy` 数据、SWMM 输入/输出、训练权重或完整运行目录。完整源码和运行目录仍保存在本地 `GNN-UDS/surrogate` 项目中。

