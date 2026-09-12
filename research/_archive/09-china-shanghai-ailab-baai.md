# 国内对接地图 · 上海AI实验室 + 北京智源 BAAI(2026-07)

> Vision Pro 虚拟角色项目。PI/领导层对 ≥2 源(官网+新闻/会议)核验;arXiv ID 来自搜索元数据(arxiv.org 本环境被屏蔽,建议浏览器复核)。

## 第一部分 — 上海人工智能实验室 SHLAB

### 1.1 机构与领导(已核验)
- 全称 Shanghai Artificial Intelligence Laboratory(SHLAB),2020-07 于 WAIC 揭牌。
- **主任 & 首席科学家:周伯文 Zhou Bowen**(已核验)。
- **创始人物(2023 故):汤晓鸥 Tang Xiao'ou**—CUHK 教授、**商汤创始人**(历史商汤关联,见 1.6)。
- 其他资深:**姚期智 Andrew Yao**(图灵奖、CAS 院士,与周伯文共任 2026 浦江年会主席);**乔宇 Yu Qiao**(首席/leading scientist,兼 SIAT 多媒体实验室主任);**陈恺 Chen Kai**(大模型中心负责人,领衔 **Intern-S1** 科学多模态)。

### 1.2 "书生 Intern" 家族
| 名 | 领域 | 状态 |
|---|---|---|
| **InternLM(书生·浦语)** | LLM | 多版本开源 |
| **InternVL** | 视觉-语言基座 | InternVL 3.5 最新开源 |
| **Intern-S1** | 科学多模态(1B–100B+) | "最佳开源多模态族",AI4S |
| **InternVLA-A1** | VLA:统一理解+生成+动作 | arXiv **2601.02456**(经 alphaxiv 聚合,建议直接核 arxiv)|
| **InternVLA-M1** | **空间引导 VLA**—空间 grounding+机器人控制 | arXiv **2510.13778** 已核验 |
| **InternVLA-N1** | 仿真相关 | **论文 ID unconfirmed**(仅 GitHub 描述)|
| **VeBrain(视觉具身大脑)** | "通用"具身脑:感知+空间推理+决策 | 2025-06 发布,多伙伴 |

> 你 prompt 里的 **"InternSpatial"** 非独立模型名——空间 grounding 工作以 **InternVLA-M1** 发表。"Landmark" 出现在 EchoMimic 的面部 landmark 条件,非机器人项目。

### 1.3 OpenRobotLab / 浦器(你要接触的具身AI团队)
- **OpenRobotLab(浦器)**是 SHLAB **OpenXLab 浦源**开源生态(WAIC 2022 发布 9 子实验室)的机器人/具身支柱。GitHub org **`InternRobotics`**("Building inclusive infrastructure for Embodied AI, from Shanghai AI Lab")。
- **领衔 PI(带细微差别)**:
  - **卢策吾 Lu Cewu**—上交 CSE 教授;人民日报系源称"浦器团队负责人"。
  - **庞江淼 Jiangmiao Pang**—"青年科学家/浦器 team leader"(CEAI 2024 官方议程 + 雷峰网)。站 https://oceanpang.github.io/ —"Research Scientist at Shanghai AI Laboratory. We go with Intern Robotics. Our mission is to develop Embodied AGI systems."
  - **曾嘉 Jia Zeng**—青年研究科学家;SJTU 博士 2023;领衔"虚实贯通"具身技术;做 VLA。https://zeng-jia.github.io/
  - **解读**:卢策吾=教研级 lead/SJTU 侧 PI;庞江淼=浦器现任 in-house 运营负责人;曾嘉=VLA lead。

