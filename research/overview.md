# Vision Pro 具身 AR 虚拟角色 — 立项综合报告(v2 精简版,2026-08-03)

> **1 页执行摘要**——立项决策的单一入口。架构对齐最新决定(大脑/小脑双系统 + 三层 subsumption + 多时间尺度,见 architecture-log)。**深挖看分文件,索引见 [README.md](README.md)。** v1 的"离散 token VLA 当脑"框架**已被连续动作 VLA 决定取代**(architecture-log 结论12 / open-vla),本文已更新。

---

## 0. 执行摘要

**项目命题**:在 Apple Vision Pro 上做一个 **AI 驱动的虚拟角色**,像真人一样**感知真实环境与你**(看房间、看你的屏幕、听你说话、读懂情绪/注视/手势),**产生反应**(说话、动作、情绪),以 AR 形式显示——"身体"是虚拟建模角色(Blender),**不是物理机器人**。本质 = **把具身智能的感知-认知-动作闭环放进一个虚拟身体**,用 Vision Pro 的真实传感器当它的眼耳。

**核心判断(三句话)**:
1. **技术上可行,且 2026 正处于多个前沿收敛点**——你的硬件(RTX Pro 6000 工作站 96GB + M1 Max 32GB + Vision Pro)是自托管、低延迟、全隐私方案的甜点配置。
2. **关键使能器是 visionOS 企业摄像头权限**(你已在申请):把"空间感知"提升为"视觉感知"。$99 普通开发者账号即可申请 "Development Only" 单机原型,**但用它的 app 不能上消费 App Store**(仅 B2B)。〔visionos-capabilities.md〕
3. **最大差异化与风险点**:让角色**贴真实家具坐/躺/趴**(愿景核心)+ **跨会话物体恒常性**。无现成解,需自研组合——也正是跟高校/投资人谈合作时最该亮出的命题。

**推荐范式(最新)**:**大脑/小脑双系统(GR2/GR00T 范式)**——大脑=具身推理 LLM(人格/对话/意图/规划,泛化"该干嘛");小脑=**连续动作 VLA**(π0/π0.5/GR00T N1.7,在你 rig 关节空间微调,泛化"怎么动");下方还有行为基线 + <200ms 反射层。〔architecture-log 结论12 / open-vla〕

---

## 1. 愿景(北极星)〔memory/project-vision-scenarios〕

角色**像室友一样栖居你的真实空间**:坐你的躺椅、躺你的床、趴你的桌、**你敲代码时盯着你的虚拟屏幕**。这不是花絮,是**架构验收清单**——它要求 affordance 感知、几何贴合姿态、防穿模、持久锚定、屏幕感知、行为状态机全部到位。

---

## 2. 推荐端到端架构(多时间尺度 + 三层 subsumption)

```
┌─────────────────────── Apple Vision Pro(显示 + 原生传感器)───────────────────────┐
│  passthrough 真房间 + 虚拟角色渲染 @90Hz(纯动画平滑,不经大脑)                     │
│  传感:SceneReconstruction mesh、WorldTracking+WorldAnchor、Hand/Eye/Face 追踪、    │
│       CameraFrameProvider(企业,864×704 立体 L/R ~60fps,只像素无深度/分割)        │
└───────────────────────────┬──────────────────────────────────────────────────────┘
                              │ 30Hz ingest(原始帧/mesh/手眼脸/Mac 截屏)
                              ▼  ───────── LAN(强烈建议 10GbE,<5ms 单程)─────────
┌─────────────────── M1 Max 32GB Mac(编排 + 渲染 + 本地反射)────────────────────────┐
│  渲染 90Hz(RealityKit)│ 快反射 <200ms(本地:注视 IK / barge-in / 重心)            │
│  持续行为基线(idle/呼吸/眨眼/微动,大脑无指令时撑"活着")                            │
│  action→animation 映射(VLA 关节轨迹 → IK/blendshape/navmesh,含贴家具拟合)        │
└───────────────────────────┬──────────────────────────────────────────────────────┘
                              ▼
┌─────────────────── RTX Pro 6000 96GB 工作站(大脑常驻跑热,7×24)───────────────────┐
│  感知 5–10Hz:SAM3.1 / VLM / 屏幕-VLM / 情感融合 → 更新场景图                        │
│  认知 1–5Hz ┌─ 大脑:具身推理 LLM(人格/对话/意图/规划)→ 意图 + 语音               │
│             └─ 小脑:连续动作 VLA(π0/GR00T,感知→rig 关节轨迹)                   │
│  持久动态场景图(物体恒常性)+ ForesightService(常驻前瞻预判)                      │
└───────────────────────────────────────────────────────────────────────────────────┘
```
五个频率分层,认知与渲染解耦:90Hz 渲染 / 30Hz ingest / 5–10Hz 感知 / 1–5Hz 认知 / <200ms 事件反射。200–500ms 刻意反应读起来**自然**。

**角色的"视觉" = 视角无关的 3D 场景图**(结构化文本喂大脑),不是 POV 像素;仅"凑近看"才渲染 POV。**物体恒常性**:世界=你走过之处的累积,记忆=信念/规划,实时感知=交互时核验(挪了躺椅就重核)。〔architecture-log 结论7/8/9〕

---

## 3. 三端算力切分〔visionos-capabilities / open-vla / scene-memory / user-perception-dialogue〕

