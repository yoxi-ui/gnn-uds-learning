# S06 当前正式基线与降雨工况结果

更新时间：2026-09-28

本目录保存 S06 Shunqing 复现与扩展的轻量结果附件，来源是本地 `GNN-UDS/surrogate` 项目中的正式训练、冻结基线分析、降雨工况分析和完整条件化训练。目录不包含原始数组、SWMM 文件或模型权重。

## 统一协议

- 数据：148 场事件，113 个节点，131 条管道；
- 划分：118 train / 27 validation / 3 held-out test；
- 留出事件：`test_bpswmm_118`、`test_bpswmm_133`、`test_bpswmm_1612`；
- 模型：`GATConv + Conv1D`，10,000 次更新运行的 `best` checkpoint；
- 输入/输出：60 步历史 → 60 步预测；
- 测试：`stride=60`、不重叠、teacher-forced、真实历史状态输入，不反馈模型预测状态；
- 测试规模：43 个窗口、2,580 个预测时间步；
- 训练归一化参数只由 118 个训练事件计算，最终误差在反归一化物理单位上计算。

## 冻结基线

- `frozen_baseline.json`：冻结配置、最佳更新 9098 和数据划分说明；
- `run_summary.json`：10,000 次训练运行摘要；
- `three_run_comparison.md`：2,000/5,000/10,000 次训练对比；
- `three_run_comparison.csv`：三档训练汇总表；
- `loss_curves.png`、`loss_curves.csv`：训练/验证损失曲线及数据；
- `event_diagnostics.md`、`event_diagnostics.csv`、`event_diagnostics_detailed.csv`：三个留出事件的总体和变量级指标；
- `event_metrics_comparison.png`、`event_10000_variable_diagnostics.png`：逐事件和变量级可视化。

10,000 次冻结基线在三个留出事件上的总体结果为：Node RMSE `1.9949`、Pipe RMSE `1.5784`、Flood volume RMSE `2.2512`、Flood F1 `0.9920`。

## 降雨工况分析

- `baseline_report.md`：冻结基线和总雨量分组报告；
- `intensity_analysis_report.md`：最大逐步雨强分组、bootstrap 95% CI 和相关性报告；
- `event_rainfall_features.csv`：逐事件降雨特征；
- `event_teacher_forced_metrics.csv`：逐事件 teacher-forced 误差；
- `event_rainfall_error_merged.csv`：降雨特征与误差合并表；
- `rainfall_group_error_summary.csv`：总雨量/峰值雨强分组统计；
- `rainfall_error_correlations.csv`：峰值雨强与误差的 Pearson/Spearman 相关；
- `rainfall_error_correlations_baseline.csv`：总雨量分析的完整相关性表；
- `rainfall_group_thresholds.json`、`intensity_analysis_manifest.json`：训练事件阈值来源和 bootstrap 配置；
- `total_rainfall_error_scatter.png`、`total_rainfall_group_boxplots.png`、`max_intensity_error_scatter.png`、`rainfall_feature_error_scatter.png`、`validation_group_boxplots.png`：分组和连续变量图形。

验证集最大逐步雨强与 Node/Pipe/Flood RMSE 的 Pearson 相关约为 `0.9367/0.9494/0.9311`，高于总雨量对应的 `0.7012/0.7083/0.6855`。这是事件级关联，不是因果证明；三个留出事件也没有超出训练集最大峰值雨强。

## 完整降雨条件化训练

- `conditioned_run_summary.json`：完整单 seed、5,000 次训练摘要，最佳 epoch `4876`，最佳验证损失 `0.0281887360`；
- `conditioned_s06_config.yaml`：条件化训练配置；
- `rainfall_condition_features.csv`、`rainfall_condition_stats.json`：六个 forcing-only 特征及仅由训练事件拟合的标准化统计；
- `conditioned_comparison_report.md`：完整训练后的统一比较和筛选结论；
- `conditioned_event_metrics.csv`、`conditioned_group_summary.csv`、`conditioned_vs_baseline_event_metrics.csv`：逐事件和分组比较表。

完整训练的验证集高峰值雨强组相对冻结基线变化为：Node RMSE `-15.44%`、Pipe RMSE `-13.53%`、Flood volume RMSE `-64.97%`；验证集整体事件级平均变化为 Node `-16.91%`、Pipe `-18.71%`、Flood `-56.90%`。负值表示误差增加。该模型作为完整训练但未通过预设筛选标准的探索性负结果保存，不应宣称已经解决强降雨泛化问题。

条件向量是整场事件的静态降雨摘要，包含预测窗口之后的降雨信息，因此适合已知完整降雨过程的情景模拟，不是严格的 history-only 在线预测。

## 数据与复现边界

当前正式实验是基于 S06 思路的本地复现与扩展，不声称与 S06 原文的隐藏 event-ID 划分完全一致。原始 `.npy`、SWMM `.inp/.out/.rpt`、模型权重、完整日志和运行目录仍保留在本地源码项目。
