# 五条 track 的从零里程碑细化(2026-08,工作组核验产出)

> 配套 [roadmap-phased.md](roadmap-phased.md)(策略 + 关键路径)。本文是 5 条 track 的里程碑级拆解 + 核验到的 Apple 原生/库现状 + 风险。所有外部事实 2026-08 web 核验,标 ✅/⚠️。里程碑标 [阶段/复杂度]。

## ⚠️ 关键核验发现(影响计划,必读)
1. **NavMesh(`NavigationMeshResource/Component/Controller`)是 visionOS 27 beta(WWDC26 sess 279),不是 vOS26** → Phase A sim 用 clip/stub 兜底,locomotion 上 NavMesh 推迟到 vOS27。✅
2. **RealityKit 无第一方离屏 render-to-texture API** → 连续第一人称 POV 渲染(架构结论 13③)是**真实 R&D 缺口**;Mac 用 `ARView.snapshot()` 轮询 / 自写 Metal 离屏 pass 兜底,VP 更难,可能降分辨率/事件触发。⚠️
3. **RealityKit 无原生动画状态机/Animator**(无 Mecanim)→ 必须在 `AnimationPlaybackController`+`BlendTreeAnimation` 上**自建**状态机。vOS27/RCP3 出了原生 Animation Graph+State Machine(WWDC26 sess 393),但运行时 `triggerTransition()` API 文档稀疏("still emerging")→ Phase A 自建,vOS27 再评估迁移。⚠️
4. **`SimulationBody` 不是真实 API** → "坐真家具"物理沉降用 `PhysicsBodyComponent`+`SceneUnderstandingComponent` 的 physics/collision(vOS26 GA,让 scene mesh 参与碰撞)。✅
5. **HOV-SG 是 RSS2024 研究代码,非维护库** → 2D→3D lift 须**自行移植/重写**。⚠️
6. **无开源 VLA 开箱输出"虚拟 rig 关节空间"** → π0/GR00T N1.7 是 EEF/相对位姿,非关节向量;自定义适配层(数据/动作空间差距)是 motion track **最大不确定性**。⚠️
7. **GR00T N1.7 LoRA recipe 有 bug**(Reddit 有 workaround)→ **π0/π0.5 微调管线更成熟,首选**;GR00T 备选。⚠️
8. **LangGraph 安全**:`Redis checkpointer RCE(CVE-2026-27022,<1.0.10)`+ CSA checkpoint 两漏洞(2026-03)→ 须补丁≥1.0.10/加密后端/不外暴露。⚠️
9. **Graphiti 偏好 OpenAI Structured Output** → 本地 vLLM 须开 guided-decoding(xgrammar);且 Issue #1116 OpenAI provider 忽略 api_base → 须显式 custom LLM client。⚠️
10. **ForesightService 无现成产品** → 自建(参考 AHEAD arXiv:2607.15172),加 guardrail(预测只影响微反应/注视预备,不直接驱动粗动作,可关闭)。⚠️
11. **visionOS 27/macOS 27 Spatial Preview 框架** → Mac app 推空间内容到配对 VP 双向实时同步,可作 **Phase C(A→AR 出)低摩擦迁移桥梁**,记为 leverage。✅
12. **企业 entitlement 卡 Phase B 全部**(ARKit 真实感知 P6-P10、BM11)— 申请中($99)。Phase A 不受影响。✅
13. **真实遮挡 = Apple 现成**:`OcclusionMaterial`(不可见材质,藏住其后虚拟物)+ `SceneReconstruction` mesh → 真实家具/墙正确遮挡角色(官方指南 "Obscuring Virtual Items Behind Real-World Items")。**经典 AR 遮挡难题开箱解决,大减风险**;非全自动(须把 OcclusionMaterial 套到重建 mesh,墙遮挡社区反馈得自造几何,家具级无问题)。✅
14. **rig 采用标准人形骨骼(用户决定,非自定义)**:动作空间 = SMPL-X/HumanML3D ~22–24 关节 + ARKit 52 blendshape;render rig 用 Mixamo/RPM + 标准 retarget。→ AMASS/BEAT/HumanML3D 运动数据**原生对齐**(微调数据免造);化解"无开源 VLA 输出关节空间"顾虑(微调让 action head 学人形关节旋转)。最大不确定性 #1 从"自定义骨骼+适配"降到"就是微调"(通用成本)。✅

---

