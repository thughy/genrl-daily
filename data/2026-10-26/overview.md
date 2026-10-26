**一句话结论**：**初步（召回未完成）**。本期无必读/值得看。范围内最相关是 **ImageRSI**（冻结 T2I + 局部 verify/repair + 经验库 + 对 judge 做 SFT→GRPO）——受控同预算下局部修复远超 Bo2，经验库有 +10pp 曲线，但 GRPO 仅 +3.4pp 且缺等数据续 SFT，生成器未训，标了解即可。

**对我们训练的含义**：
- 不改变「对生成器做哪类 RL」的选择；若做 **agentic 修图**，优先投局部缺陷锁定的 repair，而不是再采样 BoN；经验库要有配对 A/B 准入才有量级收益。
- Judge 上 GRPO 的边际小于 repair/经验；上 GRPO 前先做等数据续 SFT 对照。

### 一屏框架（本期）

| 论文 | 归格 | 按这一格应在哪出问题 | 实际 |
|---|---|---|---|
| ImageRSI | ⑤ test-time verifier × VLM judge；judge 侧 ② GRPO | 评测与经验 verdict 同源；GRPO 与数据量混杂；生成器无更新 | 命中：Gemini 兼经验与 CSA；+3.4pp 无续 SFT 对照；生成器冻结 |
| StackGen3D | ⑤×规则 BoN | 选优偏差 | 仅 BoN，了解即可 |
| DRAMA-Eval | 评测（不入格） | LLM-judge 可靠性 | metrics 观察 |
