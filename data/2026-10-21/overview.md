**一句话结论**：本期没有必读。最值得看的是 RoboJudge（2610.27782）：它在 800 条人标的具身生成视频上测了 30 个 judge，结果是同一个 VLM 判「有没有按指令做」还算可靠（最强 r 0.733），判「物理对不对」明显不可靠（最强 r 0.611）；专门做具身的 MLLM 更差，物理轴 r ≤ 0.26。**对我们训练的含义**：世界模型 / 音视频 RL 如果用 VLM 当 reward，物理这一轴的信号最弱，也最容易被 RL 放大成 hacking。选 judge 前，先在我们自己生成器的人标集上把物理和指令两轴分开测单轴 r；物理轴先用规则或检测器兜底，VLM 只负责指令对齐。

**30 秒速览**
- **RoboJudge**（值得看，发现 × 站得住）：最强闭源 judge 物理轴 r 0.611，指令轴 r 0.733；Cosmos-Reason2、RoboBrain2.5 等具身模型物理轴 ≤ 0.26，VideoPhy2 / VideoScore2 ≤ 0.20。自训的 9B judge 合并 r 0.719，但对最强闭源模型的物理轴增益只有 +0.034，置信区间含 0。它没被拿去当 RL reward，也没和 WorldModelBench 比。相对 PhyMetric（VLM 物理裁判 ρ ≤ 0.293）：第一次在具身视频上按两轴分开测。
- **CheckDistill**（了解即可，机制·小 × 存疑）：teacher 多看 checklist 和逐项判断，在学生自己的打分 token 上做 OPSD，部署时约 10 个 token 出分。对同骨干同数据训的 GRPO 版 RM 只多 0.3–2.6 个点（无种子）。它确实用 DiffusionNFT 训了 Qwen-Image-Edit（ImgEdit 4.29 → 4.46），但没有「换别的 RM」的对照，人像、动作子类反而下降。
- **LoCA**（了解即可，机制 × 存疑）：AR flow TTS 上，把句级 advantage 按 ASR 定位的出错帧做句内重分配，再加两侧阈值。CV3 错误率 GROW 5.55% → 4.76%，但只跑了一个种子，单拿任一组件都约等于 GROW。
- **PhysCanvas**（了解即可，机制 × 存疑，10:04 后放出）：视触觉机器人世界模型，用冻结小 RM 在解码帧上直接回传梯度做后训练；任务结果项用 BCE 匹配真实帧上的判断（成功、失败都复现），世界模型与真机成功率的 Pearson 0.602 → 0.885；在里面 GRPO 训 π0.5，真机 59.1% → 70.8%，但对未后训 WM 的增量多在标准差内。
- 一行带过：GenOlympiad（难题上 VLM 先画辅助线，和人一致率 68.4% → 91.5%）、Attractive Faces（4 个 T2I RM 最看重的是脸好不好看，只读了摘要）、NVV-GRPO、HealBench、Risk Decomposition、Intra-Rubric Reward Divergence（LLM）。

**框架表（更新方向 × reward 来源）**

| 更新方向 | 可微模型打分 | VLM 打分 | 规则 / 检测器 | 人类偏好 |
|---|---|---|---|---|
| ① 可微 reward 直接回传 | PhysCanvas（世界模型后训，ResNet RM） | | | |
| ② 黑盒 reward 策略梯度 | | | NVV-GRPO（摘要） | |
| ③ 黑盒正负对比回归 | | CheckDistill（DiffusionNFT 训编辑） | LoCA（ASR 定位，GROW 式） | |
| ④ 离线成对偏好 | | | | |
| ⑤ 收敛目标改写 | | | | |
| 推理时（不训练） | | | Risk Decomposition（外部物理引导） | |

**放不进格子的**（发现类，挂在 reward 维）：RoboJudge 检验「VLM 打分」这一列的前提，结论是物理轴不成立、指令轴大体成立；GenOlympiad 检验同一前提，结论是给视觉证据才成立；Attractive Faces 检验「人类偏好 / 可微模型」列，发现 RM 学到的是吸引力。

**按格应在哪出问题（预测 → 实际）**
- CheckDistill（③ × VLM，08:35 登记）：增益主要来自 teacher 的证据 ✓；RL 部分没有 RM 对照 ✓；没有独立人评 ✓。
- RoboJudge（发现，08:55 登记）：自训 judge 在自家基准上赢，但不是独立分布 ½（源 clip 不相交，生成器相同）；没当过 RL reward ✓。
- LoCA（③ × 规则，09:24 登记）：比 GROW 好 10–15% ✓（实际 14%）；reward 与评测同源 ✓；没有朴素的阈值对照 ½。

**跨篇收敛（本期 + 近 7 天）**：没有新的原理候选。收敛 4（笼统自动分数测不出具体属性）加入第五个独立来源 RoboJudge，并细化成「物理轴最弱」。新观察「VLM judge 的可靠度主要来自实例级证据，而不是推理长度」（CheckDistill、GenOlympiad），只有 2 个来源。「reward 要落在被更新的那段」加 LoCA，现在是第二个来源。PhysCanvas 给出一个可迁移的小点：世界模型的「结果」项应匹配真实结果（含失败），而不是最大化成功，否则当评估器会系统性高估；只有 1 个来源，先记观察。

**对我们训练的含义（世界模型 AR / 双向 / DMD / 音视频）**
1. **选 judge**：同一批 rollout，用 Gemini / GPT / 自训小 judge，分物理、指令两轴测和人评的单轴 r，再决定 reward 组合；不看合并 r。
2. **给 judge 证据**：先测「实例级核对清单 / 辅助线」能把一致率提多少，再考虑蒸馏成快速 RM。
3. **AR 的 credit**：句级 / 段级 reward 可以按能定位的错误片段做段内零和重分配（LoCA），在 AR 音视频上值得做一个小对照。
