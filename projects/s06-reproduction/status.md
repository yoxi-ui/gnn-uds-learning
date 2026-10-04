# S06 复现与降雨工况分析状态

更新时间：2026-10-01

本文件是 S06 学习和实验的当前状态入口。详细的按日期记录见 [`learning_logs/s06/2026-09-24_to_2026-09-28.md`](../../learning_logs/s06/2026-09-24_to_2026-09-28.md)，轻量结果附件见 [`results/s06_current`](results/s06_current)。

## 1. 研究范围与实验边界

研究对象是 Shunqing 城市排水管网的 GNN surrogate，目标是理解空间拓扑、时间模块和降雨工况对节点/管道水力状态预测的影响。

当前工作包含两个不能混合的阶段：

1. 早期模型消融：98/25 划分、6→1 短序列，主线为 `GATConv + Conv1D`，包含 NN/GAT、输出级 edge-flow fusion 和 Conv1D/GRU 对照。
2. S06 风格正式基线：根据公开信息重建 148 场事件，固定为 118/27/3，输入 60 步、预测 60 步，使用不重叠 teacher-forced 窗口，并在此基础上做降雨工况诊断。

正式基线不是 S06 原文的完全复现：当前 event-ID 划分是依据公开信息重建的，25 场 `test_bpswmm_*` 事件也没有全部单独隔离。因此报告中使用“本地复现与扩展”，不写成“完全复现 S06”。

## 2. 冻结正式基线

| 项目 | 值 |
|---|---:|
| 图规模 | 113 节点、131 管道 |
| 事件总数 | 148 |
| 划分 | 118 train / 27 validation / 3 held-out test |
| 留出事件 | `test_bpswmm_118`、`test_bpswmm_133`、`test_bpswmm_1612` |
| 模型 | `GATConv + Conv1D` |
| 最佳 checkpoint | 10,000 次更新运行的 `best` |
| 最佳更新 | 9098 |
| 最佳验证损失 | `0.0256911553` |
| 训练设备 | RTX 4050 / CUDA / cuDNN |
| batch size | 8 |
| 验证方式 | 固定 8 个验证 batch |
| 时间协议 | 60 步输入 → 60 步输出，`stride=60` |
| 测试协议 | 真实历史输入的 teacher-forced、不重叠窗口 |
| 测试规模 | 43 个窗口、2,580 个预测时间步 |

三场留出测试的总体指标：

| 指标 | 数值 |
|---|---:|
| 节点 RMSE | 1.9949 |
| 管道 RMSE | 1.5784 |
| 洪涝体积 RMSE | 2.2512 |
| 洪涝分类 F1 | 0.9920 |

洪涝分类 F1 的未四舍五入值为 `0.99196`。

归一化参数只使用 118 个训练事件计算。训练和评估代码分别保存独立运行目录、`best`/`final` checkpoint、周期 checkpoint、损失数组和运行摘要。训练时优化归一化损失；最终 RMSE、MAE、R² 等指标在反归一化的 SWMM 物理单位上计算。

## 3. 三档训练与逐事件诊断

| 更新次数 | 最佳轮次 | 最佳验证损失 | 节点 RMSE | 节点 R² | 管道 RMSE | 管道 R² | 洪涝体积 RMSE | 洪涝体积 R² | 洪涝分类 F1 |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 2,000 | 1,842 | 0.0481 | 4.8447 | 0.9722 | 3.5280 | 0.9650 | 5.7178 | 0.9847 | 0.9848 |
| 5,000 | 4,993 | 0.0352 | 2.3900 | 0.9932 | 1.9401 | 0.9894 | 4.5303 | 0.9904 | 0.9893 |
| 10,000 | 9,098 | 0.0257 | 1.9949 | 0.9953 | 1.5784 | 0.9930 | 2.2512 | 0.9976 | 0.9920 |

一次“更新”指一次随机 batch 参数更新，不等于完整遍历一次训练集。10,000 次模型在所有总体连续变量指标上最好，当前使用其 `best` 而不是 `final` checkpoint。训练结束后没有继续运行训练进程，GPU 已释放。

10,000 次模型逐事件结果：

