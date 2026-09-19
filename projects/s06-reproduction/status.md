# S06 复现状态

## 环境

- 源码目录：`GNN-UDS/surrogate`
- Conda 环境：`gnn_uds`
- Python：3.10
- TensorFlow：2.10.0
- Spektral：1.3.1
- NumPy：1.23.5
- SciPy：1.15.0
- 关键库已完成导入检查。

## 当前阶段

正在进行 SWMM 数据生成：

```powershell
python main.py --simulate --env shunqing --data_dir paper_like --rain_suffix bpswmm --processes 1
```

最近一次检查时，123 个输入事件中约 90 个输出事件已经完成，当前进程仍在运行。数据生成结束后，`paper_like` 中才会出现 `states.npy`、`perfs.npy`、`rains.npy`、`edge_states.npy`、`event_id.npy` 和 `dones.npy`。

## 下一步顺序

1. 等待完整模拟结束并确认上述 `.npy` 文件生成；
2. 用 2 个 epoch 做训练 smoke test；
3. 将 smoke test 扩展到 100 或 500 个 epoch；
4. 再尝试 60→60、3 层空间 GNN、edge fusion 和 flooding 配置；
5. 记录 RMSE/MAE、洪涝分类指标、推理延迟、吞吐量和显存；
6. 做 GNN 与普通 NN、node-edge fusion 与无 fusion 的对照实验。

## 重要提醒

- CUDA 缺失提示不等于程序失败，当前可以使用 CPU；
- 第一次不要直接从 20,000 epoch 开始；
- 不要把老师未公开的数据、模型权重和大型 `.npy` 文件上传到 GitHub；
- 论文配置和仓库默认配置可能不同，实验时要记录实际参数。