## Track 1 · 基础设施 + 最小闭环骨架(infra & spine)— 关键路径起点
| # | [阶/复杂] | 里程碑 | 依赖 | Apple原生/库 |
|---|---|---|---|---|
| IS-1 | [A/低] | Xcode visionOS 工程 + RealityKit-on-Mac sim 场景(Simulator@Mac 主路径,macOS 原生回退);出 DEVENV.md 记 Simulator 传感器边界 | — | Xcode 26 / visionOS 26 SDK / RealityView / Reality Composer Pro |
| IS-2 | [A/低] | 5 端口 Swift protocol + CompositionRoot(PerceptionSource/Display/Brain/MotionResolver/MemoryService,对齐契约 A/B/C) | IS-1 | Swift protocol/Codable/Combine/AsyncSequence |
| IS-3 | [A/低] | 最小端到端闭环(stub 全栈)在 Mac 跑通 | IS-1,2 | RealityKit Entity/AnimationPlaybackController + RealityView |
| IS-4 | [A/中] | 工作站推理服务器骨架 + LAN 链路(OpenPI `serve_policy.py` ws8000 + vLLM HTTP;Mac `URLSessionWebSocketTask` 连通;ping/echo 验 10GbE) | IS-1 | URLSessionWebSocketTask/URLSession + OpenPI + vLLM |
| IS-5 | [A/中] | 集成/替换 harness + 可观测总线(stub↔真可配置注入 + 结构化事件流 JSONL/dashboard + 冒烟测试) | IS-2,3 | Combine/AsyncSequence + os.Logger/OSSignposter + XCTest |
**风险**:① Simulator 不能真建图/手眼/锚定(by design,脚本兜底);② OpenPI 仅测 Ubuntu 22.04(钉死或 Docker);③ OpenPI action 空间 per-robot,本 rig 非支持机器人→自定义客户端+关节空间微调(归 motion);④ 96GB 同驻 vLLM+OpenPI+SAM/VLM 争用→各服务 `--gpu-memory-utilization` 预留;⑤ LAN ws 须自实现重连/心跳/背压。

## Track 2 · 感知 + 场景图记忆(perception & memory)
| # | [阶/复杂] | 里程碑 | 依赖 | 技术 |
|---|---|---|---|---|
| P-1 | [A/低] | PerceptionSource 端口 + sim stub(异步吐 Observation) | IS-2 | Swift protocol |
| P-2 | [A/高⚠️] | Sim 感知生成器:角色眼虚拟相机渲 POV + 合成 MeshAnchor + 脚本手眼脸(POV 连续离屏渲染是难点) | rig,房间,P-1 | RealityKit PerspectiveCamera + snapshot/Metal 离屏⚠️ |
| P-3 | [A/中] | Sim 场景图搭建器(契约 B:sim 干净走简化 HOV-SG) | P-2 | Swift + RealityKit 几何 |
| P-4 | [A/中] | MemoryService:Graphiti(时序 Neo4j)+ Milvus/BGE-M3 持久场景图(sim/真实共用) | P-3 | 工作站 Python + ws |
| P-5 | [A/中] | 相关性编译器:挑相关子图→紧凑文本喂大脑;为 motion 暴露 part/affordance/SDF_ref 查询 | P-3,4 | Swift + Graphiti/Milvus |
| P-6 | [B/高] | ARKit 真实感知接入(SceneReconstruction+Plane+Room+World+Hand+Eye+**CameraFrameProvider 企业 864×704**;世界=走过累积无预扫描) | **entitlement⚠️**,P-1,VP | ARKit providers + main-camera-access |
| P-7 | [B/高] | SAM3.1 + VLM 2D 感知管线(实例分割+开放词表标注+CLIP) | P-6,GPU | SAM3.1(Meta 2026-03)✅ + Qwen3-VL⚠️未核版本 |
| P-8 | [B/高⚠️] | 2D→3D lift 到 mesh(反投影 mask→per-face 标签→聚合 3D 实例,HOV-SG 式) | P-6,7 | HOV-SG 移植/重写⚠️ + Open3D |
| P-9 | [B/高] | 真实场景图融合 + 物体恒常性(WorldAnchor UUID 跨会话;走过留图/路过刷新/新房间实时搭) | P-8,4 | Graphiti+Milvus |
| P-10 | [B/中] | 执行前核验(staleness>θ/confidence<φ 触发局部重感再执行) | P-9 | Swift hook + 工作站重感 |
| P-11 | [C/高⚠️] | 角色第一人称 POV 渲真实 mesh(双视觉通道,喂小脑 VLA+VLM) | P-6,rig | 离屏 render⚠️(同 P-2) |
| P-12 | [D/中] | 感知去噪/对齐/降级鲁棒(sim→真实 gap:mesh 膨胀/遮挡/光照) | P-9,10,11 | Swift + 工作站管线 |
**风险**:① RealityKit 离屏 POV 是真实缺口(见发现 2);② 用户脸捕获不可得(VP 外摄像头看不见用户自己)→情绪走语音+注视+手势;③ entitlement 卡 P6-P10;④ HOV-SG 非维护库(发现 5);⑤ CameraFrameProvider 无烘焙深度(立体深度自算,但几何免费来自 SceneReconstruction);⑥ 契约交叉点一致性(uuid/PartNode 命名 seat/backrest)须跨 track 早期钉死。

