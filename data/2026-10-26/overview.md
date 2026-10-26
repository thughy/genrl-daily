**一句话结论**：**初步（召回未完成）**。无必读/值得看。补评 **ACWM-RM**：真视频 IDM 在生成视频 AF 上不可靠，作 Flow-GRPO reward 会反降 nDTW 并伤 VQ；互补 mismatch 无参考 RM 可联合抬 VQ+AF，但 Pref VQ 与评测重叠、无人评 → 了解即可。另有 **ImageRSI**（局部 repair≫Bo2，GRPO 边际小）。

**对我们训练的含义**：
- 动作条件世界模型 / 视频 RL：上 GRPO 前先在**生成 rollout** 上测 IDM/AF 打分器；不可靠就不要当主 reward。
- 不改变「对 T2I 生成器做哪类 RL」的选择；agentic 修图仍优先局部缺陷锁定。

### 一屏框架（本期）

| 论文 | 归格 | 按这一格应在哪出问题 | 实际 |
|---|---|---|---|
| ACWM-RM | L5 reward-model × ② Flow-GRPO；诊断⑥ | Pref VQ 与评测同源；人造 AF 标签；单骨干 RL | 命中：PSNR/LPIPS overlap；nDTW 非人评；仅 DreamDojo-2B；IDM 反伤双轴成立 |
| ImageRSI | ⑤ test-time verifier × VLM judge；judge 侧 ② GRPO | 评测与经验 verdict 同源；GRPO 与数据量混杂；生成器无更新 | 命中：Gemini 兼经验与 CSA；+3.4pp 无续 SFT 对照；生成器冻结 |
| StackGen3D | ⑤×规则 BoN | 选优偏差 | 仅 BoN，了解即可 |
| DRAMA-Eval | 评测（不入格） | LLM-judge 可靠性 | metrics 观察 |
