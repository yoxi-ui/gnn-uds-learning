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
- [下一阶段路线图](ROADMAP.md)

## 当前方向

以 **GNN surrogate modelling for urban drainage networks** 为科研主线，逐步连接到强化学习控制和 AI inference。

核心问题是：用图神经网络学习排水管网的时空水力响应，在保证预测误差可接受的同时，降低传统 SWMM 模拟的计算成本，并为后续 DRL/MPC 提供快速环境。

