# 数据格式

所有内容都是 UTF-8 JSON。长文本字段都写 Markdown（支持表格、列表、代码）。
任何长文本字段也可以写成 `{"file": "2026-10-09/xxx-details.md"}`，路径相对 `data/`，适合很长的「完整细节」。

站名、副标题、线上地址、**方向标签列表**都在 `config.json` 里配置；增删方向只改 `config.json` 的 `directions`（`id` 用于数据，`label` 用于显示，顺序即筛选按钮顺序）。

## 每日文件 `data/YYYY-MM-DD.json`

| 字段 | 必填 | 说明 |
|---|---|---|
| `date` | ✔ | 与文件名相同，如 `2026-10-09` |
| `headline` | ✔ | 一句话头条：今天最重要的进展 |
| `papers` | ✔ | 论文数组，可为空 `[]`（当天没有达标论文） |
| `trend_summary` |  | 当天趋势总结（Markdown） |
| `brief` |  | v2：重做类和未入选论文「一行带过」，显示在速览底部。数组项为 `{"title", "line", "link"?, "originality"?, "credibility"?}`，`line` 支持行内 Markdown |
| `revision_note` |  | 改版说明（Markdown），在头条下方折叠显示 |
| `fyi_open` |  | `true` 时「了解即可」区默认展开（降级但仍有深度卡的论文需要被看到时使用） |
| `version` |  | 内容版本号（仅记录，不影响渲染） |
| `draft` |  | `true` 时不发布 |
| `example` |  | 仅示例数据使用；正式 `data/` 中出现会被 build.py 拒绝 |

### `papers[]` 每篇论文

卡片按下面顺序固定展示（空字段自动省略）：
**评级 + 评级理由 → 核查深度 → TL;DR → 核心 insight → 与已有工作的区别/脉络 → 方法（机制）→ 关键证据 →（Reward 核查表）→ 样例结论（含图/对比网格）→ 代码结论 → 疑点与信息缺口 → 完整细节（默认折叠）**

**v2 卡片（原创性优先，2026-10-10 起）**：只要论文写了 `lineage`，卡片就改用下面的顺序，旧数据不受影响：
**评级 + 原创性×可信度 + 评级理由 → 核查深度 → 新知识（`tldr`）+ 相对谁（`delta`）→ ① 脉络与增量（`lineage` 小表 + 反事实结论）→ ② 机制（`insight` + `method`）→ ③ 证据（`evidence`）→ ④ 扎实度（`solidity` + Reward 核查表 + 样例/图 + 代码结论）→ ⑤ 疑点（`concerns`）→ ⑥ 对 Henry 的启发（`takeaway`）→ 完整细节**

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✔ | 当天唯一的短 slug（小写字母/数字/连字符），用作锚点 `#p-<id>` |
| `rating` | ✔ | `must`（必读）/ `worth`（值得看）/ `fyi`（了解即可，页底折叠，不进汇总页） |
| `directions` | ✔ | 数组，取值见 `config.json`，默认：`online-rl`（Online RL，GRPO 等）、`offline-dpo`（Offline/DPO）、`diff-reward`（可微 reward 优化）、`ar-unified-rl`（AR/统一模型 RL）、`reward-model`（Reward model 与 reward hacking）、`video-rl`（视频生成 RL）、`distill-rl`（少步/蒸馏与 RL）、`test-time`（Test-time scaling）、`other` |
| `title` | ✔ | 论文标题（原文） |
| `tldr` | ✔ | 一句话结论（速览和汇总页显示的就是它） |
| `rating_reason` |  | 一句话评级理由（紧跟评级显示） |
| `coverage` |  | 核查深度：`{"paper": v, "samples": v, "code": v, "note": "..."}`，v ∈ `full`（已深读）/ `partial` / `none`（未看）/ `unavailable`（未公开） |
| `institutions` |  | 机构数组 |
| `published` |  | 发表信息，如 `arXiv 2026-10-08 (v2)` |
| `links` |  | `{"paper": url, "project": url, "code": url, "other": [{"label": "...", "url": "..."}]}` |
| `highlight` |  | `true` 时优先进入汇总页「本周最重要」 |
| `insight` |  | 核心 insight |
| `context` |  | 与已有工作的区别 / 脉络 |
| `method` |  | 方法（机制）：loss、advantage、KL、采样/去噪步上的 credit assignment 等 |
| `evidence` |  | 关键证据（数字、表格、消融） |
| `reward_check` |  | 可选「Reward 核查」小表，见下 |
| `samples` |  | 样例结论：亲眼看样例后得出的结论（过饱和、构图趋同、多样性塌缩等） |
| `media` |  | 样例图/视频/对比网格，显示在「样例结论」里，见下 |
| `code_analysis` |  | 代码结论（实现是否与论文一致，写 文件:行号） |
| `concerns` |  | 疑点与信息缺口 |
| `details` |  | 完整细节，`<details>` 默认折叠 |
| `originality` |  | v2 两维评级之一：`mechanism`（新机制）/ `finding`（新发现）/ `redo`（重做） |
| `credibility` |  | v2 两维评级之二：`solid`（站得住）/ `doubtful`（存疑）/ `insufficient`（证据不足）。必读 = 原创（mechanism/finding）且 solid |
| `delta` |  | 半句「相对谁」：相对最接近的已有工作，本文改了哪一处。显示在速览和 TL;DR 下面 |
| `lineage` |  | ① 脉络与增量，见下。有这个字段，卡片就切换为 v2 顺序 |
| `solidity` |  | ④ 扎实度的文字部分（reward 重叠、hacking、人评、样例、代码），后面接 `reward_check`、`samples`/`media`、`code_analysis` |
| `takeaway` |  | ⑥ 对 Henry 的启发：可迁移性、什么时候值得用 |

