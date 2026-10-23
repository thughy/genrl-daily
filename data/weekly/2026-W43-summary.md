## 一句话
本周学到的主轴仍是 **「信号何时可信」**，新增两块：①RoboJudge——具身视频生成裁判里 **过程对齐（PA）系统性弱于结果对齐（IA）**；②DiffusionRVGE——黑盒 reward 的组内 ridge 估梯度，把估计问题从 d 维降到组内 k 维。流程上：本质标准进 **影子两周**（不改两轴评级）；D2G「单次运行」事实错误已更正；首次正式召回抽检硬漏 0。

覆盖：**10-19 ~ 10-23**（ISO W43）。来源标签：实测 / 原文核对 / 笔记转述 / 推断。

## 1. 证实 / 推翻 / 加强
1. **RoboJudge（2610.27782，值得看，发现×站得住）——L7 具身视频裁判。** 同模型内 PA < IA（Gemini-3.7-Flash PA 0.611 / IA 0.709；GPT-5.5 PA 0.489 / IA 0.733）；专用具身 MLLM PA ≤ 0.261；自训 RoboJudge-9B 合并 +0.089 但测试生成器全在训练集、未当 RL reward（原文核对）。**对我们**：上具身/机器人视频 RL 前，先分测 PA vs IA，不要只用结果分。
2. **DiffusionRVGE（2610.22973，值得看）——效率侧机制。** 组内 ridge 估 û，E cos² = min(n/k,1)；余弦 0.009→0.107；GenEval 0.90 用 22.8 GPU-h vs NFT 102（原文核对）。平滑 reward 偏好同轨迹小扰动，离散 reward 要独立种子组（§6.3）。OCR 版式趋同已记更正（App.H.1）。
3. **DFR（2610.24897）——方法扎实但可预测性高。** 与 Self-Verifier 盲排时 Codex 判 DFR 增量可由 FDC/DRIFT 推出；我同意「专家可预测方向+量级」→ 不升本质（回测笔记）。
4. **P11 / D2G 事实更正（事故级数字错误）。** 原卡片写「单次运行」→ §5.1 实为 Stage II **3 种子**；§5.4 位移 **1.8±0.6 px vs >15 px**。10-15 期 notice + brief + overview 已于 2026-10-23 更正上线（commit b8f5116）。评级不变（仍了解即可/重做类数据点）。
5. **无新增正式原理候选。** 「评判信号衰减」仍 2 源观察；P10/P11/收敛 4 维持。

## 2. 各线最强基线（本周变动）
- L7：**+RoboJudge 协议提示**（分测 PA/IA）；其余 IsoVRBench / SyncReward-Bench / AVC 不变。
- L2：DiffusionRVGE 进入「黑盒→OPSD 式」候选对照（效率），不取代 DiffusionOPSD 协议位。
- L3/L4/L5/L6/L8：本周无更换最强。

## 3. 对我们训练的含义 → 先跑哪个对照
| 洞见 | 我们哪里会踩 | 先跑 |
|---|---|---|
| PA ≪ IA（RoboJudge） | 机器人/世界模型视频 RM | 同一裁判分测过程轴与结果轴；PA 低就不要当唯一 reward |
| RVGE 组内 ridge | 黑盒 reward + 组相对 | 同预算扫：独立种子组 vs 同轨迹 η 扰动（按 reward 是否平台型） |
| D2G/P11 共享前缀 | 视频 GRPO 分叉 | 继续：按组内方差占比选分叉（零训练） |

## 4. 预测命中率（10-19 ~ 10-23）
- **预先登记且可计**：DiffusionRVGE、ProactGen、TempAct、MetaReach（预测保留待全文）、NR-SMC、CheckDistill、RoboJudge。粗计：**命中为主**（CheckDistill 5/部分1；RoboJudge 3/部分1；NR-SMC 3/3；RVGE 效率方向命中）。
- **未事先登记、不计命中率**：MARTINI、Expert Supervision TTS、DualHeads、AdvFidelity、HeyGen Voice（今日方法与结果同次读）。
- **系统偏差续观察**：发现类（RoboJudge）这次有登记；方法类「可推导」在 DFR/RVGE 回测里被 Codex 纠偏——与影子标准一致。

## 5. 事故与更正
| # | 事件 | 处置 |
|---|---|---|
| 1 | **D2G「单次运行」事实错误**（G6 旧账 + 回测再确认） | 10-15 期 notice/overview/brief 更正上线；lineage 此前已对 |
| 2 | **10-22：138 篇延迟公告漏筛**（G11 补筛） | pastweek 补筛 IN 0；Correction Training 一行+转少步；G11 仍待 Henry 确认 |
| 3 | **周五 G9 未过**（API 窗口 0，列表仍周四） | abs 尾部召回 + 完整版注明 G9 状态；poll/tail 仍后台 |
| 4 | Self-Verifier 底座参数更正（10-22） | 已在 10-15 notice |

## 6. G6 抽查「重做/落选」3 篇（本周）
| 论文 | 当时 | 复看 | 误拒？ |
|---|---|---|---|
| DualHeads 2610.28858 | 了解即可（机制×存疑） | 推导干净；缺 OPSD/RISE-DMD 同协议；参考池已被 reward 筛 | 不误拒 |
| AdvFidelity 2610.29380 | 了解即可（机制×存疑） | 少步同事评值得看；我降级因未比最强+参考来自 RL 老师——理由仍成立 | 不误拒（口味分歧已记） |
| HorizonComposer 2610.28829 | 了解即可（重做×存疑） | CFGRL + 像素 reward；无代码/数据替换消融 | 不误拒 |

**误拒 0**。不触发调门槛。

## 7. 召回抽检（首次正式周五流程）
- 脚本：`recall_spotcheck.py 2026-10-19` → OUT 池 2781，risky 810，样 10+30。
- **硬漏 0 / 软漏 2**：FrameGRPO（视频理解 GRPO，非生成）；AEON（视频+3D 世界模型，应转文献追踪而非静默 OUT——流程上属软漏）。
- 阈值未触发（硬≥1 或软≥3）。与 10-22 回测同型。不改 triage 提示。

## 8. Codex 分歧 tally（本周，含 10-22 本质回测）
- 记录 13 条（多在回测）：**Codex 对 4 / 我对 3 / 都对一半 1 / Codex 部分 1 / 未决 4**。
- 类型偏「方法」的 T1 已测与「诊断 vs 方法」评级。
- 教训已写入流程：先验包、变量必须已测、种子原文定位、附录失败案例检索。

## 9. 影子本质清单（2026-10-23 起，约至 11-06）
本周 **新精读未判影子「本质/本质·存疑」**（HeyGen Voice = 重做；DualHeads/AdvFidelity/CheckDistill 维持技巧或机制·小）。  
回测遗留待 Henry：CriticHack 分类、WorldAlign 假阳性风险。频率自检：影子本质 0 ≤2；本质·存疑 0 ≤3。

## 10. 待 Henry
- G10/G11 是否正式写入（目前待确认）。
- CriticHack 最终类别。
- 影子两周后是否切 v2.3 正式评级（约 11-06）。
- 11 月 routine 改约 09:13（夏令时结束）。

---
## 已核对清单（G7 摘要）
| 断言 | 核对 |
|---|---|
| D2G 3 seeds + 1.8±0.6 px | 2610.17882 §5.1 / §5.4；站点 b8f5116 |
| RoboJudge PA/IA 数字 | 2610.27782 原文；10-22 已更正混模型笔误 |
| 召回抽检池 2781/810 | sample.json 2026-10-19 |
| HeyGen 62.2% / 8332 | §5.4 Tab.11 原文 |
