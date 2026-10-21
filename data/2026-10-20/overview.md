**一句话结论**：本期没有必读。最值得看的是 DFR（2610.24897，Oxford / KAIST / Mila）。KL 系数 β 本身只是个幅度旋钮，调大调小都在同一条「reward–多样性–漂移」曲线上走。DFR 换了一个方向：在每个样本的 advantage 里加一项「这张图离同组其他图有多远、离 base 模型的样本有多近」（分布距离的一阶变分）。这样在同一个 GenEval / OCR reward 阈值下，它能拿到 β 怎么调都到不了的点：多样性更高，离 base 的漂移也更小（3 个种子、同 GPU-h 预算）。**对我们训练的含义**：视频和音视频 RL 里「多样性塌缩 + 运动被压」的老问题，可以直接试 latent-MMD 这一项。它不用编码器，每步开销约多 9%。先跑的对照是「同 GPU-h 下扫 β」对「β + latent-MMD」，同时记 Dynamic Degree 和同 prompt 多种子的 DreamSim。

**30 秒速览**
- **DFR**（值得看，新机制·小 × 站得住）：同 reward 阈值下，β 扫描到不了的「多样性高 + 漂移小」的点，加一项分布距离的一阶变分就能到。GenEval 0.96 阈值下多样性 DreamSim 0.174 → 0.196，FID 119.9 → 110.8。增量相对 RAM 的 β 扫描和 Flow Density Control：一阶变分只当标量权重，不对它求导，所以黑盒 reward 也能用；另外给了一个不需要编码器的 latent MMD 实例。DreamSim 实例就是 DRIFT。
- **NR-SMC**（了解即可，重做 × 存疑）：把 SMC 搬到确定性 flow matching 上，在重采样时锚定终点估计、重新加噪。同 NFE 时和 Best-of-4 打平（ImageReward 1.3720 对 1.3696），每个 prompt 反而慢 4 秒。又一次印证「按墙钟算，BoN 不输搜索」。
- **TTS 偏好优化实证**（了解即可，发现·印证 × 存疑）：DPO 用同一批专家语音，只把 β 从 0.1 改成 0.01，WER 就从 4.59% 涨到 29.78%。另外，专家语音的似然升或降都预测不了 WER。
- **MARTINI**（了解即可，机制 × 证据不足）：用少量 oracle 标注学多个廉价 reward 的组合，再用来做推理时倾斜。图像实验是单次运行，引导强度仍要靠 oracle 来选。
- **队友转来**：少步蒸馏同事的 11 条「归 RL」OpenReview 投稿只有摘要，RISE-DMD（DMD 的目标换成在线拟合的 reward 倾斜 score）、DualHeads（两个分类头 = KL 正则 reward 最大化）进观察，其余了解即可；TempAct、MetaReach 沿用 10-19。多模态文献追踪的 4 条：EVA（逆动力学动作 reward 训视频世界模型）观察，FluidGen 了解即可，AVTime（理解侧描述 reward）、MAPS（广告脚本 agent）在范围外。
- 一行带过：FluidGen（DPO 后 FVD 反而变差，reward 来自模型自己预测的速度场）、BSIT（树搜索，只按 NFE 比）、RGDR（推理时修正，同范数下「对的方向」4.34 对随机方向 1.31，但选候选和评测用的是同一个 VLM 裁判）、SGE-Goal（编辑排行榜靠前的模型做机器人目标图反而更差）。

**框架表（更新方向 × reward 来源）**

| 更新方向 | 可微模型打分 | VLM 打分 | 规则 / 检测器 | 人类偏好 |
|---|---|---|---|---|
| ① 可微 reward 直接回传 | | | | |
| ② 黑盒 reward 策略梯度 | | | | 艺术家共创（2610.24010，按偏好强度加权，HCI） |
| ③ 黑盒正负对比回归 | | | **DFR**（RAM + 分布泛函项；GenEval / OCR） | |
| ④ 离线成对偏好 | FluidGen（自预测速度场打分） | | | TTS 偏好优化实证（专家 vs 学生语音） |
| ⑤ 收敛目标改写 | | | | |
| 推理时（不训练） | NR-SMC（ImageReward）· BSIT（ImageReward） | RGDR（Doubao 裁判）· MARTINI（学到的代理组合） | GeoART（几何先验） | |

**放不进格子的**：DFR 改的不是更新方向这一维，而是 reward 维。它在任务 reward 之外加了一项**依赖当前分布**的逐样本 reward，所以在列维度上应该单开一类「分布项」。SGE-Goal 是发现类，它检验的是「通用编辑质量分可以当 reward / 选择标准」这条前提（见 metrics.md）。

**按格应在哪出问题（预测 → 实际）**
- DFR（③ + 分布项，08:18 读结果前登记）：本质是 batch 依赖的 reward shaping ✓；最接近的前作是 Flow Density Control ✓；评测多样性和训练 reward 同源 ✗（作者已经把同源格划掉，主表用的是 OpenCLIP 分支）；不和其他多样性方法比 ½（DRIFT 作为实例算是比了，GARDO / ReNFT 没比）；增幅约 29% ✓。
- NR-SMC（推理时，08:22 登记）：只比 vanilla / FK ✓；引导 reward = 评测 reward ✓；同 NFE 和 BoN 打平 ✓。

**跨篇收敛（本期 + 近 7 天）**：**本期无**（没有 ≥3 篇独立指向同一机制）。新观察「端点锚定重加噪用于推理时探索」：NR-SMC 和 BSIT 两篇独立地都在 flow 模型上「先估终点，再独立加噪」造新粒子或子节点，只有 2 个来源，不立候选。这和 P10（训练状态必须落在采样器实际访问的分布上）存在张力：NR-SMC 自己也承认，这样做改变了生成先验。另外，P3「方向 > 幅度」新增一个站得住的正例（DFR）和一个弱旁证（RGDR）。

**对我们训练的含义（世界模型 AR / 双向 / DMD / 音视频）**
1. **多样性 / 漂移正则**：先跑「同 GPU-h，β 扫描 vs β + latent-MMD」，记 reward、Dynamic Degree、同 prompt 多种子 DreamSim 三条曲线。MMD 直接在 VAE latent 上算，每个 prompt 要 8 个 base 参考样本。
2. **推理时搜索**：任何 SMC / 树搜索方案，先和**同墙钟**的 BoN 比，再考虑上线。
3. **TTS / 音频 DPO**：β 先扫两个数量级，并以生成指标（WER、SIM）为准，不用似然诊断来挑 checkpoint。

**召回**：逐号抓 abs v1 共 1,954 篇（2610.23857 – 25818），10 类内 915 篇。RL / reward 关键词命中 64 篇。Codex 初筛了全部 915 篇摘要（IN 11、EDGE 28、FEWSTEP 11），我逐条复核了 IN、EDGE 的理由，读了其中 24 篇摘要，并检查了 64 条关键词命中里被标 OUT 的条目，没发现漏判。G9a 通过（API 1,093），G9b 未过（RSS 1,229，差 11%），见顶部说明。精读 DFR；NR-SMC、MARTINI、TTS 实证、FluidGen、BSIT、RGDR、SGE-Goal 由 Codex 抽取要点，我抽查了 5 处关键数字，全部和原文一致。
