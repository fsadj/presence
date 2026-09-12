# research/ 索引(v2,2026-08-03)

> Vision Pro 具身 AR 虚拟角色 · 立项研究合集。**从 [`overview.md`](overview.md) 开始读**——它是 1 页执行摘要,综合了下面所有文件。

## 怎么读
1. **先读 [`overview.md`](overview.md)**:命题 + 推荐架构 + 算力切分 + 路线图 + 风险 + 国内对接。
2. **架构权威看 [`architecture-log.md`](architecture-log.md)**:立项讨论钉死的认知架构(大脑/小脑双系统、三层 subsumption、多时间尺度、场景图即认知、物体恒常性)+ 待决策项。**与 overview 有出入处,以 architecture-log 为准。**
3. 有兴趣深挖某块再翻对应分文件(下表"一句话要点")。

> 📌 **2026-08-03 整理**:原 22 个散文件已去冗余合并为 **13 个**(国内 9→2;01+04→1;12 并入 open-vla;00 刷新),并统一为描述性英文名。原始散文件归档在 [`_archive/`](_archive/),可逆。状态:✅=联网核验;⚠️=待核;📝=设计纪要。

---

## 文件清单(分层)

### 🏛 基础(Foundation)
| 文件 | 一句话要点 | 状态 |
|---|---|---|
| [**overview**](overview.md) | **capstone**:1 页执行摘要——命题/推荐架构(大脑/小脑)/算力切分/路线图/风险/对接 | 综合 |

### 🧠 架构 / 设计纪要(权威,当前思路)
| 文件 | 一句话要点 | 状态 |
|---|---|---|
| [**architecture-log**](architecture-log.md) | 📝 多时间尺度环 / 三层subsumption / **大脑(具身推理LLM)+小脑(连续VLA)** / 场景图即认知 / 物体恒常性(记忆=信念+实时=核验)+待决策项 | 📝 |
| [contracts-v1](contracts-v1.md) | 📝 数据契约全集 A–E:A 大脑意图 / B 场景图 / C 小脑动作空间 / D 感知 Observation+管线 / E 传输+仲裁+插件+记忆+时序衔接(**Phase A 可开工**) | 📝 |
| [competitive-positioning](competitive-positioning.md) | 📝 竞品对比;差异化=真实房间grounding+持久记忆+贴家具+屏幕感知+原生Swift+VLA锚定 | 📝 |
| [open-vla](open-vla.md) | ✅ 连续动作VLA(π0/GR00T,标准 SMPL-X 骨骼)首选;**附 B:适配层+微调最小化+未来成熟 VLA 赌注**(Phase 1 零微调,小脑 LoRA 可选延后);含早期 VLA 历史附录 | ✅ |
| [data-and-iteration](data-and-iteration.md) | 📝 数据规格+现成集(AMASS/BEAT/MEAD)+缺口自采+两阶段+换新VLA重微调+自我迭代闭环 | 📝 |
| [world-model-verdict](world-model-verdict.md) | ✅ 结论:今天世界模型不简化架构(领域往"显式记忆+基础模型"收敛);窄采用为ForesightService | ✅ |
| [roadmap-phased](roadmap-phased.md) | 📝 分阶段研发:sim-first(RealityKit-as-sim)+ A→D + 端口驱动迁移 + Apple 原生映射 | 📝 |
| [roadmap-tracks](roadmap-tracks.md) | ✅ 5 条 track 从零里程碑细化(infra/感知/大脑/运动/交互)+ 核验到的 API 现状与风险(12 条关键发现) | ✅ |
| [phase-a-preflight](phase-a-preflight.md) | 📝 Phase A 开工前准备清单(勾选式):账号/申请、模型下载、环境、占位 rig、待拍板决策 | 📝 |
| [phase-a-build-log](phase-a-build-log.md) | 📝 Phase A 实测构建日志:VRM→RealityKit 管线 + 每个坑的修法(法线/metallic/去重/补灯)+ 脸通道(blendshape 绑定)+ 卡通 R&D | 📝 |

### 🔬 技术研究(Research / building blocks)
| 文件 | 一句话要点 | 状态 |
|---|---|---|
| [visionos-capabilities](visionos-capabilities.md) | ✅ 企业摄像头($99可申请,864×704立体,无深度/分割,B2B)+锚点持久(WorldAnchor) | ✅ |
| [scene-memory](scene-memory.md) | ✅ UniVR-34B/SAM3/HOV-SG/3D-Mem/Zep;分层场景图按anchor UUID;最难点=物体恒常性 | ✅ |
| [user-perception-dialogue](user-perception-dialogue.md) | ✅ 屏幕感知(GUI agent改"看屏",事件触发);混合流式对话<500ms;中文端到端强 | ✅ |
| [animation-affordance](animation-affordance.md) | ✅ 无2026端到端解,需HOSIG/CHOIS→PhySIC接触优化→RealityKit物理接地;WWDC26 NavMesh+IKcomponent;Audio2Face-3D开源ARKit52 | ✅ |
| [brain-agent-architecture](brain-agent-architecture.md) | ✅ 大脑调研:LangGraph编排+Graphiti/Milvus记忆+ReAct/SayCan;**大脑→小脑用 NL 意图串 token-prefix**(π0/GR00T 原生 prompt,免适配器);VLA 只扛身/大动作,细通道走契约A | ✅ |

### 🇨🇳 中国生态(China ecosystem)
| 文件 | 一句话要点 | 状态 |
|---|---|---|
| [china-academic](china-academic.md) | ✅ 清华/北大/上交·中科院·中科大/上海AI Lab·智源 —— PI/实验室/项目/招实习/对接优先级 | ✅ |
| [china-industry](china-industry.md) | ✅ 8家数字人公司 + SDK平台表 + HF资产 + AR具身利基(MNN-TaoAvatar居中) | ✅ |

### 📦 归档
| 路径 | 内容 |
|---|---|
| [`_archive/`](_archive/) | 12 个原始散文件,已被上方合并文件取代,保留作可逆备份。说明见 [`_archive/README.md`](_archive/README.md)。 |

---

## 关键结论速览
1. **可行**,硬件是甜点;**关键使能器=企业摄像头权限**(已申请,$99 即可原型,但不能上消费 App Store)。
2. **范式**:**大脑/小脑双系统**——具身推理 LLM(泛化"该干嘛")+ **连续动作 VLA**(π0/GR00T 在你 rig 关节空间微调,泛化"怎么动")+ 行为基线 + <200ms 反射。
3. **算力切分**:工作站=常驻大脑(大脑LLM+小脑VLA+VLM+SAM+TTS+记忆,跑热);Mac=编排+渲染+IK+端侧感知+barge-in;Vision Pro=显示+传感;**10GbE**。
4. **最大风险/差异化**:"贴真实家具坐/躺/趴"(无现成解,自研)+ 跨会话物体恒常性。
5. **最高性价比新颖通道**:屏幕感知(看你写代码),事件触发。
6. **VLA 仍需微调**(所有基础 VLA 对新本体零样本都不行),但模型/感知/跨本体机制现成,只训 rig 关节空间(LoRA 单卡可起)。
7. **国内对接**:清华刘烨斌/赵昊、北大王鹤(Galbot,招实习)、智源 FlagOpen(最低摩擦)、阿里 MNN-TaoAvatar(唯一 Vision Pro 演示)。