| 端 | 角色 | 关键负载 |
|---|---|---|
| **RTX Pro 6000 工作站 96GB** | 常驻大脑(7×24) | 大脑 LLM(Qwen3/DeepSeek/GLM)+ 小脑连续 VLA(π0/GR00T)+ VLM(Qwen3-VL/UniVR-34B)+ SAM3.1 + 屏幕 VLM + 流式 TTS + 场景图/时序KG/向量库。常驻跑热的有 Audio2Face/手势生成/生成式 idle/ForesightService;70B 深推理按需。 |
| **M1 Max Mac 32GB** | 编排 + 渲染 + 端侧低延迟 | RealityKit 渲染 + IK/接触拟合 + action→animation 映射;端侧 ASR/感知;ScreenCaptureKit;本地 VAD/barge-in;小 VLM fallback(MLX/CoreML)。 |
| **Vision Pro** | 显示 + 传感 | 企业立体帧 + mesh + 手眼脸追踪 + WorldAnchor;RealityKit 渲染。 |
| **网络** | LAN | 10GbE 强烈建议;p95 往返 15–40ms,支撑 <500ms 反应环。全留 LAN,全隐私。 |

---

## 4. 分阶段路线图

| 阶段 | 目标 | 交付物 | 依赖 |
|---|---|---|---|
| **P0 现在** | 跑通核心链路 | 占位 rig + 纯 mesh 感知 + 工作站对话 LLM + Apple Speech/TTS;感知→认知→动作→RealityKit 渲染闭环;工作站起 OpenPI 推理 + 定 v1 rig 关节动作空间 | 无(今天可做) |
| **P1 摄像头+屏幕** | 角色"看见"房间与屏幕 | 企业摄像头→SAM3.1→语义场景图;屏幕感知通道;反应式对话 <500ms | entitlement 到位 |
| **P2 愿景核心** | 坐躺椅/躺床/趴桌 + 记住房间 | 贴家具 affordance 姿态拟合(自研);跨会话物体恒常性;首次 VLA LoRA 微调;共话语手势 + lip 同步 | P1 + 动画 R&D |
| **P3 产品化** | 对外谈合作/报项目 | 行为丰富度、干净 IP 资产(外包)、企业/B2B 分发、学术/资金对接 | P2 + 美术 + 合作 |

---

## 5. 关键风险(排)
1. **贴真实家具的 affordance 姿态合成**(P2 核心)——无现成解,需 检索+IK+接触优化+物理 组合。〔animation-affordance〕
2. **跨会话物体恒常性**——visionOS 无语义身份 API,需自建关联管线。〔scene-memory〕
3. **企业 entitlement 商业天花板**——用主摄像头的 app 不能上消费 App Store,仅 B2B。〔visionos-capabilities.md〕
4. **VLA 仍需微调**——所有基础 VLA 对没见过的身体零样本都不行;但模型/感知/跨本体机制现成,只训你 rig 关节空间这段。〔open-vla / data-and-iteration〕
5. **跨 Mac+工作站+VP sub-200ms barge-in** + 干净 AEC。〔user-perception-dialogue〕
6. **IP 干净度**——游戏解包资产仅可丢弃式早期原型;对接前必须替换为合规资产。〔memory〕
7. **Apple API 变化节奏**——visionOS 26/27 快速演进,架构须留接口可降级。

---

## 6. 国内产学研对接(优先级)〔china-academic-map.md / china-industry-and-niche.md〕

**技术合作首选**:① 清华 刘烨斌(自动化,数字人/Gaussian Avatar,据报道被 Apple 采用)/ 赵昊(AIR,SyncTalk++/Ultraman,招实习);② 北大 王鹤(EPIC Lab)+ 银河通用 Galbot(VLA,招实习);③ 智源 BAAI FlagOpen(已入 Linux Foundation,**最低摩擦**)+ RoboBrain/RoboOS;④ 上海 AI Lab OpenRobotLab(InternVLA 空间 grounding);⑤ 阿里 MNN-TaoAvatar 团队(唯一已上 Vision Pro 演示的中国团队)。

**资金/通道**:国家重点研发计划、智源学者 3.0、国家重点实验室开放课题。**产业伙伴**:魔珐 Xmov(有 VP demo+SDK)、相芯 FaceUnity、影眸 Rodin(3D 资产生成)、世优(动捕)。

**重要事实**:精确利基"AR+具身AI+虚拟角色,中国,Vision Pro"在产品级**基本空白**=你的机会,但所有集成得自己做。

---

## 7. 深挖指引

| 想看 | 文件 |
|---|---|
| 架构权威 + 开放问题 | [architecture-log.md](architecture-log.md) |
| 动作/场景图契约 | [contracts-v1.md](contracts-v1.md) |
| VLA 选型 + 适配(连续) | [open-vla.md](open-vla.md) |
| 数据与迭代 | [data-and-iteration.md](data-and-iteration.md) |
| 竞品定位 | [competitive-positioning.md](competitive-positioning.md) |
| 感知/场景图/记忆 | [scene-memory.md](scene-memory.md) |
| 感知用户/屏幕/对话 | [user-perception-dialogue.md](user-perception-dialogue.md) |
| 动画/affordance | [animation-affordance.md](animation-affordance.md) |
| 世界模型判定 | [world-model-verdict.md](world-model-verdict.md) |
| visionOS 能力(摄像头+锚点) | [visionos-capabilities.md](visionos-capabilities.md) |
| 中国学术地图 | [china-academic.md](china-academic.md) |
| 中国产业/SDK/利基 | [china-industry.md](china-industry.md) |
| **完整索引** | [README.md](README.md) |