| 事件 | 时长 | 窗口数 | 节点 RMSE | 节点 R² | 管道 RMSE | 管道 R² | 洪涝体积 RMSE | 洪涝体积 R² | 洪涝分类 F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `test_bpswmm_118` | 11 h | 12 | 1.8196 | 0.9960 | 1.4186 | 0.9942 | 1.8468 | 0.9982 | 0.9916 |
| `test_bpswmm_133` | 13 h | 14 | 2.6633 | 0.9944 | 2.1401 | 0.9914 | 3.1970 | 0.9972 | 0.9924 |
| `test_bpswmm_1612` | 16 h | 17 | 1.3738 | 0.9962 | 1.0534 | 0.9947 | 1.4121 | 0.9980 | 0.9918 |

`test_bpswmm_1612` 表现最好；`test_bpswmm_133` 连续变量误差最高。三个事件的洪涝分类 F1 差异很小，主要误差来自连续变量回归。

## 4. 降雨工况诊断

每场事件提取总雨量、最大逐步雨强、平均雨强、历时、有效降雨步数、峰值时刻、前期无雨时间、峰前雨量和峰后雨量。所有分组阈值只使用 118 个训练事件拟合。

| 特征 | 第 1 个阈值 | 第 2 个阈值 |
|---|---:|---:|
| 总雨量 | 26.6 mm | 42.2 mm |
| 最大逐步雨强 | 7.3 mm/h | 13.6 mm/h |

验证集按最大逐步雨强分组的事件级 RMSE 均值：

| 峰值雨强组 | 事件数 | 节点 RMSE | 管道 RMSE | 洪涝体积 RMSE |
|---|---:|---:|---:|---:|
| low_peak | 5 | 1.0379 | 0.7754 | 1.0311 |
| medium_peak | 5 | 1.7895 | 1.3719 | 1.9095 |
| high_peak | 17 | 3.7089 | 2.8506 | 4.5764 |

验证集 Pearson 相关系数：

| 特征 | 节点 RMSE | 管道 RMSE | 洪涝体积 RMSE |
|---|---:|---:|---:|
| 总雨量 | 0.7012 | 0.7083 | 0.6855 |
| 最大逐步雨强 | 0.9367 | 0.9494 | 0.9311 |

误差随峰值雨强分组增强而升高，且最大逐步雨强的相关性高于总雨量。删除验证集最差事件后，大雨组仍保持较高误差，因此高误差不是由单个事件完全造成。但这是事件级关联，不是因果证明。三场留出事件都没有超过训练集最大峰值雨强，不能据此宣称“未见强降雨泛化失败”。

## 5. 静态降雨条件化筛选

完成单 seed、5,000 次更新的最小条件化模型：

- 六个 forcing-only 特征：总雨量、最大逐步雨强、历时、峰值时间比例、峰前比例、峰后比例；
- 标准化参数只使用训练事件；
- 条件向量直接拼接到节点/管道解码器；
- 使用同一 118/27/3 划分、60→60 teacher-forced 协议和窗口设置；
- 特征只读取 `rains.npy`、`event_id.npy` 和 `event_manifest.csv`，没有使用水力状态、预测结果或误差指标；
- 标准化均值和标准差只由 118 个训练事件计算，训练、验证、测试没有事件交集；
- 条件输入实际写入模型，并通过 embedding 广播到节点和管道解码器；
- 相对变化定义为 `100 × (baseline - conditioned) / baseline`。

为避免把早期约 1,500 次 checkpoint 误认为最终失败，另外完成了完整单 seed 训练：最佳 epoch 为 `4876`，最佳验证损失为 `0.0281887360`。完整训练的验证集高峰值组 Node/Pipe/Flood RMSE 分别为 `3.9400`、`2.9713`、`6.6009`，相对冻结基线变化为 `-15.44%`、`-13.53%`、`-64.97%`；验证集整体事件级平均相对变化为 `-16.91%`、`-18.71%`、`-56.90%`。负值表示误差增加。

结论：完整训练后的静态整场降雨摘要直接拼接仍没有达到“高雨强改善且普通降雨不退化”的预设标准，因此将它作为经过完整训练但未通过筛选的探索性结果，不继续多 seed、FiLM、25 场 source-separated 重训练或高雨强压力测试。由于条件向量包含预测窗口之后的整场降雨信息，它适用于已知完整降雨过程的情景模拟，不应表述为 history-only 在线预测。

## 6. 早期消融状态

早期 `GATConv + Conv1D` 主线已完成以下对照，完整表格见 [`experiment_results.md`](experiment_results.md) 和 [`results/model_comparison`](results/model_comparison)：

