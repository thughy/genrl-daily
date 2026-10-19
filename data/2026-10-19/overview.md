**一句话结论**：本期没有必读。最值得看的是 DiffusionRVGE（2610.22973）：它用组内的「reward 差 ~ 干净图像差」做 ridge 回归，从黑盒 reward 里估出一个 reward 梯度方向，再按 DiffusionOPSD 那种「沿梯度移动的 detached 目标」更新速度。同硬件、用官方代码复现的 DiffusionNFT 对照下，达到同一自动指标阈值快 2.1–6.7 倍。**对我们训练的含义**：音画同步、VLM 打分这类 reward 不可微，这给了一个「黑盒 reward → ① 形状方向」的现成办法；但视频上回归类方法压运动最严重，上之前先在同 GPU-h 下比 NFT 和 RVGE，**必须同时记 Dynamic Degree**。

**30 秒速览**
- **DiffusionRVGE**（值得看，新机制 × 站得住〔限效率主张〕）：相对 FDFO（成对有限差分）和 DiffusionNFT，改成多个对比联合 ridge 回归、在干净图像坐标里拟合；干净坐标让方向和真实梯度的 cos 从 0.009 升到 0.107（95% CI [0.092, 0.106]）。另一个可迁移发现：**reward 平滑时，同一轨迹上的小扰动对比**更好（PickScore 等预算下高 0.4–0.7）；**reward 离散时，独立种子组**更好（GenEval 0.92 对 0.84–0.85）。这正是 P11「共享程度是旋钮」的一次等预算扫描。没有人评，训练 reward 就是评测 reward。
- **TempAct**（了解即可，重做 × 存疑）：在 AR-CoPO 的 chunk 分叉局部对比上，加了一个 GSPO 训练的 LLM 规划器。值得记两条消融：硬切换分步 prompt 后得分**低于**单 prompt（0.248 对 0.325）；执行器用整段 reward 反而**不如不训**（0.313 对 0.340）。Gemini 评测只涨 0.04–0.05，单次运行。
- **MetaReach**（观察，机制 × 证据不足，只有摘要）：在世界模型 GRPO 里先把解码器适配到模型自己的输出分布再打分，并按块分配优势。对我们世界模型 RL 最相关，但「打分器跟着被评者一起训」可能让 reward 变宽松，要等全文。
- **ProactGen**（一行，重做 × 存疑）：用 RL 训练「看草稿后提问」的 VLM，增益大头来自看过参考视频的模拟用户，RL 本身只多 0.006–0.010。
- **AVC-Reward 复评**：维持「了解即可」，可信度从「存疑」改为「站得住（测量部分）」。另记 DeSync 提醒：不要单用 AVC-Score，也不要单用 DeSync 作 reward。
- 队友转来的 RPO、NPO 沿用 10-16 评级；FLARE、Dual-Support OPO、WoRL、PhysDMD 只有摘要，都是了解即可或观察（见页底一行带过）。

**框架表（更新方向 × reward 来源）**

| 更新方向 | 可微模型打分 | VLM 打分 | 规则 / 检测器 | 人类偏好 |
|---|---|---|---|---|
| ① 可微 reward 直接回传 | FLARE（DMD + latent RM，摘要） | | | |
| ② 黑盒 reward 策略梯度 | ProactGen（训提问策略，DreamSim）· PhysDMD（中间步 GRPO，WM 能量，摘要） | TempAct（Qwen3-VL 局部遵循 + PickScore） | MetaReach（LPIPS/MSE 对 GT，摘要） | |
| ③ 黑盒正负对比回归 | DiffusionRVGE（PickScore / HPS / CLIP）· WoRL（AR-NFT + NVRM，摘要） | | DiffusionRVGE（GenEval / OCR）· Dual-Support OPO（摘要） | |
| ④ 离线成对偏好 | | | | |
| ⑤ 收敛目标改写 | | | | |

