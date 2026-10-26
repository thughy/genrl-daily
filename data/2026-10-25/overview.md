**结论**：周末补扫未发现过线新机制。两篇队友抄送均属「已知对齐框架搬到视频物体编辑」——PhysInsert = Diffusion-DPO + 插入区局部 mask；UniVOE GRAR ≈ DiffusionNFT 符号优势回归、去掉 reference（编辑任务以输入视频为内容锚）。对我们训练：不改变「选 NFT/DPO 还是 GRPO」的决策；若做局部编辑后训练，优先做 **global vs local vs 二者相加** 与 **有无 reference** 两个对照。

| 更新方向 | 本期 |
|---|---|
| ① 可微 reward | — |
| ② 黑盒 PG | — |
| ③ 黑盒正负回归 | UniVOE GRAR（了解即可） |
| ④ 离线成对偏好 | PhysInsert Spatial DPO（了解即可） |
| ⑤ 收敛目标改写 | — |
