# 框架速览（2026-11-03）

按「更新方向从哪来」：

| 格 | 本期 |
|---|---|
| ① 可微 reward 回传 | — |
| ② 黑盒策略梯度 | **LARC**（了解即可）：UMM 上 latent 轨迹 + info-gain GRPO |
| ③ 正负对比回归 | — |
| ④ 离线偏好 | — |
| ⑤ 收敛目标改写 | — |
| ⑥ 测量/诊断 | **Fair Cast**（值得看）：bf16 RN 导出死区 + 无偏导出修复 |

对我们训练：先核对 RL 栈的 master→serve cast；UMM latent 叙事暂不跟跑，等「R=−Lflow」类对照。