- NN vs GAT：GAT 的节点预测明显更好，说明拓扑信息有增益；NN 更快，部分管道边指标略优。
- 输出级 `edge_fusion`：NN 开启后节点指标大幅改善；GAT 开启后节点略改善但管道误差上升，整体总损失变差。
- Conv1D vs GRU：当前设置下 Conv1D 训练损失更低、耗时更短；GRU 尚未完成全套反归一化物理指标。
- 这里的 `edge_fusion` 是输出级边流量一致性融合，不是完整的节点-边隐层双向交互。

## 7. 未完成事项

- GCN 对照实验尚未完成，当前存在维度报错。
- GRU 的完整反归一化 RMSE/MAE/R² 测试尚未补齐。
- 当前正式基线指标仍是 teacher-forced、非重叠窗口；自回归滚动已在 27 个验证事件和 3 个留出事件上独立运行，结果见本节后新增的滚动记录。
- 尚未完成 25 场 source-separated 重训练、高雨强压力测试和动态降雨 forcing encoder。
- 重叠窗口、25 场 source-separated 重训练、高雨强压力测试和动态降雨 forcing encoder 仍未完成。

原始 `.npy`、SWMM 输入/输出、模型权重和完整运行目录保留在本地 `GNN-UDS/surrogate` 项目，不上传本仓库。

## 8. 2026-09-30 自回归与速度基准

### 自回归滚动

冻结 `s06_formal_10000_b8_seed42/best`，从每个事件最初 60 步真实历史开始，之后每个 60 步块反馈模型预测的节点/管道状态。未来 `states[...,3]` lateral-inflow boundary 仍使用数据记录值，所以这是 state-feedback rollout，不是完整 history-only 闭环。

27 个验证事件的最终累计 RMSE（事件级 mean / median / P90）为：Node `3.1958 / 2.3853 / 5.4453`，Pipe `2.4907 / 1.8511 / 4.4166`，Flood volume `4.2240 / 3.2167 / 7.1305`。3 个留出事件对应为：Node `2.2210 / 2.0396 / 2.6958`，Pipe `1.7480 / 1.5946 / 2.1842`，Flood volume `2.4756 / 2.1729 / 3.1451`。验证集最高最终 Node RMSE 为 `12.5545`，说明长时误差存在事件级退化，不能只报告均值。

本地结果目录：

- `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/autoregressive_validation/`
- `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/autoregressive_heldout/`

### 速度与资源

新增 `D:/论文/GNN-UDS/surrogate/scripts/benchmark_s06.py`，测量 SWMM 端到端事件仿真、surrogate warm rollout、单样本延迟、batch 吞吐、硬件和显存。冻结模型共 `1,064,396` 个 trainable parameters。首轮 3 个留出事件的 SWMM 耗时为 `61.03–189.48 s`，surrogate rollout 为 `0.406–15.08 s`；第一条记录包含 TensorFlow 首次 trace。每个事件独立 2 次重复测量后，三场事件的稳定 warm speedup（按 forecast steps 归一化）为 `236.49x/256.06x/252.46x`，范围约 `236.5x–256.1x`。重复测量中 warm 单个 `60→60` 调用约 `14.1 ms`，batch `1/2/4/8/16` 的吞吐约 `4093/4681/4028/3691/3629` 预测步/秒；不同 batch 的第一次调用包含形状 trace，不能混入 warm 统计。

本次 benchmark 只读已有数据和 checkpoint；没有启动训练，也没有把本地权重、原始数组或 SWMM 输出加入仓库。详细过程见 [`2026-09-30_autoregressive_and_benchmark.md`](../../learning_logs/s06/2026-09-30_autoregressive_and_benchmark.md)。

## 9. 2026-09-30 反馈间隔消融

新增 `D:/论文/GNN-UDS/surrogate/scripts/feedback_interval_s06.py`。冻结同一个 `60->60` checkpoint，在 27 个验证事件上比较 K=`5/15/30/60` 分钟反馈间隔；每次只采用 60 分钟输出的前 K 帧，在边界只替换一帧观测状态，保留 K-1 个预测中间帧。四个 K 使用相同事件和相同预测时间段，未来 lateral-inflow 仍是记录值，因此这是 offline ideal-feedback diagnostic，不是 MPC 证据。

验证集事件级 Node RMSE mean/P90/max：K=5 为 `2.0467/3.0954/9.2758`，K=15 为 `2.2867/3.3095/9.7694`，K=30 为 `2.5172/3.8190/10.6273`，K=60 为 `2.8588/4.7102/11.9514`。Pipe 和 Flood volume 也呈同方向变化；K=5 的洪涝漏报率均值为 `0.00844`，每节点峰值水深幅度误差均值为 `0.0599`。K=5 单场 warm 总推理约 `1.94 s`，K=15/30/60 分别约 `0.65/0.33/0.17 s`。

