# 2026-10-01：反馈现实性检查

## 目标与冻结条件

本轮不训练新模型，也不接入 MPC。冻结 `s06_formal_10000_b8_seed42/best`、`60->60` 输入输出、K=5 反馈间隔和 27 个验证事件。每场事件从同一段 60 分钟真实历史开始，预测时间段和事件配对关系保持一致。

本轮只回答两个问题：

1. K=5 的滚动收益是否依赖完整节点/管道状态观测？
2. K=5 的滚动收益是否依赖记录中的未来 lateral-inflow？

这两项都是离线信息上界/敏感性检查，不是传感器部署、降雨预报或控制性能证明。

## 实验一：部分观测反馈

脚本：`D:/论文/GNN-UDS/surrogate/scripts/partial_observation_s06.py`

结果：`D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/partial_observation_validation_fixed/`

每次反馈只在边界最后一帧覆盖协议声明的通道；未观测动态通道继续使用模型预测。节点通道为 `depthN`、`q_us`、`q_ds`、flood；管道通道为 `depthL`、flow 和已知静态 `setting`。`setting` 不被描述成传感器数据。初始 60 分钟历史完整真实，因此该实验偏向有利于反馈的上界。

| 协议 | Node RMSE mean/P90/max | Pipe RMSE mean | Flood-volume RMSE mean | Flood F1 | 漏报率 | worst-block Node mean/max |
|---|---:|---:|---:|---:|---:|---:|
| `full_state` | 2.0467 / 3.0954 / 9.2758 | 1.5150 | 2.5706 | 0.9919 | 0.00844 | 8.8219 / 34.1355 |
| `no_pipe_state` | 18.9699 / 29.5514 / 33.6007 | 12.5570 | 15.3047 | 0.9919 | 0.00844 | 50.6949 / 83.8795 |
| `node_depth_q` | 18.9701 / 29.5515 / 33.6008 | 12.5570 | 16.5084 | 0.9745 | 0.02721 | 50.6952 / 83.8795 |
| `node_depth_only` | 18.9705 / 29.5522 / 33.6013 | 12.5570 | 18.3676 | 0.9623 | 0.02564 | 50.6968 / 83.8823 |
| `open_loop` | 18.9769 / 29.5567 / 33.6054 | 12.5570 | 20.0495 | 0.9023 | 0.05428 | 50.7049 / 83.8891 |

`full_state` 与之前反馈间隔实验的 K=5 结果完全对齐。相对 `full_state`，去掉动态管道 `depthL/flow` 后 27/27 场 Node、Pipe、Flood 和 worst-block 指标均恶化：Node +16.9232、Pipe +11.0420、Flood-volume +12.7340、worst-block Node +41.8730。仅节点水深反馈时峰值时间偏差绝对值均值约 29.81 分钟；open-loop 约 49.44 分钟。

结论：K=5 的稳定收益高度依赖动态管道状态反馈，不能解释为仅凭节点水深传感器即可实现的现实控制周期。

## 实验二：因果 forcing

脚本：`D:/论文/GNN-UDS/surrogate/scripts/causal_forcing_s06.py`

结果：`D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/causal_forcing_s06_validation/`

保持完整状态反馈，只改变每个 60 分钟模型调用的 forcing。`recorded_oracle` 使用记录未来值；`persistence` 在每个 K=5 边界重复 `states[block_start-1,...,3]`；`noisy_persistence` 在 persistence 上乘以固定 seed、`sigma=0.2` 的 mean-one lognormal 误差。forcing 误差只在实际采用的 5 分钟前缀上计算。

| 模式 | Node RMSE mean/P90/max | Pipe RMSE mean | Flood-volume RMSE mean | 漏报率 mean | worst-block Node mean | adopted forcing RMSE mean |
|---|---:|---:|---:|---:|---:|---:|
| `recorded_oracle` | 2.0467 / 3.0954 / 9.2758 | 1.5150 | 2.5706 | 0.00844 | 8.8219 | 0 |
| `persistence` | 2.4008 / 3.8741 / 9.5621 | 1.7159 | 4.5413 | 0.00995 | 10.1619 | 3.7890 |
| `noisy_persistence` | 2.4142 / 3.8778 / 9.5627 | 1.7252 | 9.3641 | 0.01030 | 10.1831 | 10.0712 |

相对 `recorded_oracle`，persistence 的 Node RMSE 平均增加 0.3541、Pipe 增加 0.2009、Flood-volume 增加 1.9706，Node 指标 27/27 场变差；noisy persistence 的对应增量为 0.3675、0.2102、6.7934，Flood-volume 27/27 场变差。forcing 的峰值深度最大值指标受模型 hmax/clipping 约束，不用作本轮主要结论。

结论：K=5 的收益对记录未来 forcing 有明显依赖；persistence 只是简单的 history-only 代理，不是实际降雨预报模型。

