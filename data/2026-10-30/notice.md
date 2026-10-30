# 2026-10-30 说明（最小版 · 初步 · 召回异常）

- **预期公告**：Thursday, 29 October 2026（北京周五 08:00 = 美东周四 20:00）。status.arxiv.org：周三晚 mailing 因数据库维护取消；Tue 14:00–Thu 14:00 EDT 接收的稿件应在周四晚一并公布。
- **实测（约 08:20 CST）**：RSS 频道 `pubDate=Thu, 29 Oct 2026` 但 **item=0**；API submittedDate 窗口 total=0；`/list/*/new` 曾返回旧 Wed 页或 429。非假期表日。
- **G1**：工作日 API=0 且非假期 → 发「召回异常」而非「无产出」；15 分钟后重试 / 补充 run。
- **G9**：未过；本页标「初步」，不阻塞发布。
- **10-29 续跑**：deep 已收口，papers=[]（见该日 DECISION）。