按验证集主判据选择 K=5，随后只对 3 个留出事件执行一次确认：Node/Pipe/Flood volume RMSE mean 为 `1.3345/1.0183/1.7173`，洪涝 F1 mean `0.99343`，漏报率均值 `0.00684`。精确观测峰值时刻的局部 depthN 误差不随 K 单调改善，且当前未来 forcing 与 `act: false` 仍限制了控制解释。完整协议、峰值诊断和结果路径见 [`2026-09-30_feedback_interval_ablation.md`](../../learning_logs/s06/2026-09-30_feedback_interval_ablation.md)。

## 10. 2026-10-01 反馈现实性检查

在不训练新模型的前提下，继续固定 `s06_formal_10000_b8_seed42/best`、`60->60` 预测和 K=5 反馈间隔，分别检查状态观测缺失与 forcing 信息缺失。两项实验都使用同一批 27 个验证事件、同一预测时间段和初始 60 分钟真实历史；结果目录保留在本地 surrogate 项目。

### 部分观测反馈

脚本为 `D:/论文/GNN-UDS/surrogate/scripts/partial_observation_s06.py`，结果为 `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/partial_observation_validation_fixed/`。每次反馈只在边界最后一帧覆盖协议声明的通道，未观测动态通道继续使用模型预测；`setting` 是已知静态输入，不把它描述成传感器观测。初始 60 分钟历史仍完整真实，因此这是观测可得性的上界诊断，不是传感器部署模拟。

`full_state` 与前一轮 K=5 基线完全对齐（Node RMSE `2.0467`，Pipe RMSE `1.5150`，Flood-volume `2.5706`）。事件级 mean / P90 / max：

| 反馈协议 | Node RMSE | Pipe RMSE | Flood-volume RMSE | Flood F1 | 漏报率 | 最差块 Node RMSE |
|---|---:|---:|---:|---:|---:|---:|
| `full_state` | 2.0467 / 3.0954 / 9.2758 | 1.5150 / 2.3508 / 6.3265 | 2.5706 / 4.6240 / 9.5917 | 0.9919 | 0.00844 | 8.8219 / 16.8310 / 34.1355 |
| `no_pipe_state` | 18.9699 / 29.5514 / 33.6007 | 12.5570 / 19.3269 / 21.9495 | 15.3047 / 22.8324 / 30.4743 | 0.9919 | 0.00844 | 50.6949 / 75.7075 / 83.8795 |
| `node_depth_q` | 18.9701 / 29.5515 / 33.6008 | 12.5570 / 19.3269 / 21.9495 | 16.5084 / 25.7475 / 34.0244 | 0.9745 | 0.02721 | 50.6952 / 75.7075 / 83.8795 |
| `node_depth_only` | 18.9705 / 29.5522 / 33.6013 | 12.5570 / 19.3269 / 21.9495 | 18.3676 / 27.6223 / 36.3181 | 0.9623 | 0.02564 | 50.6968 / 75.7113 / 83.8823 |
| `open_loop` | 18.9769 / 29.5567 / 33.6054 | 12.5570 / 19.3269 / 21.9495 | 20.0495 / 33.5453 / 40.8919 | 0.9023 | 0.05428 | 50.7049 / 75.7199 / 83.8891 |

相对 `full_state`，去掉动态管道 `depthL/flow` 后 27/27 场 Node、Pipe、Flood 和最差块指标均恶化：Node RMSE 增加 `16.9232`，Pipe 增加 `11.0420`，Flood-volume 增加 `12.7340`，最差块 Node RMSE 增加 `41.8730`。仅节点水深反馈时峰值时间偏差绝对值均值约 `29.81` 分钟，`open_loop` 约 `49.44` 分钟。K=5 的稳定收益高度依赖动态管道状态反馈，不能解释为只要节点水深传感器就能实现的现实控制周期。

### 因果 forcing

脚本为 `D:/论文/GNN-UDS/surrogate/scripts/causal_forcing_s06.py`，结果为 `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/causal_forcing_s06_validation/`。保持完整状态反馈，只改变每个 60 分钟模型调用的 lateral-inflow boundary：`recorded_oracle` 使用记录未来值；`persistence` 在每个 K=5 边界重复 `states[block_start-1,...,3]`；`noisy_persistence` 在 persistence 上乘以固定 seed、`sigma=0.2` 的 mean-one lognormal 误差。forcing 误差按实际采用的 5 分钟前缀计算，尾部不足 60 步的 padding 不进入该指标。