### 留出事件确认（n=3，仅方向性）

为检查上述 forcing 敏感性是否在留出事件上保持方向，使用同一冻结模型、K=5、完整状态反馈和相同的 forcing 协议，对 3 个 held-out 事件追加运行。结果目录为 C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/causal_forcing_heldout_confirm/。

| 模式 | Node RMSE | Pipe RMSE | Flood-volume RMSE |
|---|---:|---:|---:|
| recorded_oracle | 1.3345 | 1.0183 | 1.7173 |
| persistence | 1.6025 | 1.1712 | 2.8465 |
| noisy_persistence | 1.6168 | 1.1824 | 6.6327 |

相对 recorded_oracle，persistence 的 Node/Pipe/Flood-volume 增量为 +0.2680 / +0.1529 / +1.1292，3/3 个事件均变差；noisy_persistence 的增量为 +0.2823 / +0.1641 / +4.9155，同样 3/3 个事件均变差。该结果与 27 场验证集的方向一致，但 n=3 仅作留出方向确认，不报告泛化置信区间，也不把它解释为实际降雨预报性能。

## 统计收尾：paired bootstrap 与 pooled 指标

脚本：`D:/论文/GNN-UDS/surrogate/scripts/summarize_feedback_realism.py`

结果：`D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/feedback_realism_summary/`

统计口径固定为 27 个 `split=validation` 事件，事件配对键优先使用 `event_id`。`source_set` 不用于解释训练/测试关系。宏平均直接对事件级指标平均；pooled RMSE 使用各事件 `count × RMSE²` 汇总，pooled Flood precision/recall/F1/miss rate 使用事件级 `TP/TN/FP/FN` 汇总。95% 区间为事件层 paired bootstrap，固定 `seed=20261001`、10,000 次重采样。

关键结果（括号内为宏平均 delta 的 95% CI；pooled delta 同步写入 `summary.csv`）：

| 比较 | Node RMSE delta | Pipe RMSE delta | Flood-volume RMSE delta | Flood miss-rate delta | worst-block Node delta |
|---|---:|---:|---:|---:|---:|
| `no_pipe_state - full_state` | +16.9232 `[14.3204, 19.4746]` | +11.0420 `[9.3309, 12.6901]` | +12.7340 `[10.8264, 14.5413]` | 0 | +41.8730 `[35.5467, 47.8511]` |
| `persistence - recorded_oracle` | +0.3541 `[0.2714, 0.4430]` | +0.2009 `[0.1543, 0.2508]` | +1.9706 `[1.3237, 2.7157]` | +0.00151 `[0.00119, 0.00186]` | +1.3400 `[0.7751, 2.0256]` |
| `noisy_persistence - recorded_oracle` | +0.3675 `[0.2836, 0.4566]` | +0.2102 `[0.1641, 0.2590]` | +6.7934 `[4.6754, 9.0921]` | +0.00186 `[0.00149, 0.00224]` | +1.3612 `[0.8048, 2.0338]` |

`no_pipe_state` 的 Node/Pipe/Flood/worst-block 误差均为 27/27 场变差。`persistence` 的 Node/Pipe 均为 27/27 场变差，Flood-volume 为 26/27 场变差；`noisy_persistence` 的 Node/Pipe/Flood-volume/worst-block 均为 27/27 场变差。Flood F1 的 pooled delta 为 `-0.001254`（persistence）和 `-0.001290`（noisy persistence）；它是分类敏感性指标，不替代洪涝体积误差。完整宏平均、pooled 值和 CI 见结果目录中的 `metrics.json` 与 `summary.csv`。

## Hague 动作接口 pilot

配置：`D:/论文/GNN-UDS/surrogate/envs/config/hague.yaml`（`act: true`，`R1/R3/weir1`，每个动作档位 `0/0.5/1.0`）。

脚本：`D:/论文/GNN-UDS/surrogate/scripts/hague_action_pilot.py`

结果：`C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/hague_action_pilot/`。pilot 在原始输入的副本上运行，不覆盖网络文件；使用内嵌 `1yr12hr` 雨量序列，并将临时副本的 rain-gage 间隔对齐到该序列的 6 分钟采样。9 条轨迹（默认、3 个执行器的单动作低/高扰动、2 条固定 seed 的离散随机安全轨迹），每条 6 个 5 分钟块。

- 三个执行器的命令都被 SWMM 读回，最大 `|setting-command|=0`；动作写入链路通过。
- 在事件起始 5 分钟块，`weir1` 扰动已改变相关流量、节点水头和系统累计洪涝；`R1/R3` 在该起始块未出现可见差异，不能据此判定其全事件无效。
- SWMM 的自适应 routing step 使 300 秒推进后的记录时间出现约数秒偏移（例如 `00:05:02`）；首块 signed delta 只作接口/响应诊断，不能当作严格同一时刻的因果效应估计。
- 该 pilot 只验证动作接口和初步 SWMM 响应，不是 action-conditioned surrogate 验证，也不是 MPC 结果。下一步仍需按降雨事件生成 train/validation/test 轨迹，并做相同初态、相同 forcing、单动作扰动的成对响应测试。