### `lineage`（v2，可选）

```json
"lineage": {
  "rows": [
    {"work": "DiffusionOPSD（Zhou 2026）", "link": "https://arxiv.org/abs/2608.24646",
     "how": "它怎么做（行内 Markdown）", "change": "本文改了哪一处（行内 Markdown）"}
  ],
  "change_header": "可选，覆盖第三列表头，默认「本文改了哪一处」",
  "note": "可选补充（Markdown），显示在表格下方",
  "counterfactual": {"value": "partly", "text": "懂这些工作的人能否预测到结果，以及为什么（Markdown）"}
}
```

- `rows` 必填且非空，每行都要有 `work`、`how`、`change`；放最接近的 2–3 篇，而且必须真的读过。
- `counterfactual.value`：`unpredictable`（预测不到，是新知识）/ `partly`（部分能预测）/ `predictable`（能预测到，属于重做类）。
- 旧字段 `context` 在 v2 卡片里会显示在表格下方。

### `reward_check`（可选）

```json
"reward_check": {
  "train_reward": "PickScore + OCR 准确率",
  "eval_reward": "GenEval、PickScore、HPSv2.1、ImageReward",
  "overlap": {"value": "partial", "note": "PickScore 既训练又评测"},
  "heldout": "HPSv2.1 为 held-out；无人评",
  "hacking_signs": "高 CFG 下饱和度上升，人脸趋同",
  "note": "可选补充（Markdown）"
}
```

| 字段 | 说明 |
|---|---|
| `train_reward` | 训练用的 reward（字符串或数组） |
| `eval_reward` | 评测用的 reward / benchmark |
| `overlap` | 训练与评测是否重叠：`yes` / `partial` / `no` / `unknown`，或 `{"value": ..., "note": "..."}`；`yes`/`partial` 标红 |
| `heldout` | 是否有 held-out reward 或人评 |
| `hacking_signs` | 可见的 reward hacking 症状（没有就写「未见」并说明看了多少样例） |
| `note` | 补充说明 |

字段都可省略；整个 `reward_check` 不写就不显示。

### `media`

普通图片/视频：`{"src": "assets/YYYY-MM-DD/xxx.webp", "caption": "..."}`，可放 ≤2MB 的 `.mp4`。

对比图网格（同一 prompt 下 base vs RL 等）：

```json
{
  "grid": {
    "cols": ["SD3.5 base", "+Flow-GRPO", "+本文"],
    "rows": [
      {"label": "prompt 1：三只红色的猫", "cells": ["assets/2026-10-09/x-p1-base.webp", "assets/2026-10-09/x-p1-grpo.webp", "assets/2026-10-09/x-p1-ours.webp"]},
      {"label": "prompt 2", "cells": ["...", "...", null]}
    ]
  },
  "caption": "同 seed 对比；本文第 3 列无过饱和"
}
```

每行格数必须等于 `cols` 列数；`null` 表示缺图。手机上网格可横向滑动。

媒体文件必须放在 `assets/<同一天日期>/` 下，先用 `optimize_assets.py` 处理：图片 webp、宽 ≤1200px；视频 ≤2MB。

## 周总结 `data/weekly/YYYY-Www.json`（ISO 周，如 `2026-W41`）

| 字段 | 必填 | 说明 |
|---|---|---|
| `week` | ✔ | 与文件名相同 |
| `title` | ✔ | 一句话：本周方向整体走向 |
| `summary` | ✔ | 周总结正文（Markdown） |
| `range` |  | `["2026-10-05", "2026-10-09"]` |
| `highlights` |  | `[{"date": "2026-10-09", "paper_id": "xxx", "title": "...", "note": "..."}]`，链接到当天卡片 |
| `threads` |  | 方向脉络（Markdown） |
| `watchlist` |  | 下周关注（Markdown） |

## 最小示例（结构示意，非真实内容）

```json
{
  "date": "2026-10-09",
  "headline": "……",
  "papers": [
    {
      "id": "paper-slug",
      "rating": "must",
      "rating_reason": "……",
      "directions": ["online-rl"],
      "title": "……",
      "tldr": "……",
      "links": {"paper": "https://arxiv.org/abs/xxxx.xxxxx"},
      "coverage": {"paper": "full", "samples": "partial", "code": "unavailable"},
      "reward_check": {"train_reward": "……", "eval_reward": "……", "overlap": "no"}
    }
  ],
  "trend_summary": "……"
}
```
