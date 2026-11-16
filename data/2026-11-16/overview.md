## 结论：先验评估链，不换训练配方
**Verifier Choice值得看**：总分只偏人评−2.85pp的裁判，仍能把生成器人差25.44pp压成.44pp。关键不是“裁判有偏”这句旧话，而是固定图像、固定人标的生成器条件偏差能大到掩盖方法差距；同构第三生成器与renderer对照使它成为直接诊断证据。

| 研究对象 | 更新哪个对象 | reward/参照来源 | 本轮结论 |
|---|---|---|---|
| Verifier Choice | 不更新生成器；干预测量协议 | 人工valid/count标签 | worth，固定suite诊断成立 |
| Chekhov’s Gun | 不更新；同prompt配对审计 | 既有alignment/preference指标与标签 | D1→后续D2，未终判 |
| PhyRubric | rubric与judge；事件/物理分开 | 共享rubric人标 | D1→后续D2，未终判 |
| PhysLoop | 模拟约束生成/修复工作流 | 物理验证+工作流反馈 | D1保留，不是已证递归训练闭环 |

诊断不强塞五格梯度更新；缺训练多样性不是排除它的理由。反过来，固定suite反转也不等于所有视频RL结果被推翻。