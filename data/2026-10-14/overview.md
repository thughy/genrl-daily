**一句话结论**：这期最值得学的不是某一篇，而是三篇独立论文共同指向的一条原理——**前向过程 RL（DiffusionNFT 这一族）里，回归用的参考速度和训练状态必须和「实际采样分布」对上；对不上，就会在 reward 方向之外多出一项偏置，轻则覆盖收窄，重则训练崩溃。** 单篇里只有 GuidanceNFT 达到「值得看」。

**对我们训练的含义**：我们的模型用 CFG（或把 CFG 蒸进去）、做 DMD 少步、做 AR 分块采样，**三处都是「采样分布 ≠ 前向重加噪分布」的典型场景**。以后上 NFT 类 RL，第一个要定的设计选择是：参考速度从哪来（冻结的条件预测 / CFG 场 / 在 rollout 上在线拟合）、训练状态从哪来（重加噪端点 / 回放采样器实际状态）。先跑的对照：同一 rollout，「默认 NFT」vs「在线拟合参考」vs「回放采样器状态」，盯 reward、多样性和 Dynamic Degree，并测参考误差 ‖e‖ 随训练的变化。

### 一屏框架：更新方向（行）× reward 来源（列）

| | A 可微模型打分 | B VLM / judge | C 规则 / 检测器 | D 人类偏好 |
|---|---|---|---|---|
| ① 可微 reward 直接回传 | | | | |
| ② 黑盒策略梯度 | | SpatialCritic（RM + Flow-GRPO）、Audio8-TTS（realism critic）| FlowGRPO-VocalSep（ViSQOL/SI-SDR）、GenRevisit（GenEval2）| |
| ③ 正负对比回归（NFT 族）| **GuidanceNFT**（PickScore/HPS/CLIP 组合；也用 GenEval、OCR → 跨 A/C）、DiffusionART（10-13）| Fork-NFT（VideoAlign/AudioBox/JavisScore）| GeoFix（FOV 校准几何 reward + NFT）| |
| ④ 离线成对偏好 | | Golden Anchor（VideoReward 标签 → Flow-DPO）| | 日文手写 DPO（只读摘要）|
| ⑤ 收敛目标改写 | | | | |

**按格预判**（本期是回扫，预判在部分结果已知后补写，不计入周五命中率）：③ 格先查「参考/状态错配」和多样性；GuidanceNFT 预判命中（核心收益来自修参考，但多样性没量）。② × B/C 先查训练 judge 与评测是否同源：SpatialCritic、VocalSep 命中（评测与训练 reward 同族/同指标）。任意 × C 先查运动/相机是否被压：GeoFix 自己测到了（去掉校准后 Dynamic 0.425→0.395）。

**放不进格子的（发现类，挂在它检验的格子下）**：

- *Imperfect Verifiers / CS-VGB*（推断时，不训练）：挂在 ③ 的前提上——「去噪核和重加噪核必须配对，配对错误的平稳分布偏差随错配大小而变」，是 P10 在推断时的同构旁证。
- *GeoFix 的 overlap 偏差*：挂在 C 列——几何一致性类 reward 会把「相机少动」误当成「几何更好」。
- *SpatialCritic 的 GenEval 泛化崩溃*：挂在 C 列——拿 GenEval 规则当训练 reward，SD3.5 七项 OOD 均分从 78.4 掉到 51.9（Flow-GRPO）。

### 跨篇收敛：P10 候选（可信度按「独立印证」评）

**原理**：NFT 的回归目标隐含一个「参考速度」；只有当它等于 rollout 分布在该状态的条件均值速度、且训练状态来自 rollout 实际访问的分布时，正负两支才只沿 reward 方向推。否则更新 = 预期方向 Δ + (1−2α)·e，e 是参考误差，这一项与 reward 无关。

| 来源 | 场景 | 证据（来源标签） | 单篇可信度 |
|---|---|---|---|
| GuidanceNFT 2610.15856（Tsinghua/Duke）| CFG / 内置 guidance 采样 | Eq.10：更新偏置 (1−2α)e（原文核对推导）；树形玩具里 v_ref=v_c 时偏置项 RMS 0.32，是预期更新 0.13 的约 2.4 倍（原文核对 App.B.2，**是玩具模型，不是 SD3.5**）；SD3.5 CFG-4：参考误差越大崩得越快，analytic 修正的场分歧 5.43→148.06，GuidanceNFT 保持约 0.008（原文核对 App.E）| 存疑（单次训练）|
| DiffusionART 2610.12572（NUS/Kling）| 普通 NFT（无 CFG）| Prop.1：NFT 力 = 2g + (2q−1)e，e = 前向速度 − 采样场（原文核对，推导我手算核过）| 存疑（无干预实验）|
| Fork-NFT（OpenReview WGavcJGfuS，匿名）| 22B 八步 AR 音视频 | Tab.4：重加噪端点训练 JavisScore 0.162 < 不训 0.186；回放采样器状态 0.205（原文核对）| 存疑（单次、无人评）|
| 旁证：CS-VGB 2610.15281 | 推断时搜索 | 去噪/重加噪核配对 → 平稳分布精确；配对误差 → 有界偏差（原文核对定理陈述，未核证明）| 不计入独立来源 |

**为什么算独立印证**：三组作者互不引用（我在 GuidanceNFT 全文里检索 DiffusionART / Fork-NFT / Astrolabe，均无），场景不同（CFG 图像、无 CFG 图像/视频、少步 AR 音视频），证据类型不同（推导 + 玩具精确解、分解推导、端到端消融）。**综合可信度：中高**——方向一致、形式几乎相同（(1−2α)e 与 (2q−1)e 只差符号约定）；但三篇都没有第三方复现，且没有一篇在视频 NFT 上直接测过 ‖e‖。

**可迁移原理分**（与是否推送分开排）：GuidanceNFT 5/5 · Fork-NFT 4/5 · DiffusionART 4/5 · GeoFix 3/5 · CS-VGB 3/5 · SpatialCritic 2/5 · 其余 ≤1。
