
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

## 阶段四：连接 AI inference（部分完成）

- [x] 测量单样本推理延迟
- [x] 测量不同 batch size 的吞吐量
- [x] 记录 CPU/GPU、内存/显存和模型参数量
- [ ] 尝试减少层数、hidden dimension 或序列长度
- [ ] 在保证误差可接受的情况下测试量化或蒸馏

## 阶段五：连接 DRL/MPC（部分完成，控制原型待开展）

- [ ] 用 surrogate 替代部分环境 rollout
- [x] 比较 SWMM rollout 与 surrogate rollout 的速度和误差
- [x] 在 27 场验证事件上完成 5/15/30/60 分钟反馈间隔消融，并用 K=5 做 3 场留出确认
- [x] 完成 K=5 的部分观测与因果 forcing 现实性检查（仅作离线敏感性诊断）
- [ ] 将 surrogate 接入一个简单 MPC 或 DRL 环境
- [ ] 评估控制性能、训练速度和长 rollout 稳定性

补充：已完成冻结 S06 surrogate 的长时自回归评估、本地速度基准、反馈间隔消融，以及 K=5 的部分观测和因果 forcing 现实性检查。验证集按主安全指标选择 K=5，并在 3 场留出事件上确认；现实性检查显示，去掉动态管道状态反馈或将记录 forcing 换成 persistence/noisy persistence 后，误差和尾部风险明显上升。因此 K=5 只定义为记录未来 lateral-inflow、完整节点/管道状态观测和冻结模型下的离线理想反馈候选，不能直接视为部署周期或完整 history-only 控制环境。进入 MPC 前仍需先解决动作接口并验证动作响应。

2026-10-01 补充：已完成 K=5 的反馈现实性上界检查。27 场验证事件的部分观测实验表明，去掉动态管道 `depthL/flow` 后 Node RMSE 从 `2.0467` 升至 `18.9699`，27/27 场变差；仅节点水深反馈时 Flood F1 降至 `0.9623`，open-loop 降至 `0.9023`，峰值时间偏差均值约 `49.44` 分钟。因果 forcing 实验表明，使用上一帧 lateral-inflow 的 persistence 使 Node RMSE 平均增加 `0.3541`，27/27 场相对记录未来 forcing 变差；带 `sigma=0.2` 噪声时 Flood-volume RMSE 平均增加 `6.7934`。因此 K=5 只能作为记录未来 forcing 与完整状态观测下的离线诊断候选，不能支撑部署或 MPC 结论。下一优先级改为确认可控设施、生成 `act=true` 的动作条件化 SWMM 数据并做成对动作响应验证；GCN/GRU、量化和蒸馏继续后置。

2026-10-01 统计与动作接口补充：新增 `summarize_feedback_realism.py`，完成 27 场 `split=validation` 事件的 paired bootstrap CI、事件宏平均和 pooled 指标；结果目录为 `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/feedback_realism_summary/`。新增 Hague 小型动作 pilot，确认 `R1/R3/weir1` 三个执行器的命令可以被 SWMM 正确读回，结果为接口审计而非 surrogate/MPC 证据。下一阶段固定顺序为：按事件隔离生成 Hague 动作条件化轨迹，做成对 action-response 验证，补充隐藏管道状态估计，再实现最小 MPC。

2026-10-01 正式 Hague action-response 补充：新增 `D:/论文/GNN-UDS/surrogate/scripts/hague_action_response_protocol.py`，在固定 `seed=20261001` 的 8 场 `pilot_validation` 事件上运行 72 条成对轨迹；Hague 没有原生 train/validation/test 标签，因此该名称只表示本次确定性协议 split。5 分钟控制间隔、15 分钟保持与恢复，按 simulation elapsed timestamp 对齐；setting 写入最大误差为 0。`weir1` 的 absolute-flow 方向一致率为 `1.000`（95% CI `[1.000, 1.000]`，平均高低动作差 `0.5723`），通过预注册 0.90 gate；`R1/R3` 在 8 场中均无有效流量响应，暂不进入 action-conditioned surrogate 训练。正式结果目录为 `C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/hague_action_response_protocol_final3/`。下一步不是直接生成完整训练集，而是先与导师确认 R1/R3 物理配置，或用 `weir1` 做单执行器 pilot；通过 action-response 与隐藏状态估计门槛后再接最小 MPC，Astlingen保留为备用/第二网络。

## 每周记录模板

每周至少记录：

- 本周学了什么；
- 跑通了什么；
- 遇到的错误和解决方法；
- 一组可复现的参数；
- 一个新的实验问题；
- 下一周只做哪一件最重要的事。
