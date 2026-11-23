# 2026-11-23 结论与框架（D2增补，覆盖未闭环）

**不立刻改训练配方。** Activation Steering带来的是诊断：在已测教师/奖励上，冻结基座+输入无关逐块偏置已足以重现大部分特定reward收益；**90.0–96.3%不是通用质量恢复，多样性teacher/fit仅base .517/.522**。SPM把“采样器速度”和“分布forward velocity”拆开，但实际surrogate、输入gap与同强度正则对照仍待补。

| 线 / 更新方向 × reward | 已核新知识 | 终判与限制 |
|---|---|---|
| ② Online PG / 记录奖励 | SibPO：只对唯一记录未来给奖会伤条件频率 | 维持值得看，不宣称全面画质最优 |
| 少步模型偏好接口 | BlockAlign：监督部署不用的子头可能比不训更差 | 维持值得看，不改配方；蒸馏本身分工不变 |
| ⑥ RL增益诊断，附②向量RL | Activation Steering：输入无关干预能恢复六图像教师90.0–96.3%指定reward增益 | **值得看·新发现×站得住，限充分性诊断**；不是全网单向量/无训练/免teacher训练，副作用也复制 |
| ③ 黑盒signed residual regression × A/C | SPM：shadow posterior为每call提供reward方向，fake分布速度替代采样器速度 | **了解即可·新机制×存疑**；仅surrogate，非无偏保证，归本追踪RL主评 |
| L5 Reward / 情绪 | EmoReward v2 Dance Overall低于基线 | 维持了解即可，勘误保留 |
| 队列未完成 | CANDOR25457、Keystone25530、RiDEx22052、Advantage-Gated23038、T2I RM17585、Motion-Aware RM22495 | 只登记，未精读未定级；不静默当落选 |

### 对训练的含义
- Activation Steering先用作teacher-fit诊断探针：同prompt/seed配对，同时看独立裁判、human quality与多样性，不只看训练reward。参数少65倍不是训练算力省65倍。
- SPM先验收同teacher-KL/同更新RMS下EM是否确实缩小shadow/deployed输入gap；Lemma条件与完整实现近似分开，不因终点涨分宣称机制已证。
- 种子口径：26627主fit每setting一个training seed，仅OCR/PickScore子集做3训练seed检查；主表五个图像行一个sampling seed、GenEval4、Wan2.1为2、Wan2.2为4。25998训练seed数未报告，不擅写单seed。

### 跨篇与覆盖限制
本轮新增两篇共同提醒“被测/被监督的表示”不一定等于实际部署计算；**仅两篇，不据此新增≥3篇独立收敛原理**。既有SibPO、BlockAlign结论保留，未改变本线最强方法或W44持有。

积压列表/API统计与本日新mailing分开。周一23日未计覆盖，X/GitHub/种子共引等仍缺；11-20周报仅延误提纲，OUT召回抽检/全周分歧汇总未完成，不报虚构胜率或零漏检。
