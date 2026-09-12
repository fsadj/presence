# 3D 场景语义理解 + 持久/空间记忆 — 给 Vision Pro 角色(2026-08,串行,优先最新)

> 角色"真正理解并跨会话记住真实房间"的决策级调研。UniVR-34B/SAM3 场景图/时序记忆/WorldAnchor 补偿全覆盖。

## 一句话结论
你描述的能力(具身角色感知、理解、跨会话记住真实房间)**在你这套硬件上今天刚刚可行,且各部件正在快速收敛**。四件事定调:
1. **UniVR-34B 假设基本正确,一处修正**:字节 UniVR-34B(arXiv 2607.12800,2026-07)真实,~34B,**确实基于 Emu3.5**(已对论文核验;早期某网页摘要说"否则"是错的)。它在视觉 token 空间原生推理(无文本 CoT),34B 接近 Gemini-3-Pro+Nano-Banana-2。HF 开源权重 `ByteDance/UniVR-34B-Planning`,代码 github.com/bytedance/UniVR。**注意它是视觉推理/规划模型,不是 3D 房间地图模型**——给你规划器(会"想象"下一帧),不给你持久场景图。
2. **自你 spec 以来最重要的 2D 感知升级是 SAM 3**(Meta,2025-11,arXiv 2511.16719)。Promptable Concept Segmentation=开放词表文本提示检测+分割+追踪一体,精度翻倍。**把你 SAM/SAM2+Grounding-DINO+Florence-2 融合栈压成一个模型**。SAM 3.1(2026-03)加实时视频模式。
3. **3D lifting 的 2025–2026 状态 = 基于 Gaussian Splatting 的开放词表场景图**。OpenGaussian、PanoGS、LangSplatV2、OpenGS-SLAM、ReSemGS-SLAM(2026)均接近实时。HOV-SG 线 + ConceptGraphs 给你**持久场景图记忆基底**。
4. **visionOS WorldAnchor 限制真实但可补**:visionOS 无 `ARWorldMap`,只有 `WorldAnchor`(UUID+位姿),ARKit 跨会话按 ID 匹配静默重定位。所以**把语义场景图存工作站、按 anchor UUID 索引**;visionOS 管几何重定位,你的服务器管语义内容。

难点不在模型,而在:(a) 把 864×704 立体+SceneMesh 融成工作站能近实时语义融合的流;(b) 保持*物体*跨会话身份稳定(物体恒常性);(c) LAN 延迟预算。

## 1. 开放词表 3D 场景理解 & 语义 grounding(最新在前)

