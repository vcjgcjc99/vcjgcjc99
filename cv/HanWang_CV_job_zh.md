# 汪涵

+86-15956609883 | wanghan334452@gmail.com | 山东大学软件学院  
GitHub | Google Scholar

**求职方向：** 具身智能 / 机器人学习 / Vision-Language-Action / 强化学习

---

## 教育背景

**中国科学院自动化研究所｜博士研究生（博0，2027 级）** · 2027.09 – 2032.06  
拟从事具身智能、机器人学习与多模态智能体相关方向研究。

**山东大学（985 / 211 / 双一流）｜软件学院 · 软件工程专业人工智能班｜本科** · 2023.09 – 2027.06  
GPA：3.984/5，综合成绩专业排名：6.18%，CET-6：548；获北京中关村学院深澜计划优秀营员 Offer。  
竞赛：全球 AI 算法精英大赛全国二等奖，全国大学生数学竞赛省级一等奖，全国大学生数学建模竞赛省级二等奖。  
核心课程：高等数学 A+，概率论 A+，数据分析与挖掘 A+，离散数学 A+，数据结构 A，计算机视觉 A。

---

## 论文成果

1. **[Accepted]** Han Wang, Wei Du, Yanyu Xu, Lizhen Cui. *Dynamic Bias Compensation in Soft Actor-Critic: A Method for Stable Value Estimation.* Pattern Recognition, 2026. [CCF-B · JCR Q1 · 中科院一区 TOP] [10.1016/j.patcog.2026.114410](https://doi.org/10.1016/j.patcog.2026.114410)

2. **[Oral]** Han Wang, Wei Du, Yanyu Xu, Lizhen Cui. *SAC-B: Soft Actor-Critic with Bias for Suppressing Q-value Overestimation in Off-Policy Reinforcement Learning.* DAI 2025. [CCF-C · Oral] [10.1145/3772429.3772436](https://doi.org/10.1145/3772429.3772436)

3. **[Preprint]** Hongnan Ma, Han Wang, Shenglin Wang, et al. *CreativeGame: Toward Mechanic-Aware Creative Game Generation.* arXiv:2604.19926, 2026. [项目页](https://yiweishi-cn.github.io/CreativeEvolutionGame/index.html)

---

## 科研项目经历

### EBIM 具身智能比赛 Task 2｜上海星骞元机器人科技有限公司 / 上海交通大学 · 2026

指导者：顾商定（上海交通大学长聘副教授）、冯骞（TUM PhD）

聚焦 **EBIM Task 2：Deformable Material Handling / Thermal Pad Placement**，在 Mobile FR3 Duo 双臂平台上完成导热垫边缘抓取、跨板搬运、姿态对齐与放置；竞赛 **EBiM Benchmark Competition 2026 · Task 2 全球第 4 名**（StarCore SJTU · 官方榜单），并进入 Phase II 真机验证。

**CV 方案（`cv_planb_v2`，[StarCoreRobotics/ebim-challenge](https://github.com/StarCoreRobotics/ebim-challenge)）要点：**

- **架构**：将感知与执行收敛为 **Isaac Sim 单进程** 状态机（S1→S6），取消 ROS 双进程与 anchor 握手，便于分阶段调试与提交。
- **S1 感知**：头载 RGB-D + HSV 颜色 mask 估计 pad **+Y 悬边（lip）** 与目标板 **board_target** 世界坐标（多帧中位数稳定）；控制环仅使用机器人相机与本体 FK，真值仅用于本地打分。
- **S2 接近**：**显式「先下降、再水平前伸」**（S2a/S2b）；**冻结腕部专家三明治构型**（J5–J7 固定），平移仅用 J1–J4，避免笛卡尔横扫与越界顶入 sit 区。
- **S3 抓取**：稳定等待后合爪并锁存闭合；**gripper≥0.5** 才进入揭起，防止空爪 peel。
- **S4–S6**：近端关节小增量抬升（避免 TCP +Z 滑边）；搬运/放置段结合 **专家轨迹回放** 与 **Lula IK** 将放置帧末端位姿平移至检测板位；本地按官方口径自评 **orientation × IoU**（评分：`grasp_success × orientation × IoU`）。

---

### 强化学习与智能决策研究｜山东大学软件学院 · 2025.04 – 2025.11

围绕 Off-policy RL 中 Q 值估计偏差与训练不稳定，提出 **SAC-B / 动态偏差补偿（DBC-SAC）**，提升 Critic 估计稳定性。

- 在 **六个标准连续控制基准** 上，整体性能与 **SOTA DSAC-v2 持平**；在 **边缘计算平台** 部署评测中取得 **最优或并列最优** 表现，且 **单步推理时延更低**，更适合资源受限在线决策。
- 成果发表于 DAI 2025（CCF-C，Oral）与 Pattern Recognition（CCF-B，双一区 TOP）；开源：[github.com/vcjgcjc99/SAC-B](https://github.com/vcjgcjc99/SAC-B)。

---

### LLM Agent 与科学发现 Benchmark｜UCL / University of Bristol · 2026.03 – 至今

指导教师：汪军教授、杨梦月老师

推进第一作者项目 **ScienceBench**：构建 **交互式科学发现世界**，在复杂动力系统下考察模型能否 **自主发现规律**——并行搭建 **理论世界**（假设形成、知识迁移与修正）与 **实验世界**（可执行交互与隐藏测试），通过自动化评测与历史回放评估 Agent 能否形成并迭代可检验的科学规律。在研进行中（尚未投稿）。

---

### 具身大模型与世界模型奖励｜北京中关村学院 · 深澜计划 · 2026.03 – 2026.06

指导教师：于超老师

围绕 **π0.5 + JEPA-2 世界模型** 路线，在 RLinf / Isaac Sim 类环境中探索 VLA 持续学习与内在奖励设计。

- **初步结论**：利用 **π0.5 训练过程中的 checkpoint 与仿真器交互** 来训练世界模型，有助于 **奖励（reward）头训练不崩溃**；相较仅依赖专家轨迹 **L1** 的监督，基于世界模型的 **reward 模块展现出附加成效**（对策略信号有补充而非单纯复现专家）。
- 完成工程链路摸底与短周期营员训练；与后续 EBIM 真机项目形成「仿真—世界模型—操作」的能力衔接。

---

### CreativeGame｜上海交通大学 / Bristol · 多智能体游戏生成与进化 · 2026

指导教师：温睦宁老师、杨梦月老师

参与 **CreativeGame** 多智能体迭代式 HTML5 游戏生成框架（[arXiv:2604.19926](https://arxiv.org/abs/2604.19926) · [技术报告页](https://yiweishi-cn.github.io/CreativeEvolutionGame/index.html)）：Planner 检索机制库并输出 **`CurrentMechanicSet`**，驱动 Skeleton → Feature → Visual → Refinement 分阶段 codegen；Tester / Evaluator 对计划与实现做对照打分，Reflector 将机制级增量写入谱系记忆与全局 archive（774+ 机制条目、多 lineage 迭代）。

- 参与游戏规则建模、计划—实现一致性评测与多轮演化流程；预print 已挂 arXiv，并获 **机器之心** 推广报道。

---

## 技能与研究方向

**研究方向：** 具身智能、机器人学习、VLA、世界模型奖励、强化学习、LLM Agent 科学发现评测。  
**具身实践：** VLA / 世界模型微调、双臂可变形物体操作（Isaac Sim 分阶段 CV+控制）、仿真到真机验证。  
**科研产出：** 一作 / 主要作者覆盖强化学习、ScienceBench、CreativeGame；具备论文写作、实验设计与系统评测经验。  
**技术栈：** Python / PyTorch，Isaac Sim 与机器人仿真，RLinf，多智能体系统与自动化评测。
