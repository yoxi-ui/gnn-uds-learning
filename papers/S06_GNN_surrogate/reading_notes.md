# S06：排水管网 GNN 替代模型论文笔记

## 论文信息

Zhang et al., “Graph neural network-based surrogate modelling for real-time hydraulic prediction of urban drainage networks”, *Water Research*, 263, 122142, 2024.

## 研究问题

SWMM 等传统水力模拟器具有较好的物理解释性，但大量重复模拟会带来较高计算成本。论文训练时空 GNN surrogate，近似排水管网的动态水力响应，用于实时预测。

## 图和特征

- 105 个 manholes + 8 个 outfalls；
- 131 条 conduits；
- 节点特征：水深、节点入流、节点出流等；
- 边特征：管道水深、管道流量等；
- 空间图使用邻域关系，模型中使用 GAT 学习邻居权重。

## 输入输出

- 输入：过去 60 分钟状态和未来 60 分钟 runoff boundary；
- 输出：未来 60 分钟节点状态、边状态和 flooding 相关结果；
- 本质：学习一个从图上的历史时空状态到未来时空状态的近似动力学映射。

## 模型

- GAT：空间消息传递；
- temporal convolution：时间依赖；
- residual/skip：稳定深层训练；
- node-edge fusion：融合节点与管道状态；
- flooding determination：单独处理洪涝识别。

## 损失

- 节点状态 MSE；
- 边状态 MSE；
- flooding BCE；
- 多任务损失需要关注归一化和权重平衡。

## 关键结果

单次 60 分钟预测耗时大致为 SWMM 5.70 s、NN surrogate 0.034 s、GAT surrogate 0.064 s。论文说明 surrogate 的主要价值是将昂贵的水力模拟变成快速预测，从而支持实时控制和大量 rollout。

## 局限和可改进点

- 跨管网泛化仍需验证；
- 测试降雨数量有限；
- 尚未充分展示 DRL/MPC 闭环收益；
- 可以进一步研究量化、蒸馏、ONNX/TensorRT、批量推理和长 rollout 稳定性。

## 精读检查表

- [ ] 复现节点、边和特征定义
- [ ] 核对训练/验证/测试划分
- [ ] 复现 baseline 和指标
- [ ] 记录单样本延迟与批量吞吐量
- [ ] 做 node-edge fusion 消融
- [ ] 检查长时间 rollout 的误差累积