| forcing 模式 | Node RMSE | Pipe RMSE | Flood-volume RMSE | 漏报率 | 最差块 Node RMSE | 采用 forcing RMSE |
|---|---:|---:|---:|---:|---:|---:|
| `recorded_oracle` | 2.0467 / 3.0954 / 9.2758 | 1.5150 / 2.3508 / 6.3265 | 2.5706 / 4.6240 / 9.5917 | 0.00844 / 0.01045 / 0.01186 | 8.8219 / 16.8310 / 34.1355 | 0 / 0 / 0 |
| `persistence` | 2.4008 / 3.8741 / 9.5621 | 1.7159 / 2.7764 / 6.4988 | 4.5413 / 8.8245 / 16.4385 | 0.00995 / 0.01192 / 0.01403 | 10.1619 / 22.7454 / 34.1384 | 3.7890 / 7.9069 / 13.4614 |
| `noisy_persistence` | 2.4142 / 3.8778 / 9.5627 | 1.7252 / 2.7817 / 6.4999 | 9.3641 / 20.5813 / 32.0021 | 0.01030 / 0.01232 / 0.01417 | 10.1831 / 22.6428 / 34.1574 | 10.0712 / 21.3469 / 31.4518 |

相对 `recorded_oracle`，`persistence` 的 Node RMSE 平均增加 `0.3541`、Pipe 增加 `0.2009`、Flood-volume 增加 `1.9706`，Node 指标 27/27 场变差；`noisy_persistence` 的对应增量为 `0.3675`、`0.2102`、`6.7934`，Flood-volume 在 27/27 场变差。K=5 的收益对记录未来 forcing 有明显依赖；这不是实际降雨预报模型的结果，只是 history-only forcing 代理的敏感性检查。

### 留出事件确认（n=3，仅方向性）

为检查 forcing 敏感性的方向是否延续到留出事件，在同一冻结模型、K=5 和完整状态反馈协议下追加 3 个 held-out 事件。结果目录为 C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/causal_forcing_heldout_confirm/。

| forcing 模式 | Node RMSE | Pipe RMSE | Flood-volume RMSE |
|---|---:|---:|---:|
| recorded_oracle | 1.3345 | 1.0183 | 1.7173 |
| persistence | 1.6025 | 1.1712 | 2.8465 |
| noisy_persistence | 1.6168 | 1.1824 | 6.6327 |

相对 recorded_oracle，persistence 的 Node/Pipe/Flood-volume 增量为 +0.2680 / +0.1529 / +1.1292，3/3 个事件均变差；noisy_persistence 的增量为 +0.2823 / +0.1641 / +4.9155，同样 3/3 个事件均变差。该结果与 27 场验证集方向一致，但 n=3 仅作为留出方向确认，不报告泛化置信区间，也不把它表述为实际降雨预报性能。

### 当前决策边界

- K=5 继续定义为“记录未来 lateral-inflow、完整节点/管道状态观测和冻结 `60->60` 模型下的离线理想反馈候选”，不称为真实部署中的最优控制周期。
- 部分观测与因果 forcing 均显示，信息协议一旦接近现实，滚动误差和尾部风险会显著上升；当前 checkpoint 只适合预测侧误差诊断，不支撑 MPC/DRL 控制结论。
- Shunqing 配置仍为 `act: false`，冻结模型没有学习闸门/泵动作响应。下一阶段必须先确认可控设施，生成按事件隔离的动作条件化 SWMM 轨迹，并通过相同初态、相同 forcing、单动作扰动的成对响应测试；在此之前不接入 MPC。

### 2026-10-01 统计收尾与 Hague pilot

已新增 `D:/论文/GNN-UDS/surrogate/scripts/summarize_feedback_realism.py`，对部分观测和因果 forcing 的 27 场 `split=validation` 事件执行事件级 paired bootstrap（seed `20261001`，10,000 次），并同时输出事件宏平均与按样本计数加权的 pooled 指标。split 判断只使用显式 `split` 字段，不使用混合的 `source_set`。结果位于 `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/feedback_realism_summary/`。