### 1.1 3D 感知 VLM / "空间 LLM"(最新在前)
| 模型 | 时间 | 是什么 | 对你意义 |
|---|---|---|---|
| **ByteDance UniVR-34B**(arXiv 2607.12800)| **2026-07** | ~34B,基于 **Emu3.5**;VQ-VAE 图像 tokenizer + 文本/视觉 token 统一 next-token,SFT+GRPO RL。生成**视觉推理轨迹**(未来帧)而非文本 CoT。HF `ByteDance/UniVR-34B-Planning`;github.com/bytedance/UniVR。VR-X 基准 ~+25% | 最佳开源"视觉空间思考"规划器。FP16 ~68GB 装进 96GB,余量给独立感知栈。**确认基座=Emu3.5** |
| **Emu3.5**(BAAI,arXiv 2510.26583)| 2025-10 | 原生多模态世界模型,视觉+语言联合 next-token,大规模 RL。**BAAI 非**字节 | UniVR 的基座 |
| CVPR 2026 2nd 3D-LLM/VLA Workshop(3d-llm-vla.github.io)| 2026 | 最新 3D LLM/VLA 场地 | 跟踪前沿发布 |
| Robin3D | 2025 | 新一代 3D LLM 空间智能 | 关注,细节少 |
| Spatial 3D-LLM(OpenReview)| 2025 | 渐进空间感知 3D MLLM | 比初代 3D-LLM 空间推理强 |
| **Video-3D LLM**(CVPR 2025)| 2025 | 视频 grounding 的 3D LLM | **比静态扫描 3D-LLM 更贴你实时摄像头流** |
| SD-VLM(NeurIPS 2025)| 2025 | 带深度空间测量/理解 | 与 SpatialBot 竞争的深度感知 VLM |
| SR3D(arXiv 2509.13317)| 2025-09 | 3D 区域提示 VLM | 自然语言 3D 区域 grounding |
| Perspective-Aware Reasoning via Mental Imagery(ICCV 2025)| 2025 | VLM 经心理模拟换视角 | 角色推理用户视角 vs 自己 |
| Prompt-Guided Spatial RGB-D(arXiv 2510.11996)| 2025-10 | RGB-D 提示空间 VLM | **直接贴"我有 RGB+自己深度"管线** |
| IJCAI 2025 综述:Enable LLM with 3D Capacity | 2025 | 3D 能力 LLM 综述(多视图/RGB-D/点云) | 全景地图 |
| SpatialBot(arXiv 2406.13642;ICRA 2025)| 2024/2025 | 单目深度作第二输入的 VLM+SpatialQA | 单目空间问答;有真实立体深度时弱于 RGB-D 法 |
| SpatialRGPT(NeurIPS 2024)| 2024 | 区域建议 grounding VLM;处理 box/区域 prompt | 开源最强区域 grounding 空间推理(524 引)|
| 3D-LLM/LEO/LL3DA/ChatScene | 2023–2024 | 基础 3D LLM(点云/多视图)| 基线,已被上面 2025–2026 超越 |
| SpatialVLM(Google DeepMind)| 2024 | 原生空间关系 VLM | 概念参考 |

**小节结论**:认知层 headliner = **UniVR-34B**(原生视觉空间推理,装得下)。需要*区域 grounding* 绑定场景图坐标时,SpatialRGPT/SpatialBot 是实用开源选择。

### 1.2 2D 基础模型(最新在前)
| 模型 | 时间 | 你栈中的角色 |
|---|---|---|
| **SAM 3 / 3.1**(ai.meta.com/blog/segment-anything-model-3;arXiv 2511.16719;github.com/facebookresearch/sam3)| **2025-11/2026-03** | **对你管线最大简化**。Promptable Concept Segmentation=开放词表文本提示检测+分割+追踪一体,PCS 上 2× 精度。3.1 加 Object Multiplex + 更快实时视频 |
| Grounded SAM 2 管线(github.com/idea-research/grounded-sam-2)| 2024–2025 | SAM 3 前的事实基线:Grounding DINO/1.5/DINO-X/Florence-2→SAM2→追踪。需最高检测 AP 时仍有用(Grounding DINO 1.5:54.3 COCO AP)|
| DINO-X(IDEA Research)| 2024–2025 | 统一开放词表检测/分割 |
| Florence-2(微软)| 2024 | 轻量统一 VLM(检测/字幕/OCR),SAM2 备选 grounding 前端 |
| SAM 2(Meta)| 2024 | 视频分割+追踪,SAM3 前主力 |
| CLIP/SigLIP | 2021+ | 几乎所有下面开放词表 3D 法的骨干;用来把文本/物体查询 embed 进场景图 |

**建议**:以 **SAM 3.1 为 2D 感知核心**;Grounding DINO 1.5 作 SAM 3 弱类的 fallback。

