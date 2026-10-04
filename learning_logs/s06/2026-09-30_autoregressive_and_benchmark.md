# 2026-09-30：自回归滚动与速度基准

## 本次目标

冻结 `s06_formal_10000_b8_seed42/best`，不重新训练，验证模型在反复使用自身预测状态时的误差变化，并测量 surrogate 与 SWMM 在同等事件时长下的本地推理/仿真速度。

## 自回归协议

- 模型：冻结 S06 `GATConv + Conv1D`，最佳更新 `9098`。
- 每个事件只使用最初 60 步真实历史；之后每个 `60→60` 块把预测的节点状态和管道状态反馈到下一块。
- 下一块的节点 Flood 通道由模型预测概率按 `0.5` 阈值转为二值；管道 setting 通道保留事件数据中的常量设置。
- lateral-inflow boundary 使用数据中记录的未来 `states[..., 3]`。因此这是 state-feedback rollout，不是完整的 history-only 闭环，也不是降雨预测实验。
- 归一化参数和 Node/Pipe/Flood 指标定义与正式 teacher-forced evaluator 一致。

输出位于本地 surrogate 项目：

- `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/autoregressive_validation/`
- `D:/论文/GNN-UDS/surrogate/model/shunqing/s06_formal_10000_b8_seed42/autoregressive_heldout/`

### 事件级最终累计误差

| split | 事件数 | Node RMSE mean / median / P90 | Pipe RMSE mean / median / P90 | Flood volume RMSE mean / median / P90 |
|---|---:|---:|---:|---:|
| validation | 27 | 3.1958 / 2.3853 / 5.4453 | 2.4907 / 1.8511 / 4.4166 | 4.2240 / 3.2167 / 7.1305 |
| held-out test | 3 | 2.2210 / 2.0396 / 2.6958 | 1.7480 / 1.5946 / 2.1842 | 2.4756 / 2.1729 / 3.1451 |

验证集累计误差不随时间单调增长，受事件雨型影响明显。27 场验证事件最终 Node RMSE 最大为 `12.5545`（`bpswmm_1011`），说明少数事件会出现明显长时段退化；不能用平均曲线声称 rollout 稳定。按滚动块统计，验证集 1/2/4/8 小时的 Node/Pipe/Flood chunk RMSE 均值分别为：

| horizon | Node | Pipe | Flood volume |
|---:|---:|---:|---:|
| 1 h | 1.194 | 0.961 | 1.437 |
| 2 h | 2.756 | 2.190 | 4.538 |
| 4 h | 4.345 | 3.408 | 6.008 |
| 8 h | 1.779 | 1.273 | 1.488 |

8 小时处的下降反映不同事件长度/雨型的样本组成，不表示模型在单个事件内自动恢复。长时段控制用途仍需反馈校正或缩短模型块长。

## 速度基准

脚本：`D:/论文/GNN-UDS/surrogate/scripts/benchmark_s06.py`。

硬件/软件：RTX 4050 Laptop GPU（6141 MB）、TensorFlow 2.10.0、Python 3.10.21、Windows；冻结模型共 `1,064,396` 个 trainable parameters；TensorFlow 记录的 peak allocation 约 `700 MB`，`nvidia-smi` 运行时显存约 `4.5 GB`（包含运行时缓存）。

首轮 3 个留出事件、每个事件一次重复的端到端结果：

| event | SWMM seconds | surrogate warm rollout seconds | forecast steps | 按步数归一化 speedup |
|---|---:|---:|---:|---:|
| `test_bpswmm_118` | 45.90** | 0.179** | 720 | 236.49** |
| `test_bpswmm_133` | 54.15** | 0.197** | 840 | 256.06** |
| `test_bpswmm_1612` | 64.47** | 0.241** | 1020 | 252.46** |

`*` 首轮 3 事件一次性测量中，surrogate 首次记录同时包含 TensorFlow trace，不能作为稳定 warm 延迟。`**` 为每个事件独立 2 次重复测量的 warm 值；稳定重复目录分别为 `benchmark_s06_event141_repeat/` 和 `benchmark_s06_event142_145_repeat/`，三事件归一化 speedup 为 `236.49x/256.06x/252.46x`，中位数 `252.46x`。SWMM 计时通过 ASCII 路径 `C:/s06_148_event_inp` 的 `DataGenerator.simulate` 完成，包含 PySWMM 启动、逐步推进、状态抽取和数组物化。surrogate 计时包含已加载模型的递推、GPU 同步和状态回灌，不包含模型加载；SWMM 原始事件还包含 60 步上下文，因此 speedup 已按 forecast steps 做线性步数归一化。

重复测量中单个 `60→60` teacher-forced batch 的 warm 延迟约 `14.1 ms`（P95 `15.8 ms`）。batch size `1/2/4/8/16` 的 warm 吞吐约为 `4093/4681/4028/3691/3629` 个预测步/秒；不同 batch 的第一次调用包含形状 trace，不能混入 warm 统计。

## 当前决策

这轮结果支持继续做“短块预测 + 反馈校正”的最小控制原型，但还不支持直接上 DRL 或把当前 surrogate 当作长时稳定环境。下一步优先固定滚动块长、加入周期性的 SWMM/观测反馈，再用同一速度口径评估控制性能。
