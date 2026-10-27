## 结论
联合音视频双塔上直接 GRPO，同步 reward 的 credit 容易被「另一模态组内乱动」污染；**模态锚定 + 冻塔交替**（AV-GRPO）是可操作的拆法。Flow-GRPO 系发糊时，**ODE+Langevin**（LC-GRPO）仍是 L4 已证实工具，今日正式发卡。

## 对我们训练
- 上 LTX / 联合 AV 的 GRPO：默认先锚定再交替，不要一上来双塔同更；同步评测另留非训练用的 DeSync 指标。
- 自训 Flow-GRPO 发糊：优先对照 ODE+Langevin（或 CPS），不必等新 SOTA 协议。

## 召回
- 晨间：周一 mailing new≈386 ∪ cross≈235；初筛 ≥211；无新 D2。
- 午后：周二 listing 经文献追踪转交 LC-GRPO / AV-GRPO / Fine-Grained Credit（agent）三篇，本站主读定级。

## 跨篇收敛
本期无（≥3 篇存疑指向同一新机制）。观察：token/视觉 credit 线（OVC、DATC、MART-GRPO）与 agent 过程奖励（2610.35497）仍在相邻模态，尚未落到图像/视频生成器同协议。