### 1.3 2D→3D lifting — 语义辐射/Gaussian 场(最新在前)
| 法 | 时间 | 关键点 | 实时? |
|---|---|---|---|
| ReSemGS-SLAM(KBS 2026)| 2026 | 实时语义 GS-SLAM | 是(声称)|
| Real-Time Language-Feature GS-SLAM(arXiv 2602.06991)| 2026 | RGBD SLAM 重建语言对齐稠密特征场,低延迟 track+map | 是(声称,未独立核)|
| Taking Language-Embedded 3DGS into the Wild(IEEE VR 2026)| 2026 | 无约束照片集开放词表场景理解 | 离线 |
| Relation-Centric Open-Vocab 3D Gaussian Seg(arXiv 2607.01140)| 2026-07 | 显式关系理解的开放词表 3DGS 分割 | 离线 |
| OpenInsGaussian | 2025–2026 | 实例级开放词表 Gaussian 分割+跨视图上下文融合 | 离线 |
| Unified 3D Open-Vocab Seg via GS(NeurIPS 2025)| 2025 | "Segment then Splat";静+动场景 | 近 |
| LangSplatV2(NeurIPS 2025)| 2025 | 修 LangSplat 实时弱(原 ~8.2FPS);支持高维 CLIP 特征 | 是(声称)|
| PanoGS(CVPR 2025)| 2025 | 3D 全景开放词表,grouping Gaussian 基元 | 近 |
| OpenGS-SLAM | 2025 | 开放集稠密语义 SLAM,3DGS,物体级 | 近 |
| OpenGaussian(NeurIPS 2024)| 2024 | 点级 3DGS 开放词表,基础 | 离线 |
| Gaussian-Grouping(ECCV 2024)| 2024 | 联合重建+3D SAM lifting | 离线 |
| SGS-SLAM(ECCV 2024)| 2024 | 首个基于 GS 的语义视觉 SLAM | 近 |
| Go-SLAM(arXiv 2409.16944)| 2024 | 动态 3DGS SLAM 物体级语义 | 近 |
| LangSplat(CVPR 2024 Highlight)| 2024 | CLIP 特征嵌 3DGS,V2 前基线(~8FPS)| 弱 |
| LERF/LERF++ | 2023–2024 | CLIP 进 NeRF(GS 前时代)| 慢 |
| OpenScene(CVPR 2023)| 2023 | 3D 点上稠密 CLIP,零样本查询(~778 引)| 离线 |
| ConceptFusion(RSS 2023)| 2023 | 开放集多模态 3D 地图(~458 引)| 离线 |
| OpenMask3D(NeurIPS 2023)| 2023 | 多视图 CLIP 融合的零样本 3D 实例分割(~387 引)| 离线 |

### 1.4 实时稠密/语义 SLAM(最新在前)
WildGS-SLAM(CVPR 2025,单目,野外鲁棒);Robust GS-SLAM One-Shot Init(arXiv 2601.00705,2026,TUM 2.5–3.2 FPS);SPLAT-SLAM(纯 RGB 全局优化);GlORIE-SLAM(单目);SplaTAM(RGB-D 稠密);MonoGS(CVPR 2024,首单目 GS-SLAM);**Photo-SLAM(ORB-SLAM3 追踪+Gaussian 映射,最快实时经典+GS 混合)**;RD-SLAM(3DGS+G-ICP)。⚠️ ReSemGS-SLAM / arXiv 2602.06991 的 2026 FPS 数字**未独立核验**,按作者声称处理。

## 2. 持久 & 空间记忆