## 2026-10-01 正式 Hague 成对 SWMM action-response protocol

为替代单场景接口 pilot，新增 `D:/论文/GNN-UDS/surrogate/scripts/hague_action_response_protocol.py`，在原始 `hague.inp` 的临时副本上运行正式的成对动作响应审计。结果目录为 `C:/Users/sztkk/Documents/Codex/2026-09-30/codex-threads-01a0e751-5b65-78f2-a286/outputs/hague_action_response_protocol_final3/`；其中保存 `protocol.json`、`pilot_event_manifest.csv`、`action_trace.csv`、逐轨迹 `states/perfs/rainfall/actual_settings`、`response_summary.csv`、`bootstrap_summary.csv` 和 `report.md`。该重跑已完成，未修改原始 `.inp`、原始数据或模型权重。

Hague 仓库的 `hg_train50_events.csv` 没有原生 train/validation/test 标签，因此不能把这次划分写成数据集官方 split。协议固定 `seed=20261001`，按事件确定性划分为 `pilot_validation=8`、`pilot_test=10`、`pilot_train=30`，另有 2 场因持续时间/降雨/日期过滤排除，共 48 场 eligible 事件。正式结果只对 8 场 `pilot_validation` 事件作 gate 判断。

每场事件使用同一初态和同一 forcing，运行 9 条轨迹：baseline、R1/R3/weir1 各自的低档和高档阶跃、`all_low` 与 `all_high` 组合；动作档位为 `0/0.5/1.0`，baseline 为 `[0.5, 0.5, 0.5]`。控制间隔为 5 分钟，单执行器阶跃保持 15 分钟后恢复 15 分钟，共 72 条轨迹。高低动作的差异按 simulation elapsed timestamp 插值到共同 5 分钟网格，不能按 SWMM 自适应 routing step 产生的数组行号配对。

动作写入审计通过：72 条轨迹中 SWMM 读回的实际 setting 与命令最大绝对误差为 `0`。预注册主指标为 hold 窗口内的 `absolute_flow`，有效响应需超过协议 epsilon；方向门槛为事件级方向一致率至少 `0.90`，并同时报告事件级 paired bootstrap（`10,000` 次，`seed=20261001`）95% CI。

| 执行器 | 有效事件 | 方向一致率 | 95% CI | 高低动作平均流量差 | gate |
|---|---:|---:|---:|---:|---:|
| `R1` | 0/8 | NA | [NA, NA] | 0.0000 | 未通过 |
| `R3` | 0/8 | NA | [NA, NA] | 0.0000 | 未通过 |
| `weir1` | 8/8 | 1.000 | [1.000, 1.000] | 0.5723 | 通过 |

源码审计显示 `R1`、`R3` 是 `ORIFICES`，且 `FlapGate=false`（输入中为 `Gated NO`）；在当前 Hague 配置和正式协议的 8 场事件中，虽能正确写入 setting，但没有达到 epsilon 的有效流量响应。`weir1` 是 `WEIRS`，高低动作的平均流量差为 `0.572314`，bootstrap 95% CI 为 `[0.445004, 0.715489]`。因此当前版本只把 `weir1` 视为通过 SWMM 响应 gate 的执行器；`R1/R3` 在确认物理配置或控制含义前，不进入 action-conditioned surrogate 训练。这个结论限定于当前网络配置和协议，不把 R1/R3 外推为所有场景都无效。

这一步只证明 SWMM 的动作写入和当前协议下的物理响应，不证明 surrogate 学会了动作响应，也不证明可以进行 MPC。下一决策点是：与导师确认是否修正 `R1/R3` 的物理配置，或先用单执行器 `weir1` 做最小 action-conditioned pilot；若 Hague 单执行器不足，再将 Astlingen作为第二网络/备用方案。未完成上述确认前，不生成完整 `act=true` 训练集、不训练新 surrogate、不接入 MPC。

## 决策边界

- K=5 仅定义为记录未来 lateral-inflow、完整节点/管道状态观测和冻结 `60->60` 模型下的离线理想反馈候选。
- 现实性检查显示信息协议一旦接近真实条件，滚动误差和尾部风险显著上升；当前 checkpoint 不支撑部署或 MPC/DRL 控制结论。
- Shunqing 仍为 `act: false`，冻结模型没有学习闸门/泵动作响应。下一阶段先确认可控设施，生成按事件隔离的 `act=true` 动作条件化 SWMM 轨迹，并做相同初态、相同 forcing、单动作扰动的成对响应验证；通过前不接入 MPC。