### 1.4 签名具身论文/项目
- **InternVLA-M1**—*"A Spatially Guided VLA Framework for Generalist Robot Policy"*,arXiv **2510.13778**(2025-10)。统一**空间 grounding + 机器人控制**;空间线索桥接 指令→动作。项目 https://internrobotics.github.io/internvla-m1.github.io/ ;GitHub https://github.com/InternRobotics/InternVLA-M1
- **InternVLA-A1**—统一理解/生成/动作,声称 arXiv 2601.02456,12 项真机+仿真基准超 π0。(ID 仅经聚合见,直接核 arxiv。)
- **VeBrain**—首个集成视觉感知+空间推理+具身决策的"通用"机器人脑(2025-06)。
- **书生具身全栈引擎**—与上海国人形机器人创新中心发布;书生具身操作大模型+数据引擎,sim-real 融合。

### 1.5 数字人/虚拟 avatar(对 Vision Pro 角色相关)
- **EchoMimic**—*"Lifelike Audio-Driven Portrait Animations through Editable Landmark Conditioning"*,arXiv **2407.08136**(2024-07),**AAAI 2025**,230+ 引。
  - **出处注**:canonical repo 是 **`antgroup/echomimic`**——主所有者是**蚂蚁集团**,SHLAB 相关者合著。按**蚂蚁集团主导合作**处理,非纯 SHLAB 产品。
  - 音频/面部 landmark/二者→肖像视频;V2 扩到半身对话动画。https://github.com/antgroup/echomimic
- SHLAB 数字人相关能力:多模态生成(InternVL/Intern-S1)、3D 视觉、EchoMimic 音驱动画。**未在已核验源中识别 SHLAB 主导的专门"虚拟人产品"**——unconfirmed。

### 1.6 商汤/产业关系
- 关联是**历史性、人脉型,非公司**:SHLAB 已故创始人汤晓鸥亦是商汤创始人。SHLAB 是政府背景新型研发机构,非商汤子公司。与商汤等联合开源 InternLM 屡见中文媒体。勿把"SHLAB=商汤"简单化。

### 1.7 合作通道
- **OpenXLab 浦源**开源生态;书生·浦源大模型挑战赛、浦科 AI-for-Science 平台。
- **浦江 AI 学术年会**(SHLAB 主办,国际学术交流;2026 由姚期智+周伯文主持)—天然社交场。
- **浦江书院**(产业+学术双导师联培博士)、**联培博士**(2027 批,含上海大学等)。
- **SHLAB 实验室级未发现公开可单独寻址的"开放课题"**(不像 BAAI/同济/PKU 的国家重点实验室模式)。对外/个人开发者现实通道:(a) OpenXLab 开源贡献,(b) 会议/workshop 社交(浦江、Intern Robotics Workshop),(c) 经 SJTU/HKUST/CUHK 学术合作者合著。

### 1.8 对你 Vision Pro 项目的意义
- **InternVL/Intern-S1**—感知骨干(对房间/用户/物体的视觉-语言理解)。
- **VeBrain + InternVLA-M1 空间模块**—直接相关于虚拟角色需对用户位置、物体位置、指向/反应的**空间 grounding**。
- **EchoMimic**—角色音驱面部/上身动画。
- **caveat**:以上都面向*物理*机器人。均非 Vision Pro/visionOS/Swift SDK——你是复用权重或思路,非即插库。代码多 Apache-2.0/MIT,逐 repo 核。

---

## 第二部分 — 北京智源人工智能研究院 BAAI

### 2.1 机构与领导(已核验)
- 2018-11 经北京智源行动计划成立;非营利新型研发机构,北京市政府支持。
- **理事长:黄铁军 Huang Tiejun**(北大教授)。
- **院长:王仲远 Wang Zhongyuan**(新华网访谈确认)。
- **学术副院长:唐杰 Tang Jie**(清华教授,**悟道 WuDao 奠基 lead**)。
- **具身智能大模型负责人:王鹏伟 Wang Pengwei**(已核验;领衔 RoboBrain、RoboOS、ORCA 世界模型;人大企业博导;前阿里达摩院、快手)。

