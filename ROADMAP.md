
# 学习与科研路线图

## 阶段一：打基础（已完成主要部分）

- [x] CS231n：线性分类器、正则化与优化、反向传播、CNN
- [x] CS224W：图基础、节点嵌入、随机游走、消息传递、GNN 层
- [x] 理解节点级、边级和图级任务
- [x] 完成 S06 论文的第一轮精读

## 阶段二：完成最小复现（当前）

- [ ] 完成 shunqing 的 SWMM 数据生成
- [ ] 确认 `states.npy`、`perfs.npy`、`rains.npy` 等文件生成
- [ ] 运行 2 epoch smoke test
- [ ] 检查 loss 是否正常下降、模型文件是否保存

## 阶段三：论文风格实验

- [ ] 训练 60→60 的时空模型
- [ ] 比较 GCN/GAT 或不同 GNN 层数
- [ ] 做 node-edge fusion 消融
- [ ] 做 flooding 分类开关消融
- [ ] 记录 RMSE、MAE、洪涝 Precision/Recall/F1

## 阶段四：连接 AI inference

- [ ] 测量单样本推理延迟
- [ ] 测量不同 batch size 的吞吐量
- [ ] 记录 CPU/GPU、内存/显存和模型参数量
- [ ] 尝试减少层数、hidden dimension 或序列长度
- [ ] 在保证误差可接受的情况下测试量化或蒸馏

## 阶段五：连接 DRL/MPC

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
