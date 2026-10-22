**一句话结论（初步）**：截至 08:21 已放出的 344 个编号里，还没有能直接定为必读或值得看的论文；最相关的是 DualHeads（2610.28858）。它是 10-20 期从 OpenReview 摘要记下的观察项，今天 arXiv 全文放出：用两个二分类头把「黑盒 reward」变成少步生成器能直接反传的可微代理，但对比的少步基线数字全是从原论文抄的，没有种子。

**候选（初步，均待精读）**

| 论文 | 化简后的等价形式 | 归格（更新方向 × reward 来源） | 对应最强基线 | 初判 |
|---|---|---|---|---|
| DualHeads（2610.28858，少步 × reward） | 生成器最大化 w_r·logit(reward 头) + w_ref·logit(参考头)；头最优时 = 学到的 reward + KL（GAN 式密度比），权重比 = β | ①（学到的可微代理）× A/C（OCR、GenEval、PickScore 等外部打分蒸成头） | 同格：DiffusionOPSD（①线要求的基线）；少步 reward 倾斜：RISE-DMD / FLARE、TDM-R1、RTDMD | 观察 → 待精读。4 步 OCR 0.59 → 0.92、GenEval 0.63 → 0.90，与 40 步 Flow-GRPO 相当；少步基线（TDM-R1 0.95、REST 0.96）是抄录数，没跑 OPSD |
| HorizonComposer（2610.28829，驾驶视频编辑） | 离线 RL 的 advantage conditioning + 从混合数据里抽的像素级 dense reward | 放不进格子（advantage 条件化 ≈ reward 加权回归，接近 ③）× C（数据导出的像素 reward） | advantage / reward 条件化生成的前作；同格 DiffusionNFT | 疑似重做（换任务），待查 dense reward 消融 |
| Gaussian Priors Fail（2610.28849，EDGE） | VAE 高斯先验把多峰压到重叠 latent，RL 微调把重叠放大成坍缩；分子 Boltzmann 实验 | 发现类：挂 reward 维之外的「latent 几何」前提 | — | 观察：只有分子数据，能否迁移到图像 latent 待看 |

**框架表雏形（待补）**

| 更新方向 | 本期（初步） |
|---|---|
| ① 可微 reward 直接回传 | DualHeads（学到的代理头） |
| ② 黑盒 reward 策略梯度 | 暂无 |
| ③ 黑盒 reward 正负对比回归 | 暂无（HorizonComposer 接近） |
| ④ 离线成对偏好 | 暂无 |
| ⑤ 收敛目标改写 | 暂无 |
| 放不进格子的 | HorizonComposer（advantage 条件化）；Gaussian Priors Fail（latent 几何发现） |

**已知待补项**：① 剩余编号放出后补筛，API 与 RSS 对账（G9a/G9b），按 G10 取并集复核；② DualHeads 精读（查头的过拟合、与 OPSD / RISE-DMD 能否同协议比、reward hacking 样例）；③ HorizonComposer 查 dense reward 消融与数据是否开源；④ 少步关键词命中转给少步蒸馏同事；⑤ 世界模型条目（WorldStream 2610.28827）转多模态文献追踪；⑥ HF Daily Papers 10-22 还没出，出来后查漏；⑦ 跨篇收敛一节随完整版给出。
