**一句话结论**：本期没有必读，也没有值得看。最相关的两篇做的是同一件事：给少步生成器同时接一个 reward 和一个「参考判别器」，让 reward 优化不塌缩。DualHeads（2610.28858）用两个分类头，AdvFidelity（2610.29380）用一个只看伪影的判别器。两篇都是了解即可：判别器那一侧的参考图本身已经被 reward 筛过（基座 best-of-8），或者直接由 RL 老师生成，所以增益里有相当一部分是在蒸馏参考池；而且两篇都没跟 DiffusionOPSD 比。**对我们训练的含义**：DMD 学生（世界模型 / 音视频少步）接黑盒 reward，最便宜的形式就是「reward 头 + 参考判别头」，不需要 teacher score 和似然。但先跑两个对照：只用参考项（λ_R = 0），以及参考池换成没按 reward 筛过的样本，看增益里多少来自参考池。判别器的负样本用「真实片段加噪 + 学生少步还原」。HPSv3 / ImageReward / CLIPScore 不能拿来证明画质。

**30 秒速览**
- **DualHeads**（了解即可，机制 × 存疑）：SD3.5-M 4 步 OCR 0.92、GenEval 0.90，同 NFE 的 TDM-R1 是 0.95 / 0.92（抄录）。Fig.16：去掉参考头，reward 约 900 步后崩到 0；只留参考头，训练 reward 只到约 0.7（完整方法约 0.89，读图）。所以两个头各管一件事。参考池换成 Nano Banana 后 OCR 不变，GenEval 0.90 → 0.73，参考池管的是 reward 之外的一切。Z-Image 上 HPSv3 11.23 → 13.93，但 HPSv3 对少步伪影不敏感，单次运行，没比 OPSD / RISE-DMD / FLARE。
- **AdvFidelity**（了解即可，机制 × 存疑）：判别器的负样本 = 参考图加噪后由当前生成器 1–2 步还原，内容一样，判别器只能看伪影。FLUX 8 步 HPSv2 0.4125，全步 LeapAlign 是 0.4096（抄录）；GenEval（held-out）0.7759 对 0.7420。但 HPDv2 的参考图是 HPSv2 对齐过的 LeapAlign 样本，消融里「换成这些参考」一步就占了总增益的一半（0.0362 / 0.0709）；没有只用 task reward 的对照，单次运行。附带一个指标发现：干净图对 1–2 步伪影图，HPSv3 只判对 38.3%，ImageReward 48.2%，CLIPScore 40.3%。
- **ReView**（了解即可，机制·小 × 证据不足，只有摘要）：新视角视频 RL，reward 是跨视角重投影，rollout 从单帧到混合再到全视频逐步拉长，称不到 1/3 算力追平全视频 RL。全文拿不到（OpenReview 403）。对我们世界模型 RL 的省算力思路最相关，留观察。
- 一行带过：HorizonComposer（CFGRL 式 advantage 条件化 + 逐像素 reward 图，驾驶视频编辑，重做）、CineEvolve、Internal Guidance、T-DIG、DoobPO、Events, Not Pixels。另有回补的 Correction Training（漏筛事故条目，内容域外）。

**框架表（更新方向 × reward 来源）**

| 更新方向 | 可微模型打分 | VLM 打分 | 规则 / 检测器 | 人类偏好 |
|---|---|---|---|---|
| ① 可微 reward 直接回传 | AdvFidelity（HPSv2 + 伪影判别器，沿少步轨迹回传）；DualHeads（对学到的 reward 头 + 参考头反传） | | DualHeads（OCR / GenEval 蒸进 reward 头） | |
| ② 黑盒 reward 策略梯度 | | CineEvolve（训 Director / Critic，生成器不动） | ReView（重投影 reward，算法未知，摘要） | |
| ③ 黑盒正负对比回归 | | | HorizonComposer（advantage 条件化 ≈ 加权回归，逐像素 reward） | |
| ④ 离线成对偏好 | | | | |
| ⑤ 收敛目标改写 | Internal Guidance（推理时倾斜采样，理论） | | | |

**放不进格子的**：Events, Not Pixels 检验「用画质评世界模型」的前提，结论是回报误差集中在少数事件步，像素指标挑不出来（摘要）。AdvFidelity Tab.6 检验「可微模型打分」这一列能否当伪影指标：HPSv3 / ImageReward / CLIPScore 不行，Aesthetic / PickScore 可以。

**按格应在哪出问题（预测 → 实际）**
- DualHeads（①，08:2x 初判）：①格应出问题的地方是「hack 可微代理、过饱和」。实际：Fig.8 样本加细节、饱和度比起点高（粗测 48.3 → 60.3），但比 TDM-R1（73.1）克制 ½；没比 OPSD ✓。
- AdvFidelity（①，精读前未登记预测，不计命中率）。

**跨篇收敛（本期 + 近 7 天）**：没有新的原理候选。①「少步模型上的 reward 优化需要显式分布锚，否则塌缩」：有证据的来源 2 个（DiffusionOPSD 重跑 NFT 在 Z-Image-Turbo 上 10 个目标里 8 个跌破未训练的 base；DualHeads Fig.16 去掉参考头约 900 步崩溃），加 AdvFidelity 的动机图（没有定量）和 FLARE / RISE-DMD（只有摘要），还不够立候选。② 新观察「判别器 / 参考项的参考池往往已被 reward 倾斜」（DualHeads、AdvFidelity），这类方法要同时报「只用参考项」和「只用 reward 项」，2 个来源。③「先学改进方向、再蒸馏进生成器」（DoobPO 与 NFT / RISE-DMD 同构）记一笔，不计来源。

**对我们训练的含义（世界模型 AR / 双向 / DMD / 音视频）**
1. **DMD 学生接 reward**：先试「reward 头 + 参考判别头」（DualHeads 形式），对照组固定两个：λ_R = 0；参考池不按 reward 筛。
2. **对抗项的负样本**：用「真实片段加噪 + 学生少步还原」，比从纯噪声生成更对准伪影（AdvFidelity；少步同事对 Radian 得出同样的便宜实验，推断）。
3. **评测**：伪影类问题不要用 HPSv3 / ImageReward / CLIPScore 判，加人评或在 AdvFidelity Tab.6 上敏感的指标。
4. **长时域 RL 省算力**：ReView 的「单帧 → 全段」课程值得等全文；先在我们 AR 世界模型上做「只训短段 vs 课程」的小对照。