### 2.1 3D 场景图(最新在前)
- **KeySG**(2025)—真实室内开放词表*功能*场景图(含 affordance)。
- **OpenLex3D**(NeurIPS 2025)—开放词表 3D 场景表示基准。
- Open3DIS(CVPR 2024);MaskClustering(2024,视图共识 mask 图聚类,3D 实例);Open3DSG(CVPR 2024,点云开放词表场景图)。
- **HOV-SG**(RSS 2024,hovsg.github.io,arXiv 2403.17846)—**分层开放词表 3D 场景图**(CLIP 特征进 3DSSG 结构,灵感 Hydra),多楼层尺度;**你房间记忆最接近的现成基底**(~327 引;github.com/hovsg/HOV-SG)。
- **ConceptGraphs**(2023,concept-graphs.github.io,arXiv 2309.16650)—CLIP+SAM lifting 开放词表场景图(~653 引,基础)。
- Hydra(MIT,2022,实时增量 3D 场景图);Kimera(MIT,2021);Embodied Semantic Scene Graph Generation(PMLR v164,2022)。
- **SceneGraphLoc**(ECCV 2024,scenegraphloc.github.io)—查询图在 3D 场景图库里的跨模态定位。**直接相关"我在记住的房间哪儿?"**
- **OK-Robot**(RSS 2024,ok-robot.github.io,arXiv 2401.12202)—开放知识模块化家用机器人(VLM+开放词表物体导航+操控,58.5% OVMM)。**最接近你想栈的已部署例(减去角色)**。
- MR-COGraphs(2024,多机器人)。

**对"建并使用房间持久 3D 记忆的 agent",2025 最重要的两作**:
- **3D-Mem / SnapMem**(CVPR 2025,arXiv 2411.17735,github.com/UMass-Embodied-AGI/3D-Mem)—用"Memory Snapshots"(多视图信息图)的 3D 场景记忆(~92 引)。**这是你角色心智模型最接近的学术类比**。
- **ESCA**(NeurIPS 2025)—场景图生成以情境化具身 agent,修空间关系/终止态错误。

### 2.2 LLM-agent 记忆框架
| 系统 | 架构 | 最适合 | 注 |
|---|---|---|---|
| **Zep**(arXiv 2501.13956)| 时序知识图谱(Graphiti 驱动)| **DMR 基准 SOTA,超 MemGPT**。时间感知事实(~307 引)| 你*情景+时序*用户记忆最佳选 |
| **Graphiti**(github.com/getzep/graphiti)| 开源时序上下文图引擎 | 实时事实追踪;双时序建模 | Zep 内引擎,可独立用 |
| **Letta(前 MemGPT)** | 全 agent 框架+自管记忆 | agent 自治 | 比单纯记忆层耦合重 |
| **Mem0** | 向量库+可选图(Mem0g) | 灵活;长/短/语义/情景 | 想要更薄记忆层时 |
| Cognee | 知识图谱记忆层 | 结构化个人数据 | |
| LangGraph memory/LangMem | 库级原语 | 自造 | |

共识:**Zep/Graphiti 时序最准,Letta 自治,Mem0 灵活**。

### 2.3 向量库 + embedding
- **embedding**:**BGE-M3**(arXiv 2402.03216)—多语(100+ 语言)、8192 token、多功能(dense+sparse+ColBERT)一次前向(~744 引)。
- **向量库**:**Milvus**(BGE-M3 原生集成最好,3 种向量类型 via `BGEM3EmbeddingFunction`);**Qdrant**(Rust,常用快 10–25%,低开销);**pgvector**(全进 Postgres,ACID/JOIN,与关系数据同库);Chroma 原型行但无原生 BGE-M3 支持。
- 工作站 24/7 处理百万级物体/embedding 行 + 场景图:**Milvus 或 Qdrant**。要图+向量+元数据一库:**Postgres+pgvector+Apache AGE**(图扩展),简单。

### 2.4 物体恒常性 & 增量地图更新
- ConceptGraphs/HOV-SG 原生支持增量:新 SAM+CLIP 检测成候选节点,视图共识匹配时融合。
- MaskClustering 给帧间物体级融合原语。
- Open3DIS/OpenInsGaussian 改实例跨视图稳定性,缓解"是不是同一杯子"漂移。
- 长时身份:每实例节点稳定 UUID;SAM3 视频追踪;每会话 SceneGraphLoc 式跨模态定位重关联。