### 2.2 四大研究支柱(官网)
1. 大语言模型 LLM—BGE、Tele-FLM。
2. 多模态大模型 MLM—Emu、**OmniGen**、EVA、**Painter**、**SegGPT**、See3D、Bunny、VideoXL。
3. 生命大模型 LSLM—OpenComplex、实时数字孪生心脏、**智源线虫(全生物体模型)**。
4. **具身大模型 ELM**—具身模型系统、应用方案、具身数据。**最相关于你。**

### 2.3 悟道 WuDao(旗舰历史)
- 悟道 1.0(2021-03,中国首个超大规模智能模型系统);悟道 2.0(2021-06,当时世界最大稠密模型 1.75T);悟道 3.0 后开源进 FlagAI(现入 Linux Foundation);WuDaoCorpora。2025 起**从"悟道(语言)"转向"悟界(世界)"**(第7届智源大会,2025-06,四方向:神经科学、具身AI脑、生命科学、统一模态)。

### 2.4 具身AI:RoboBrain 2.0、RoboOS 2.0、ORCA(对你的旗舰)
- **RoboBrain 2.0(具身大脑)**—开源具身"大脑"LLM;模块化编解码,统一感知+推理+规划。多项空间推理/任务规划 SOTA(任务规划较 1.0 +74%)。**32B 版**。
- **RoboOS 2.0**—"世界首个具身AI SaaS 开源框架";集成 **MCP 协议**+serverless,跨本体部署;单机+云版。
- **2025-07-14 全开源**(权重+训练代码+评测)。
- RoboBrain-Dex(灵巧操控预训练)、RoboBrain-Dopamine、RoboBrain-SpatialTrace(2.0 Pro)。
- **ORCA**—BAAI"世界基座模型",王鹏伟具身模型研究中心。https://hub.baai.ac.cn/view/55796
- **RoboSkill**—具身能力"技能商店"。

### 2.5 FlagOpen(飞智)—开源系统 + Linux Foundation
- **FlagOpen**=BAAI"大模型界的 Linux"开源技术体系:算法/模型/数据/工具/评测。https://flagopen.baai.ac.cn/
- 组件:**FlagOS**(跨芯片统一 AI 软件栈,支持 18 厂商 32+ 芯片)、FlagEval、FlagData、**FlagAI(模型框架,已加入 Linux Foundation)**。
- 智源学者 3.0(BAAI Scholars 3.0)2025 年 50 名学者。

### 2.6 智源大会(旗舰社交场)
- 年度,自 2019。第7届(2025-06-5~7)发布悟界。**第8届:2026-06-12~13**,200+ 顶级科学家 + 40+ AI 公司 CEO;track 含 Agent、World Models、**具身智能**、AI 自进化、深度推理、多模态。**具身开放日**与 40+ 生态伙伴共办。智源社区 ~19 万从业者。

### 2.7 合作通道(对外/外国开发者)
BAAI 在结构上比 SHLAB 更对外友好:
- **智源学者 3.0**—公开征集,年支持 50 名研究者(FlagOS、具身AI 全栈、多模态、跨学科)。
- **FlagOpen/FlagAI(Linux Foundation)**—个人(含非中国籍)最低摩擦路径:标准开源 PR,无需中国归属。
- BAAI 行动计划原 mandate 明含"共建联合实验室"+"开放服务平台"。
- **《国际人工智能开源合作倡议》**—BAAI 与**工信部**、OpenAtom、CSDN 共发,明确国际范围。
- **关联国家重点实验室(年度开放课题,~20–30 万/项)**——最像西方 PI grant 的"开放课题"机制:
  - 跨媒体通用人工智能全国重点实验室(北大)—2025 征集明确列具身AI题,含**"可泛化的人与物体的全身交互动作生成"**——直接相关虚拟角色。
  - 自主智能无人系统全国重点实验室(同济)。
  - 多模态人工智能系统全国重点实验室(CAS 自动化所)。
  - **caveat**:这些开放课题通常要求申请人持博士/副高职称于研究机构,并与实验室成员合申。**无学术归属的个人外国开发者不能直接申**——需经中国学术合作者,或走 FlagOpen。