## Track 3 · 运动:小脑(连续 VLA)+ 动画通道 + 运动解析器(motion)— ★rig 是 gating★
| # | [阶/复杂] | 里程碑 | 依赖 | 技术 |
|---|---|---|---|---|
| **M-1** | **[A/中★GATING]** | **Rig = 采用标准人形骨骼(非自定义,用户决定)**:SMPL-X/HumanML3D ~22–24 关节+根 6-DoF+ARKit 52 blendshape 作动作空间;render rig 用 Mixamo/RPM+标准 retarget;定 chunk K(~16-64)+IK 锚点;验 RealityKit 可控。**标准骨骼→运动数据原生对齐、微调数据免造(见发现 14)** | IS stub | AnimationLibrary/BlendTree/IKComponent/USDZ |
| M-2 | [A/低] | 运动解析器 port(stub→多通道仲裁:通道+priority+interrupt+冲突压制) | M-1 | AnimationPlaybackController+BlendTree(自建仲裁) |
| M-3 | [A/中] | 行为基线(呼吸/眨眼/fidget/瞥视/情绪微表情)+ 注视 IK(look-at) | M-1,2,perception | IKComponent + BlendTree |
| M-4 | [A/中] | 脸部通道 Audio2Face-3D(音频→52 ARKit blendshape,Mac 解码@90Hz 插值,Apple Silicon 兜底) | M-1,2,brain TTS | BlendShapeAnimation + CoreML/MLX |
| M-5 | [A/中] | 片段库 + **自建**动画状态机(无原生!)+ MotionLCM in-betweening | M-1,2 | AnimationLibrary+BlendTree(自建状态机) |
| M-6 | [A/高] | 小脑数据管线 + OpenPI 服务骨架(采感知+原始 rig 信号存 LeRobot v2;冷启动 authoring/LLM 标注/蒸馏 BEAT/MEAD/AMASS)— 长 sub-track | M-1,perception,brain | OpenPI + LeRobot v2 |
| M-7 | [A/高⚠️] | LoRA 微调(rig 关节空间)+ action chunk→@90Hz 桥(receding horizon) | M-6,1,2 | **π0/π0.5 首选**(成熟)/ GR00t N1.7 备(LoRA 坑⚠️) |
| M-8 | [C/中] | NavMesh locomotion(从 SceneReconstruction mesh 生成寻路)— **vOS27 beta**⚠️ | M-5,mesh | NavigationMeshResource/Controller(vOS27 beta) |
| M-9 | [C/高★最高R&D] | affordance-fit:HOSIG/CHOIS 粗姿态→PhySIC 接触穿透 QP(对 SDF)→RealityKit 物理沉降→IKcomponent 精修;每家具 contact_offset | M-1,5,mesh,memory | PhysicsBodyComponent+SceneUnderstandingComponent physics(vOS26 GA) |
| M-10 | [D/高] | 共话语手势(重定向到就坐角色)+ 行为导演 + 活感打磨 | M-3,4,5,7,brain | BlendTree 多通道混合 + IKComponent |
**风险**:① NavMesh 是 vOS27 beta(发现 1);② SimulationBody 非真实 API(发现 4);③ 无开源 VLA 输出 rig 关节空间(发现 6)→适配层最大不确定性;④ GR00T LoRA bug(发现 7);⑤ ARKit mesh 膨胀→高坐 1-3cm(调 contact_offset);⑥ 共话语手势训于站立者→过滤穿椅臂+下身 IK 钉;⑦ 无原生动画状态机(发现 3);⑧ Audio2Face-3D Apple Silicon 端口社区维护(非 NVIDIA 官方)。