## 3. Apple 原生感知(今天 Swift 能做什么)
- **企业主摄像头**(visionOS 2+):WWDC24 enterprise APIs;Apple "Accessing the main camera"(`ARKitSession`+`CameraFrameProvider`);github.com/Waley-Z/visionos-main-camera;vision.engineer 实战;**864×704 `stereoCorrected`** 立体格式(Griffin Hurt);无烘焙深度/分割(自己算)。
- **始终可用(无需 entitlement)**:`SceneReconstructionProvider`/`ARMeshAnchor`;`WorldTrackingProvider`/`HandTrackingProvider`/`EyeTrackingProvider`;`WorldAnchor`(仅 UUID+位姿,ARKit 跨会话按 ID 静默重定位,**无 `ARWorldMap`**)。OrangeLoops 模式:自持久 JSON `UUID→内容` 映射。
- **Vision 框架(实时企业帧上)**:`VNRecognizeObjectsRequest`、`VNCoreMLRequest`、`VNDetectHumanBodyPose3DRequest`、`VNGeneratePersonMaskRequest`/`VNGeneratePersonSegmentationRequest`;WWDC25 RecognizeDocumentsRequest。
- **逐帧检测 lift 到 3D**:(1) 射线打进 `SceneMesh`(便宜,RealityKit 原生,0.5–5m 范围);(2) 立体深度(自算 864×704 左右对视差,对 mesh 漏的薄/小物体有用)。最佳:先射线,miss 再立体深度。
- **Core ML/MLX on M1 Max vs 流到工作站**:M1 Max 跑 Qwen3-VL-4B 等小模型做低延迟/隐私查询(注视、手势意图);工作站跑重 VLM(Qwen3-VL/InternVL/UniVR-34B)+ embedding(BGE-M3)+ 密集感知(SAM3.1/Grounding DINO 1.5)。**带宽**:864×704 立体@30fps RGB8≈145MB/s 原始,~70MB/s JPEG;千兆/10GbE 够,10GbE 单程 10–30ms。

## 4. 角色"心智模型"——推荐设计
可查询、持久、增量维护的语义 3D 场景图记忆,分层:
1. **几何基底(端侧 visionOS)**:`SceneReconstructionProvider`+`WorldTrackingProvider`;持久 `WorldAnchor` UUID;自存 JSON `UUID→语义节点 id`。
2. **物体/实例层(工作站)**:3D 场景图(HOV-SG 式分层:楼→房→家具簇→物体→affordance)。每节点={UUID,3D 位姿(锚 WorldAnchor),CLIP/SigLIP 特征,类别,affordance 标签,首见/末见时间戳,帧快照(Memory Snapshots),置信度}。边=空间关系(在…上/后/内/可达),由 SpatialRGPT 或 SceneGraphLoc 式推理算。
3. **情景/时序层(工作站)**:**Graphiti/Zep** 时序 KG("用户说了 X,杯子时间 T 从桌到架")。双时序建模=能答"昨天杯子在哪?"。
4. **检索层(工作站)**:**Milvus/Qdrant**+BGE-M3 对(a)物体节点字幕(b)情景事件(c)对话史做开放词表 top-K;混合 dense+sparse+ColBERT。
5. **认知层(工作站 96GB)**:**UniVR-34B**(规划/视觉推理);**Qwen3-VL-8B/InternVL3.5**(快速 VLM 实时感知问答);**SpatialRGPT**(区域 grounding 空间问答);**SAM 3.1**(可提示分割)。
6. **实时更新环(每相机帧 ~30Hz)**:Mac 取立体对+手/眼/世界位姿+射线锚点进 mesh+mesh 空处立体深度;子采样关键帧(5–10Hz)经 LAN 流到工作站;工作站 SAM3.1 分割+追踪、CLIP 嵌 mask、Grounding DINO 1.5 补窄类、MaskClustering 式融合进场景图、Graphiti 嵌文本事件、Milvus 检索;推理器(UniVR-34B/Qwen3-VL)拿{场景图子图,top-K 记忆,当前帧}→吐角色动作/语音。
7. **跨会话重定位**:启动 ARKit 恢复 WorldAnchor UUID;服务器按 UUID 匹配场景图节点;SceneGraphLoc 式跨模态定位核验房间没动;不符(家具挪了)跑增量更新环到置信阈值。