### 2.8 对你 Vision Pro 项目的意义
- **RoboBrain 2.0 + RoboOS 2.0**—模块化 感知→推理→规划 管线,正是具身虚拟角色所需;**RoboOS 的 MCP 支持**是现代 agent 式架构的干净集成点。
- **Emu/OmniGen/SegGPT/Painter**—强视觉生成/分割栈,用于角色渲染与场景理解。
- **ORCA 世界模型**—若要潜在"世界模拟器"驱动角色对房间的预期。
- **caveat**:同 SHLAB,具身栈面向物理机器人;大脑模块可复用,动作/传感层非 visionOS 原生。FlagOpen 无 Apple 平台特定移植(预期 Linux/CUDA 优先)。你 RTX Pro 6000 96GB 在这些模型推理画像内(RoboBrain 32B 舒适承载)。

---

## 第三部分 — 对比与建议
| 维度 | 上海AI Lab | 智源 BAAI |
|---|---|---|
| 具身旗舰 | InternVLA-M1(-A1/-N1)、VeBrain | RoboBrain 2.0/RoboOS 2.0/ORCA |
| 空间智能 | InternVLA-M1 空间 grounding | RoboBrain-SpatialTrace、空间推理基准 |
| 数字人/avatar | EchoMimic(蚂蚁主导合作)| Emu/OmniGen(生成),无专门 avatar 产品 |
| 最强开源合作路径 | OpenXLab 贡献 + 经 SJTU 学术合著 | **FlagOpen(Linux Foundation)+ 智源学者 + 国家重点实验室开放课题** |
| 对外国个人友好? | 正式机制有限 | **更好**:FlagOpen 国际开放;智源大会双语友好 |
| 最相关 2026 活动 | 浦江 AI 学术年会 | **智源大会 2026(6/12-13)+ 具身开放日** |

**对开发者的具体建议:**
1. **技术起点**:评估 BAAI **RoboBrain 2.0 + RoboOS 2.0(MCP)** 作认知/规划脚手架,SHLAB **InternVL 3.5** 作感知骨干,**EchoMimic** 作角色动画。均开源。
2. **BAAI 合作路径**:贡献 FlagOpen(个人最低摩擦、国际开放),再经智源学者项目或经中国学术 co-PI 走国家重点实验室开放课题提联合项目。
3. **SHLAB 合作路径**:经 Intern Robotics Workshop 或浦江会议 track 接触 OpenRobotLab(庞江淼/曾嘉)—无公开 open-call,社交驱动。
4. **最该参加的单场活动**:**智源大会 2026,6/12-13,北京**——两大实验室与整个中国具身AI 生态俱在。

## 置信度
- **高**(≥2 源含官网核验):所有 PI 名/头衔;RoboBrain 2.0/RoboOS 2.0/ORCA;InternVLA-M1(arXiv 2510.13778);EchoMimic(arXiv 2407.08136,蚂蚁主导);悟道 1.0/2.0 历史;FlagOpen/Linux Foundation。
- **中**:卢策吾 vs 庞江淼确切现任头衔(皆称"浦器 head"——大概率卢=教研/SJTU 侧,庞=in-house 运营 lead)。
- **未确认/需直接核 arXiv**:InternVLA-A1 arXiv 2601.02456(仅聚合见);InternVLA-N1(无论文 ID,仅 repo);任何独立 SHLAB "InternSpatial"/"Landmark robot" 产品名(疑为 InternVLA-M1 空间模块的非正式称呼)。

**工具限制注**:本会话 WebSearch 触预算、WebFetch 对 arxiv.org/baai.ac.cn 被屏蔽,故 arXiv 摘要页未能直接载——ID/标题取自搜索元数据,建议普通浏览器访问 URL 复核。两个标记 arXiv ID(2510.13778、2407.08136)经多独立源佐证,可靠。