**放不进格子的**：DiffusionRVGE 的目标形状属 ①（detached 梯度目标），方向却来自黑盒 reward 的回归估计，是 ①/③ 之间的桥；ProactGen 训的是生成器上游的提问策略，生成器不动；AVC-Reward（复评）是发现类，检验 C 列「现成 AV 指标可信」这条前提。

**按格应在哪出问题（预测 → 实际）**
- DiffusionRVGE（③，读结果前登记）：会打平 NFT、计算比用估算、不和 OPSD 比、没有人评 → **命中**；PickScore 过饱和 → 没查到，反而出现了 OCR 的「整图大字」hack → **部分命中**。
- TempAct（②×VLM）：组内同分、VLM 评判信号枯竭 → 论文没报近平局占比，**无法判定**（读结果前未登记，不计命中率）。
- MetaReach（②×规则）：ACA 的贡献大于 RCM；解码器吸收误差会让 reward 变宽松 → **待全文**（已登记）。
- ProactGen：读结果前未登记，不计。

**跨篇收敛（本期 + 近 7 天）**
- **P11（受控对比）加一个有等预算扫描的来源**：DiffusionRVGE §6.3 在同 batch、同优化器下比较了「同一轨迹的小扰动对比」和「独立种子组」。平滑 reward（PickScore）下前者高 0.4–0.7，而两个种子之间的差距最多 0.14；离散 reward（GenEval）下 η=0.03 塌到约 0.30，独立种子组 0.92。共享程度该按 reward 的连续性来定，这是 P11「共享过多会把信号一起共享掉」的第一份等预算扫描。PickScore 上是两个种子，GenEval 上是单次（原文核对）。
- **P10（训练状态 = 部署采样状态）**：Dual-Support OPO（只在蒸馏保留的时间步、只在自 rollout 上下文上更新）和 WoRL（消除 AR-NFT 的 clip 内 train–test 错配）都把它当设计原则。两篇只有摘要，**不计独立来源**；全文里如果有对应消融，再计入。
- **新观察「AR 视频 RL 的 reward 要落在更新的那一段」**：TempAct 有消融支持（整段 reward 0.313，不训 0.340，局部 reward 0.390）；MetaReach（按块优势）、FLARE（逐 chunk reward）、WoRL（以历史为条件的逐 clip RM）方向相同，但都只有摘要。有证据的只有 1 个来源，**不立候选**，进观察名单。

**对我们训练的含义（世界模型 AR / 双向 / DMD / 音视频）**
1. **黑盒 AV reward 的更新规则**：先跑的对照是同 GPU-h、同 reward（DeSync + AVC-Reward 组合）下比 Astrolabe 式 AR-NFT 和 RVGE 式回归，记 reward、Dynamic Degree 和人评三条曲线。对比怎么构造跟着 reward 走：平滑 reward 用同轨迹小扰动，离散或平台型 reward 用独立种子组。
2. **交互式切换动作或 prompt**：TempAct 显示硬切换会让 AR 模型退步，在切换处预告下一条件后能恢复。我们的动作条件 AR 模型值得先测「硬切换 vs 预告」。
3. **chunk 级 RL**：reward 只给切换后、被更新的那段，不要用整段分数去更新局部 chunk。

**召回**：逐号抓 abs v1 共 1,482 篇（2610.22373 – 23856），10 类内 610 篇；RL / reward 关键词命中 48 篇；对 287 篇生成相关摘要做了 Codex 初筛（IN 4、EDGE 18、FEWSTEP 7），我逐条复核了 IN、EDGE，并抽查了关键词命中却被标 OUT 的 37 条，没发现漏判。精读 DiffusionRVGE、TempAct、ProactGen，PONDER 只读了关键章节。G9 未通过（见顶部说明），**数字为初步**。

**上期补充**：[10-16 日报](https://thughy.github.io/genrl-daily/daily/2026-10-16/)（已加 AVC-Reward 更正）、[W42 周报](https://thughy.github.io/genrl-daily/weekly/2026-W42/)。
