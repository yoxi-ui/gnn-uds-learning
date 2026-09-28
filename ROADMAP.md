
# 学习与科研路线图

## 阶段一：打基础（已完成主要部分）

- [x] CS231n：线性分类器、正则化与优化、反向传播、CNN
- [x] CS224W：图基础、节点嵌入、随机游走、消息传递、GNN 层
- [x] 理解节点级、边级和图级任务
- [x] 完成 S06 论文的第一轮精读

## 阶段二：完成最小复现（已完成）

- [x] 完成 shunqing 的 SWMM 数据生成
- [x] 确认 `states.npy`、`perfs.npy`、`rains.npy` 等文件生成
- [x] 运行 2 epoch smoke test
- [x] 检查 loss 是否正常下降、模型文件是否保存

补充：已完成 500 epoch 的 GPU 训练、123 个事件测试、事件级损失汇总，以及原始物理量上的 RMSE、MAE、R² 计算。详情见 [`projects/s06-reproduction/status.md`](projects/s06-reproduction/status.md) 和 [`projects/s06-reproduction/experiment_results.md`](projects/s06-reproduction/experiment_results.md)。

## 阶段三：论文风格实验

- [x] 训练 60→60 的时空模型
- [ ] 比较 GCN/GAT 或不同 GNN 层数
- [x] 做 node-edge fusion 消融（当前记录为输出级 edge-flow fusion）
- [ ] 做 flooding 分类开关消融
- [x] 记录 RMSE、MAE、洪涝 Precision/Recall/F1

补充：已完成 148 场事件、118/27/3 划分下的冻结基线、三档训练对比、逐事件测试、总雨量与最大逐步雨强分组、bootstrap 和降雨特征—误差相关性分析。静态降雨条件拼接模型完成单 seed 筛选但未超过冻结基线。过程记录见 [`learning_logs/s06/2026-09-24_to_2026-09-28.md`](learning_logs/s06/2026-09-24_to_2026-09-28.md)，当前边界见 [`projects/s06-reproduction/status.md`](projects/s06-reproduction/status.md)。

## 阶段四：连接 AI inference（待开展）

- [ ] 测量单样本推理延迟
- [ ] 测量不同 batch size 的吞吐量
- [ ] 记录 CPU/GPU、内存/显存和模型参数量
- [ ] 尝试减少层数、hidden dimension 或序列长度
- [ ] 在保证误差可接受的情况下测试量化或蒸馏

## 阶段五：连接 DRL/MPC（待开展）

- [ ] 用 surrogate 替代部分环境 rollout
- [ ] 比较 SWMM rollout 与 surrogate rollout 的速度和误差
- [ ] 将 surrogate 接入一个简单 MPC 或 DRL 环境
- [ ] 评估控制性能、训练速度和长 rollout 稳定性

## 每周记录模板

每周至少记录：

- 本周学了什么；
- 跑通了什么；
- 遇到的错误和解决方法；
- 一组可复现的参数；
- 一个新的实验问题；
- 下一周只做哪一件最重要的事。