最该研习(2025–2026):3D-Mem/SnapMem(CVPR 2025)、HOV-SG(RSS 2024)、OK-Robot(RSS 2024)、ConceptGraphs(2023)、ESCA(NeurIPS 2025)、UniVR-34B(2026-07)。

## 5. 可行性(工作站 96GB + Mac 32GB)
| 负载 | 放哪 | 置信 |
|---|---|---|
| UniVR-34B(FP16 ~68GB)| RTX Pro 6000 96GB 单卡,余量给激活 | 高 |
| InternVL3.5-241B-A28B(MoE,~28B 激活)| 单卡临界;INT8/AWQ 或 2 卡。24/7 用 **InternVL3-78B 或 Qwen3-VL-8B** | 中(241B)|
| **Qwen3-VL-8B**(2025-09,Apache2.0)| 工作站,轻松,余量给其他。**最佳主力 VLM** | 高 |
| SAM3/3.1 | 工作站,3.1 实时视频优化 | 高 |
| Grounding DINO 1.5/DINO-X | 工作站,小 | 高 |
| BGE-M3 embedding | 工作站,极快,千/秒批 | 高 |
| Milvus/Qdrant/pgvector | 工作站 CPU/RAM,百万向量轻松 | 高 |
| Graphiti/Zep 时序 KG | 工作站,需 Neo4j 等图后端 | 高 |
| RealityKit 渲染+ARKit 感知 | M1 Max Mac 原生 | 高 |
| 轻量 Core ML/MLX(Qwen3-VL-4B)| M1 Max,亚 100ms 本地 fallback | 高 |
| 864×704 立体深度 | M1 Max Metal compute,实时可行(SGM 类可;学习方法更重)| 中 |

**LAN 带宽/延迟**:千兆 ~110MB/s 峰值,JPEG 立体对 5–10Hz 关键帧够;**30Hz 原始或双向推激活/embedding 强烈建议 10GbE**;10GbE 往返 ~15–40ms,适合 200–500ms 思考反应的角色。

## 6. 推荐架构 + 最难点 + 首读(最新在前)
### 数据流
```
[Vision Pro / M1 Max]
  ARKitSession
    ├ CameraFrameProvider(企业)→ 864×704 stereoCorrected L/R @~30Hz
    ├ SceneReconstructionProvider → ARMeshAnchor 几何
    ├ WorldTrackingProvider → 设备位姿 + WorldAnchor UUID
    ├ HandTrackingProvider → 捏/指/抓意图
    └ EyeTrackingProvider → 注视目标(射线进 mesh)
  [端侧低延迟]
    ├ RealityKit 渲染角色
    ├ Core ML/MLX:小 VLM fallback(Qwen3-VL-4B)、体姿3D、人分割、射线进 mesh 快速 grounding
    └ 压缩+流 5–10Hz 关键帧+位姿(10GbE)
[RTX Pro 6000 96GB 工作站,常驻]
  感知:SAM3.1(文本提示开放词表检测+分割+追踪)、Grounding DINO1.5(窄类 fallback)、CLIP/SigLIP(嵌 mask)、MaskClustering 关联器
  记忆:场景图(HOV-SG 式,按 WorldAnchor UUID)+ Graphiti 时序 KG(Zep)+ Milvus/Qdrant+BGE-M3 检索
  认知:UniVR-34B(规划/视觉推理)、Qwen3-VL-8B(快速主力 VLM)、SpatialRGPT(区域 grounding 空间问答)
  → 吐动作/语音指令、子图更新、锚点映射
[Vision Pro 渲染角色响应,更新锚点]
```
### 3–4 个最难点
1. **跨会话物体恒常性**:用户明天回来,角色还认得"同一个杯子"?visionOS 给 WorldAnchor 几何重定位但**不**给语义身份。需自建关联管线(CLIP 特征匹配+SAM3 追踪史+SceneGraphLoc 式核验+affordance 标签),房间重排时优雅失败。**你用例最大未解研究问题**;3D-Mem(CVPR2025)、ESCA(NeurIPS2025)最接近。
2. **帧率实时语义融合**:每帧把 SAM3 mask+CLIP 进场景图不抖,需工作站批处理+Mac 关键帧子采样。LangSplat 8FPS;LangSplatV2(NeurIPS2025)+2026 实时 language-GS-SLAM(arXiv 2602.06991、ReSemGS-SLAM)声称实时但**未独立核验 FPS**。
3. **LAN 延迟预算**:Mac→工作站→Mac 往返须 <~500ms 才有"在场感"。逼出分层:端侧 Core ML 快意图(注视/手),工作站重感知,关键帧流而非全率。
4. **对场景图推理(非仅检索)**:有 CLIP 嵌 3D 节点必要但不充分;角色须答组合问题("杯子后面有没有到杯子的清路?")。UniVR-34B/SpatialRGPT/Spatial3D-LLM 入场,但都不原生吃持久场景图。需翻译层:把相关场景图子图序列化成文本/区域 prompt 供 VLM 摄取。