## Track 4 · 大脑:具身推理 LLM + 记忆 + 编排 + 对话(brain,工作站侧)
| # | [阶/复杂] | 里程碑 | 依赖 | 技术 |
|---|---|---|---|---|
| BM-0 | [A/低] | Brain 端口协议 + stub(输出契约 A Decision JSON) | IS,契约 A | Swift protocol |
| BM-1 | [A/中] | vLLM LLM 骨干(默认 Qwen3-32B;guided-decoding 绑契约 A JSON;首 token<300ms) | 工作站 | vLLM v0.26 + Mac URLSession |
| BM-2 | [A/中] | LangGraph StateGraph + checkpointer(Perceive→Interpret+Recall→Reason→EmitAct;跨 tick 持久 emotion/intent/persona) | BM-1 | LangGraph 1.1.6⚠️CVE + checkpoint |
| BM-3 | [A/中] | 记忆后端:Graphiti 时序 KG + Milvus/BGE-M3 向量(Graphiti Pydantic 映射契约 B) | 契约 B,BM-1 | Graphiti 0.29.3⚠️ + Neo4j 5.26 + Milvus 2.5 |
| BM-4 | [A/高] | 记忆自管工具层(perceive/recall/reflect/forget)+ 相关性编译器 | BM-2,3 | Python over Graphiti/Milvus |
| BM-5 | [A/高] | 工具调用:VLM 当眼 + 场景图查询 + SayCan(选动作×affordance 接地)+ 插件发现 | BM-2,4,perception VLM | Qwen3-VL + ReAct |
| BM-6 | [A/中★收口] | 大脑→小脑 意图编译器(Decision 子集→NL token-prefix 串喂 VLA;未微调→clip/IK 兜底) | BM-2,motion M-1 | Python 编译器 + ws→serve_policy.py |
| BM-7 | [A/高] | 执行前核验/接地/抗幻觉(SayCan×affordance + staleness 重感 + Reflexion 写 Graphiti) | BM-2-5 | Python over Graphiti |
| BM-8 | [A/高] | 流式对话 + barge-in(中文,vLLM 流式<300ms;本地<200ms barge-in) | BM-1 | vLLM 流式 / vLLM-Omni + Mac AVAudioEngine |
| BM-9 | [A/中] | **Phase A 大脑出口**(集成闸:sim 闭环验收听/环顾/坐/记忆/中文 barge-in) | BM-2-8 | sim + 工作站集成 |
| BM-10 | [B/中] | 真实感知接地 + 物体恒常性(WorldAnchor uuid+Graphiti 时序) | entitlement,BM-3,6 | ARKit WorldAnchor + Graphiti |
| BM-11 | [C/中⚠️] | POV 作主动感知通道喂 VLA+VLM(定连续 vs 事件 vs 降分辨率) | Display 切 VP,BM-5 | 离屏 render⚠️ |
| BM-12 | [D/高] | 连续内心独白 + ForesightService(预判→注入活跃意图) | BM-2-8 稳固 | LangGraph 后台调度 |
| BM-13 | [D/中] | 人格持久化 + 鲁棒降级 + 安全(降级链;工具白名单;敏感词) | BM-2-4 | Graphiti 人格态 + MLX 端侧 fallback |
**风险**:① Graphiti 偏好 OpenAI Structured Output(发现 9);② vLLM guided JSON 可能拒复杂 schema(Issue #15236);③ 96GB 同驻紧→深推理 70B 按需起+卸载策略;④ Graphiti 0.30 rc→锁 0.29.3;⑤ **大脑模型优先级未决**(text+TTS vs Qwen3-Omni S2S)→须早 benchmark;⑥ 中文 S2S 未盲测;⑦ LangGraph CVE(发现 8)。

## Track 5 · 交互插件 + 活感控制层(interaction & alive)— 非关键路径
| # | [阶/复杂] | 里程碑 | 依赖 | 技术 |
|---|---|---|---|---|
| IA-1 | [A/中] | 分层控制骨架 + InteractionPlugin 协议/注册表/生命周期(trigger/precond/actions/binding/exit) | IS,rig,契约 A/B | Swift + AnimationLibrary |
| IA-2 | [A/中] | Behavior-baseline 状态机(idle/呼吸/眨眼/微动,常驻) | rig,IA-1 | BlendTree + 自建状态机 |
| IA-3 | [A/中] | Reflex v1(<200ms:注视 IK / barge-in(Silero VAD v5)/ 接近姿态) | rig,IS,IA-1 | IKComponent + Silero VAD v5 ONNX + AVAudioEngine |
| IA-4 | [A/高★最高风险] | Behavior-director v1(仲裁:优先级/中断/混叠,实现契约 A 冲突模型) | IA-1,2,3,MotionResolver,Brain | 自建 Swift(无库) |
| IA-5 | [A/中] | v1 插件集(sim:坐/躺/趴/看屏/靠近你/环顾) | IA-1,场景图 affordance,clip+IK,IA-4 | AnimationLibrary+IKComponent |
| IA-6 | [A/中] | 连续微反应层 v1(你移动→朝向/重心;注视跟随;对话→呼吸眨眼同步) | IA-3,2,感知总线,IA-4 | ARKit tracking + blendshape/IK + Swift |
| IA-7 | [A/低] | Phase A 出口门(活感可见 + 插件响应 + director 仲裁 + reflex 顺滑) | IA-1-6 | 综合 |
| IA-8 | [B/高] | 真实感知接入改造 + 完整 barge-in(真实麦克风+AEC) | PhaseA,entitlement | ARKit + Silero VAD + AEC |
| IA-9 | [C/高] | Posture 插件 binding 迁移:真实家具 affordance-fit(binding 抽象保证小脑微调后可切契约 C) | IA-5,animation PhaseC | 物理碰撞 + IK + SDF QP |
| IA-10 | [C/高⚠️自建] | ForesightService(连续前瞻→提前反应) | IA-8,大脑,IA-4 | 无现成产品(发现 10) |
| IA-11 | [D/高] | 导演层丰富化 + 抗恐怖谷标定 +(可选)迁 RCP3 Animation Graph(vOS27) | IA-4 | RCP3 vOS27 sess393 |
| IA-12 | [D/中] | 连续 backchannel + 共话语手势集成(就坐重定向) | IA-2,TTS,gesture,Audio2Face | AVSpeechSynthesizer + Audio2Face-3D + IKComponent |
**风险**:① director 是本 track 最高风险(自建仲裁+抗恐怖谷人感调参,易"连续但抽搐"或"克制但死板");② vOS27 Animation Graph 运行时 API 稀疏→自建状态机长期兜底;③ 插件 binding 的"小脑 vs fallback clip"双路径须 Phase A 早期定接口(强依赖 brain 收口);④ 跨 Mac+工作站+VP 的 <200ms barge-in 工程最难(AEC+置信 partial+音素边界 mid-utterance 取消);⑤ "看屏"依赖屏幕感知(跨 track 最难点,可能延后 Phase B 的真实 WatchScreen)。

---

## 跨 track 关键路径(综合)
```
IS-1 环境 → IS-2 端口 → IS-3 stub 闭环 → IS-4 工作站/链路 → IS-5 harness
                                  │
                                  └─► M-1 ★Rig 定义★(gating:motion+契约C+小脑)
                                        ├─► P-2 sim POV、IA-2 baseline、IA-3 reflex 可动手
                                        └─► M-2 resolver → M-3/4/5 通道
                                  │
   并行(M-1 后):P-3..5 场景图记忆 | BM-1..9 大脑 | M-6/7 小脑(长sub-track) | IA-1..7 插件+活感
                                  │
                                  └─► Phase A 出口(M-1+IS+各track的 Phase A 里程碑齐)
                                        │
                                        └─► Phase B(entitlement 到位):P-6..10 / BM-10 / IA-8
                                              └─► Phase C:Display 切 VP + M-8/9 + IA-9/10 + P-11/BM-11(POV)
                                                    └─► Phase D:BM-12/13 + IA-11/12 + M-10 + P-12(活感打磨)
```
**最大三个不确定性**(决定成败):① ~~Rig 动作空间~~ → **rig 已定为标准 SMPL-X/HumanML3D 骨架**(数据原生对齐);剩**小脑微调本身的通用成本**(不确定性已降);② affordance-fit(M-9,贴真家具无端到端解);③ behavior-director(IA-4,自建+抗恐怖谷调参)。**遮挡已由 Apple `OcclusionMaterial` 开箱解决(发现 13),不再是风险。**
**Week 1 零阻塞**:IS-1/2/3/4 + M-1(rig sourcing/锁关节集)。其余 Phase A 在 M-1 后并行铺开。