统计收尾仍支持原结论：去掉动态管道状态的 Node RMSE 增量为 `+16.9232`（宏平均 95% CI `[14.3204, 19.4746]`，27/27 场变差）；`persistence` 相对 `recorded_oracle` 的 Node RMSE 增量为 `+0.3541`（`[0.2714, 0.4430]`），Flood-volume 增量为 `+1.9706`（`[1.3237, 2.7157]`）；固定 `sigma=0.2` noisy sensitivity 的对应增量为 `+0.3675`（`[0.2836, 0.4566]`）和 `+6.7934`（`[4.6754, 9.0921]`）。这些 forcing 模式仍只作为敏感性检查，不代表真实降雨预报分布。

首个动作环境选定 Hague。新增 `D:/论文/GNN-UDS/surrogate/scripts/hague_action_pilot.py`，在原始 `hague.inp` 的临时副本上运行 9 条轨迹（默认、单执行器低/高扰动、2 条离散随机安全轨迹），控制周期 5 分钟、每条 6 步。结果位于 `C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/hague_action_pilot/`。SWMM 实测 settings 与命令完全一致（最大绝对误差 0），说明动作写入链路通过；起始块的 `weir1` 扰动出现了状态/洪涝响应，`R1/R3` 需在更多时段和事件中继续审计。SWMM 自适应 routing step 会让 300 秒推进后的记录时间出现数秒偏移，因此首块 signed delta 只作响应诊断，不作为严格同一时刻的因果估计。该 pilot 不是 action-conditioned surrogate 或 MPC 结果，下一步仍是按事件隔离生成动作数据，再做成对动作响应验证。

### 11. 2026-10-01 Hague 正式成对响应协议

上一节的单场景 pilot 已由正式的多事件协议取代。新增脚本 `D:/论文/GNN-UDS/surrogate/scripts/hague_action_response_protocol.py`，结果目录为 `C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/hague_action_response_protocol_final3/`。协议在原始 `hague.inp` 的临时副本上运行，未修改原始网络、数据或权重。

Hague 的 `hg_train50_events.csv` 没有原生 train/validation/test 标签；本次不能按官方数据集 split 表述。固定 `seed=20261001` 后，48 场 eligible 事件确定性分为 `pilot_validation=8`、`pilot_test=10`、`pilot_train=30`，另有 2 场被持续时间/降雨/日期过滤排除。当前正式 gate 只使用 8 场 `pilot_validation` 事件。每场运行 9 条轨迹（baseline、三个执行器分别低/高阶跃、`all_low`、`all_high`），5 分钟控制间隔，阶跃保持 15 分钟并恢复 15 分钟，共 72 条轨迹；高低动作在共同 simulation elapsed-time 网格上对齐。

动作写入正确：72 条轨迹的最大 `|setting-command|=0`。主 gate 为 hold 窗口 `absolute_flow` 的有效响应和方向一致率至少 `0.90`，并报告 10,000 次事件级 paired bootstrap 95% CI：

| 执行器 | 有效事件 | 方向一致率 | 95% CI | 平均高低动作流量差 | gate |
|---|---:|---:|---:|---:|---:|
| `R1` | 0/8 | NA | [NA, NA] | 0.0000 | 未通过 |
| `R3` | 0/8 | NA | [NA, NA] | 0.0000 | 未通过 |
| `weir1` | 8/8 | 1.000 | [1.000, 1.000] | 0.5723 | 通过 |

源码审计显示 `R1/R3` 是 `ORIFICES` 且 `FlapGate=false`（`Gated NO`）；当前配置下命令虽被读回，但正式 8 场协议没有有效流量响应。`weir1` 是 `WEIRS`，平均高低动作流量差 `0.572314`，95% CI `[0.445004, 0.715489]`。因此当前 Hague 版本只把 `weir1` 作为通过 SWMM 响应 gate 的执行器；`R1/R3` 在物理配置或控制含义确认前不进入 action-conditioned surrogate 训练。该结论限定于当前配置和协议，不能外推成 R1/R3 在任何工况都无效。

这仍是 SWMM 动作写入/响应审计，不是 action-conditioned surrogate 验证，也不是 MPC 结果。下一步应先和导师确认 R1/R3 是否需要修正，或以 `weir1` 做单执行器小规模动作数据；随后才可按降雨事件隔离生成 `act=true` 数据、验证动作差分响应和隐藏管道状态估计。未通过这些门槛前不训练完整控制模型、不接入 MPC；Astlingen保留为第二网络或 Hague 物理配置无法修正时的备用方案。