### 首读(最新在前)
UniVR-34B(arXiv 2607.12800;github.com/bytedance/UniVR;HF ByteDance/UniVR-34B-Planning)、Emu3.5(arXiv 2510.26583;github.com/baaivision/Emu3.5)、Real-Time Language-Feature GS-SLAM(arXiv 2602.06991)、ReSemGS-SLAM(KBS2026)、SAM3/3.1(ai.meta.com/blog/segment-anything-model-3;arXiv 2511.16719;github.com/facebookresearch/sam3)、3D-Mem/SnapMem(arXiv 2411.17735;github.com/UMass-Embodied-AGI/3D-Mem)、ESCA(NeurIPS2025)、LangSplatV2(NeurIPS2025)、Qwen3-VL(github.com/qwenlm/qwen3-vl)、InternVL3/3.5(github.com/opengvlab/internvl)、IJCAI2025 3D综述、PanoGS(CVPR2025)、SpatialBot(arXiv 2406.13642)、Zep(arXiv 2501.13956;blog.getzep.com;github.com/getzep/graphiti)、BGE-M3(arXiv 2402.03216)、HOV-SG(hovsg.github.io;github.com/hovsg/HOV-SG)、OK-Robot(ok-robot.github.io)、SpatialRGPT(anjiecheng.me/SpatialRGPT)、OpenGaussian(3d-aigc.github.io/OpenGaussian)、Gaussian-Grouping(github.com/lkeab/gaussian-grouping)、SceneGraphLoc(scenegraphloc.github.io)、ConceptGraphs(concept-graphs.github.io)、Apple visionOS 企业摄像头(developer.apple.com/documentation/visionos/accessing-the-main-camera;WWDC24;Waley-Z;vision.engineer;Griffin Hurt)、Apple WorldAnchor 持久(Tracking points in world space;Unity 讨论;OrangeLoops)、策展索引:github.com/DennisRotondi/awesome-3D-scene-graphs、github.com/ActiveVisionLab/Awesome-LLM-3D、github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM、github.com/vaew/Awesome-spatial-visual-reasoning-MLLMs。

## 局限注
严格顺序搜索预算下汇编。ReSemGS-SLAM(KBS)与 arXiv 2602.06991 的 2026 FPS 为作者声称,**未独立核验**。NICER-SLAM/EfficientGS/GauSSI 未能取 2026 更新,按 2024 基线处理。UniVR-34B-on-Emu3.5 经对 arXiv 2607.12800v1 直接引用核验(页本身被网络屏蔽未能直接 fetch)。
