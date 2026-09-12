# 动画线:贴真实家具的姿态合成 + neural motion + lip 同步(2026-08,联网核验版)

> 角色"坐躺椅/躺床/趴桌"愿景核心难点 + 实时 AI 驱动动画。硬件:M1 Max 32GB Mac(渲染+IK/接触)+ RTX Pro 6000 96GB 工作站(重 neural motion/pose,24/7)。本版已联网核验;不确定项标 **[UNCONFIRMED]**。

> 📌 **既定路线(对齐 architecture-log 结论 5 + 12,2026-08)**:动画不再是开放三选。已定为「**A 地板 + C 覆盖,分通道**」,且结论 12 引入**小脑(连续动作 VLA)**作为新增运动生产者。下表是既定分通道分配;本文其余为通道级 SOTA 调研(实现细节)。

| 通道 | 既定路线 | 说明 |
|---|---|---|
| 脸 / 情绪 / lip | **C** — Audio2Face-3D(开源,实时 ARKit52,已移 Apple Silicon) | 常驻 |
| 共话语手势 | **C** — 生成式 | |
| 过渡 / in-betweening | **C** — MotionLCM | |
| idle 微动 | 程序化 | 呼吸/换重心/fidget |
| 注视 | 程序化 IK | |
| locomotion / 走动 | **A** — 库 + navmesh(WWDC26 NavigationMeshResource) | 暂不上 C |
| **家具姿态(坐/躺/趴)** | **A + affordance 拟合** | 几何贴合优先;观众不在意重复 |
| **全身运动(新增)** | **小脑连续 VLA**(π0/GR00T rig 关节轨迹) | 见下「小脑」节;与 A/C 经运动解析器仲裁 |

**C 占比 = 运动解析器 port 上的可调旋钮**,随生成式成熟逐年 A→C(先 locomotion,再坐姿)。

## 一句话结论
1. **没有任何 2026 单一方法端到端解决你的验收测试**("坐进*我*的躺椅、贴着*这个*扫描 mesh")。它是个组合问题。最接近的单篇——**HOSIG(AAAI 2026)**、**CHOIS(CVPR 2024)**、**InterMimic(CVPR 2025)**、**PhySIC(SIGGRAPH Asia 2025)**——各解一片(场景条件生成 / 物理合理 / 单目重建)。**没有一个**吃一个 live ARKit `SceneReconstruction` mesh 输出实时、无穿模的 SMPL-X 姿态流。**你要自己组装,但架构今天可行。**
2. **你提的解耦推理架构是正确的标准 SOTA 模式**:重 neural 模型在 RTX 6000 24/7 跑;Mac 跑 RealityKit 渲染 + IK/接触拟合 + Audio2Face blendshape 流。认知→动作 tick(10–30Hz)与 90Hz 渲染解耦。延迟预算:非对抗陪伴角色 ~50–120ms 可接受。
3. **RealityKit 2025–2026 大幅增强**。WWDC26 加了 **NavMesh 寻路**(`NavigationMeshResource`/`NavigationComponent`/`NavigationController`)、交互式布料、混合现实光照、烘焙 lightmap、原生 Gaussian splatting、**场景理解 mesh 参与物理碰撞**(这是让"贴家具"原生可行的桥梁)。BlendTree 和全身 IK(`IKComponent`)已有。**无原生动画状态机**——你在 `AnimationPlaybackController` 上自建。
4. **面部管线最大使能器:NVIDIA 2025-09 开源 Audio2Face-3D v3.0**——实时输出 **ARKit 52 blendshapes**,已移植 Apple Silicon,集成 Maxine/ACE。这是 Vision Pro lip 同步的最短路径。
5. **身体最大使能器:MotionLCM(ECCV 2024)**让一步/少步文生动作真正实时;2025–2026 浪潮(MoMask++、MoLingo、MotionHiFlow、UniMo、Being-M0)给强生成先验。但它们**非场景条件**——需对家具*后拟合*。

## 1. 贴真实家具的 affordance 约束姿态(核心难点,最新在前)
文献分四类:**(a) 场景条件生成 (b) 接触/穿透优化拟合 (c) 物理控制器 (d) 纯 IK-to-surface**。无 2026 论文统一四者。

### 1a. 最新 SOTA — 场景条件 HSI/姿态生成
- **HOSIG: Full-Body Human-Object-Scene Interaction Generation with Hierarchical Scene Perception**(W. Yao 等,AAAI 2026;arXiv:2506.01579)—分层场景感知;93.3% 成功率;联合生 locomotion + 人-场交互 + 手-物接触。**当前最接近端到端的"走近并坐/爬"序列生成器**。**非实时**(扩散),训于合成场景非 live ARKit mesh。https://arxiv.org/abs/2506.01579
- **InterMimic: Whole-Body Control for Physics-Based Interaction with Dynamic Objects**(CVPR 2025)—物理模仿支持全身多物交互(含坐/躺)。最好配生成模型的运动学目标姿态。https://cvpr.thecvf.com/virtual/2025/poster/33421
- **PhySIC: Physically Plausible 3D Human-Scene Interaction and Contact from a Single Image**(SIGGRAPH Asia 2025;arXiv:2510.11649)—联合优化**人体姿态+场景几何+全局尺度+逐顶点接触图**,显式强制无穿透+地面支撑。代码 https://github.com/YuxuanSnow/Phy-SIC 。虽框架是单目重建,其优化目标正是你需要的后拟合目标。https://yuxuan-xue.com/physic/
- **TokenHSI**(CVPR 2025)—HSI 任务 token 化;用 PACER 式 locomotion 基线。
- **InterAct**(CVPR 2025)—大规模 HOI 含就坐动作。
- **InHabit**(arXiv 2026 **[ID 2604.19673 UNCONFIRMED]**)—基础模型做可扩展放置,显式对比接触+穿透优化范式。
- **Human-Object Interaction from Human-Level Instructions**(ICCV 2025)—指令驱动、物理合理、长程 HOI 合成。
- **Synthesizing Physically Plausible Human Motions in 3D Scenes**(arXiv:2308.09036)—物理控制器减 foot-slide 与身体穿透。
- **CoopDiff**(2025)—InterDiff 的接触引导扩散扩展。
- **ChairPose / 压力式椅子就坐姿态**(arXiv:2508.01850,2025)—椅子就坐专用,压力传感真值。
- **GenZI**(CVPR 2024)—首个零样本文→3D-HSI,适合 ideation,非特定真实 mesh 精度。

### 1b. 基础场景放置(旧但仍被引)
- **POSA: Populating 3D Scenes by Learning Human-Scene Interaction**(Hassan 等,CVPR 2021)—canonical "Per-Occupancy Spatial Affinity";训于 **PROX**(20 人×12 真实 3D 场景)。238+ 引。https://github.com/mohamedhassanmus/POSA
- **PROX 数据集**—https://prox.is.tue.mpg.de/
- **CHOIS: Controllable Human-Object Interaction Synthesis**(Jiaman Li 等,CVPR 2024;arXiv:2312.03913)—接触引导扩散,从文本+稀疏物体航点生同步人+物动作。https://lijiaman.github.io/projects/chois/
- **InterDiff**(Sirui Xu 等,ICCV 2023;arXiv:2308.16905)—物理信息扩散注入动力学,直接相关防穿模。246+ 引。https://sirui-xu.github.io/InterDiff/
- **TeSMo**(2024;arXiv:2404.10685)—文本+3D 场景生场景感知坐/走。
- **OMOMO / Hassan 等**—MPI-IS 接触物理线("Putting People into Scenes")。

### 1c. 就坐姿态专用(你的躺椅)
- **"Generating Sitting Poses"**(W. Zhang 等,arXiv:2308.12969,v2 2024-02)—引**骨骼姿态场**把参考坐姿适配到新椅子几何,形状感知,无需额外输入。**这极可能就是你说的 "SITS: Sit the Right Way"**——找不到字面叫 "SITS: Sit the Right Way" 的论文;**[UNCONFIRMED]** 你的缩写指这篇还是别的。https://pure.mpg.de/rest/items/item_3607651_6/component/file_3613286/content
- **Hierarchical IK for Seated Virtual Avatars**(2025)— seated avatar 实时 IK,上身稳定。直接可用于躺椅姿态稳定。
- **Automatic Sitting Pose Generation for Ergonomic Ratings**(Mao/Zhang,IEEE TVPG 2021)—非线性能量优化生用户定制坐姿。

### 1d. 把姿态 mesh 拟合到真实扫描表面(实操管线)
你的躺椅不是干净 CAD 椅,是带噪的 ARKit `SceneReconstruction` 三角 mesh。推荐管线(综合上文献):
1. 从 HOSIG/CHOIS/"Generating Sitting Poses" **条件于场景包围盒+affordance 标签(椅/床/桌)生成粗目标姿态**。
2. 跑 **PhySIC/PROX 式接触+穿透优化**:在指定接触区(臀/背/大腿/脚)最小化 SMPL-X 顶点到表面距离,惩罚穿透进场景 mesh。这是 ~50 个 SMPL-X 形状+全局平移参数的小 QP/非线性最小二乘;工作站 GPU 上用预算好的家具 SDF 几十 ms。
3. **RealityKit 物理沉降**—用新(WWDC26)场景理解 mesh 进物理的能力,让身体 `SimulationBody` 字面"坐"在真实 mesh 上,消除残余悬浮。
4. **IK 精修**用 RealityKit `IKComponent`(全身 IK)定腕/脚/头锚 + `look-at` 约束。
**关键坑**:ARKit 场景 mesh 是保守的(略膨胀),角色会"高坐"1–3cm,除非你缩小碰撞壳或把接触目标下偏。**预留每家具的美术调接触偏移**。

## 2. 神经动作合成 / 文生动作(最新在前)
### 2026 SOTA
- **MoLingo**(CVPR 2026,Tübingen/MPI-IS)—连续潜空间去噪+动作-语言对齐。
- **MotionHiFlow**(CVPR 2026)—分层流匹配替代扩散,强调物理合理。
- **MoTiGA**(CVPR 2026)—LLM 自回归 T2M + RLHF 式人对齐。
- **OpenT2M**(CVPR 2026)—大规模开源高质量数据。
- **MotionDuet / MotionMaster(HOI)**(CVPR 2026)—双条件与 HOI 泛化。
### 2025 SOTA
- **MoMask++ / SnapMoGen**(NeurIPS 2025)—HumanML3D+OmniMotion 当前顶尖。
- **Rethinking Diffusion for Text-Driven Human Motion**(CVPR 2025;arXiv:2411.16575)—掩码自回归扩散,KIT-ML+HumanML3D SOTA。
- **MotionBind**(NeurIPS 2025)、**GENMO**(arXiv:2505.01425)、**Motion-R1**(arXiv:2506.10353)、**Semi-Online Preference Optimization**(NeurIPS 2025)。
### 基础/仍相关
- **MotionLCM: Real-time Controllable Motion Generation via Latent Consistency Model**(Wenxun Dai 等,ECCV 2024;arXiv:2404.19759)—**实时声称已核验**,一/少步 LCM。216+ 引。**你管线的实时主力**。https://github.com/Dai-Wenxun/MotionLCM
- **MoMask**(CVPR 2024)—分层 VQ+掩码建模。
- **MotionGPT**(2023)、**MDM**(NeurIPS 2022)、**ReMoDiffuse/MotionDiffuse/FineMoGen/OmniControl/GloT/RealMDM**(2023–2024,HumanML3D)。
- **MMM(Generative Masked Motion Model)**—极快(0.081s/句)、极低 FID。
- **MotionCLM** **[UNCONFIRMED]**—找不到此确切名;唯一 LCM/CLM 式动作工作是 MotionLCM,疑为笔误。
### 数据集
**HumanML3D**(AMASS 派生,14.6K 带文片段)、**BABEL**(AMASS 动作标签)、**AMASS**(主 mocap 库,SMPL-X)、**SnapMoGen**(NeurIPS 2025)、**OpenT2M**(CVPR 2026)。
### 实时可行性
MotionLCM、MMM、MoMask 工作站 GPU 近实时(几十 ms)。扩散式场景条件模型(HOSIG、CHOIS)每片段秒级——**每意图跑一次,缓存**。

## 3. 物理角色控制器(最新)
- **SLMP(球面潜动作先验)**(arXiv:2603.01294,2026);**PRIOR**(arXiv:2603.18979,2026,深度地形+参数步态);**ExBody2**(arXiv:2412.13196,2024-12,全身跟踪任意参考动作并稳态,恢复坏姿态不悬浮 https://exbody2.github.io/ );**ALMI**(NeurIPS 2025)。
- 基础:**PHC: Perpetual Humanoid Control**(Luo 等,ICCV 2023,323+ 引,鲁棒通用模仿器,可从摔倒恢复,走向远处参考 https://github.com/ZhengyiLuo/PHC );**Universal Humanoid Motion Repr.(PHC++/BeyondMimic)**(ICLR 2024,179+ 引,全面运动技能表征);**OmniH2O**(He/Luo/Shi 等,IROS 2024,337+ 引,为遥操作设计但策略是通用全身跟踪器 https://human2humanoid.com );**ASE**(Peng 等,SIGGRAPH 2022,496+ 引 https://github.com/nv-tlabs/ASE );**AMP**(SIGGRAPH 2021,ASE 下的风格/先验层);**DeepMimic**(SIGGRAPH 2018);**PACER**(Rempe 等 **2023**,非 Pan;复杂地形轨迹跟随。Pan 2025 = TokenHSI,用 PACER 作基线);**ASIC**。
- **推荐**:**PHC/PHC++ 是最强现成全身控制器**;要实时用户身体遥操作用 **OmniH2O**。均非 visionOS 原生——PyTorch 跑工作站,流关节目标到 RealityKit。**坑**:需 Isaac Gym/MuJoCo,集成成本不低。

## 4. 音驱面部动画 / lip 同步(最新)
### 头条(Vision Pro 最短路径)
- **NVIDIA Audio2Face-3D v3.0 — 2025-09 开源**。输出 **ARKit 52 blendshapes**(visionOS/Personas/Memoji 标准),支持预录+**实时流**音频,含超越 lip 同步的情绪表达,**已移植 Apple Silicon**。GitHub https://github.com/NVIDIA/Audio2Face-3D ;HF https://huggingface.co/nvidia/Audio2Face-3D-v3.0 ;NVIDIA blog https://developer.nvidia.com/blog/nvidia-open-sources-audio2face-animation-model/ ;NGC NIM https://catalog.ngc.nvidia.com/orgs/nim/teams/nvidia/containers/audio2face-3d
  - **为何重要**:直接吐 RealityKit morph target 消费的 52 blendshape 权重。RTX 6000 推理,30–60Hz 流 52-float 向量到 Mac 驱动 rig。**最干净的 3D-rig 路径**(非 2D talking-head 视频)。
  - 旧专有模板:legacy Omniverse Audio2Face 用 46-blendshape;新 Audio2Face-3D NIM 用 ARKit 52(已核)。
### 最新学术
- **Speech-Driven Blendshapes for 3D Face Animation**(arXiv:2510.25234,2025-10)—直接音→blendshape 回归。
- **HighSync**(2026)、**SAiD**(扩散式 3D 面部)、**FaceTalk**(神经参数化头)、**NeuroSync**(实时 60fps transformer seq2seq 音→脸)。
### Apple 平台集成
- **ARKit 52 blendshapes**—RealityKit morph target/`BlendShapeAnimation` 原生支持。这是要 target 的 rig 标准。
- **OVR LipSync(Meta)**—旧 viseme 经典,CPU 轻,fallback。
- **NVIDIA Maxine/ACE**—捆绑 Audio2Face-3D 的实时 avatar SDK。
- **推荐**:Audio2Face-3D v3.0 在工作站,流 52-float ARKit blendshape 向量到 Vision Pro rig。这是已核实时、已核 Apple Silicon、开源的 lip 同步答案。

## 5. 共话语全身手势(最新)
- **Streaming Generation of Co-Speech Gestures via Accelerated Rolling Diffusion**(AAAI 2026)—流式滚动扩散,结构化渐进噪声,为低延迟对话设计。https://ojs.aaai.org/index.php/AAAI/article/view/39807
- **Motion-example-controlled Co-speech Gesture**(ACM 2025)、**Democratizing High-Fidelity Co-Speech Gesture Video**(arXiv:2507.06812)、**Joint Co-Speech Gesture and Expressive Talking Face**(WACV 2025)、**ConvoFusion**(多模态对话扩散)、**Emotional Speech-driven 3D Body Animation**(CVPR 2024)、**Co-Speech Gesture Video via Motion-Decoupled Diffusion**(CVPR 2024)。
- 基础:**BEAT**(Liu 等,ECCV 2022,325+ 引,76h 3D mocap+52D blendshape+音+文+情绪 https://pantomatrix.github.io/BEAT/ );**TalkSHOW**(CVPR 2023,组合面/手/身,VQ-VAE https://talkshow.is.tue.mpg.de );DiffGesture/GestureDiffuse;**BEAT2 [UNCONFIRMED 为具体 2025 论文]**。
- **推荐**:身体手势用 2025–2026 扩散模型(Streaming Rolling Diffusion 低延迟),工作站按句级节奏跑;lip 同步经 Audio2Face-3D 连续跑。**注意把手势重定向到当前就坐的角色**——多数手势模型训于站立说话者,下身须 IK 钉在躺椅。

## 6. 动作 in-betweening / 混合
- **Motion In-Betweening for Densely Interacting Characters**(SIGGRAPH Asia 2025)、**Deep Compositional Phase Diffusion**(NeurIPS 2025)、**Frequency-Domain Diffusion In-Betweening**(CMC 2025)、**SILK**(arXiv:2506.09075)、**Two-Stage Transformers**(ACM 2022,91+ 引)、**Spatial+Temporal Constraints**(IEEE 2024)。
- **你的管线**:in-betweening 缝合 idle→walk-to-recliner→sit。最干净解 = **MotionLCM 的 last-N-frames 时序控制**(已内置)—用片段 A 末几帧作片段 B 条件即得无缝过渡,无需独立 in-betweening 网络。

## 7. RealityKit on visionOS — 原生 vs 自建(精确能力)
### 实际存在(已核 Apple 文档+WWDC)
- **绑骨 USDZ + AnimationLibrary/AnimationPlaybackController**:支持。`AnimationLibraryComponent` 存命名动画;`AnimationPlaybackController` 播放。
- **Blend shape/morph target**:支持。`BlendTreeAnimation`/`BlendTreeNode` 一等类型;ARKit 52 blendshapes 直接映射。
- **Blend tree(确实存在)**:`BlendTreeAnimation` 从多 `BlendTreeNode` 构树,框架混合成一动画。**RealityKit 有 blend tree**(澄清常见混淆)。
- **IK/look-at**:`IKComponent` 给带骨骼 entity 提供**全身 IK 解算器**(iOS/macOS/visionOS)。look-at 在 Reality Composer Pro 内作约束实现。
- **WWDC24 Session 10102** "Compose interactive 3D content in Reality Composer Pro"=visionOS 角色动画权威参考(IK+blend shape+交互角色)。**WWDC25 Session 274**(SwiftUI+RealityKit)、**287**(What's new in RealityKit)。
### WWDC26(全新,已确认)
- **WWDC26 Session 279** "Explore advances in RealityKit" 加:**交互布料模拟**、**NavMesh 寻路**、混合现实光照、可定制混响、原生 Gaussian splatting、烘焙 lightmap、投射纹理。
- **NavMesh API**:`NavigationMeshResource`/`NavigationComponent`/`NavigationController`,带遍历成本与区域属性。
- **布料**:"fabric、软表面、可弯折折叠、响应力与接触的材料"。
- **场景理解 mesh 参与物理碰撞**—对你关键:`SceneReconstruction` 真实房间 mesh 喂 RealityKit 物理引擎,`SimulationBody` 角色字面"坐"在真实躺椅上。
### 不存在的原生
- **动画状态机/Mecanim 式 Animator**:**无内置**。须在 `AnimationPlaybackController`+blend tree 上自建状态图。
- **USDZ 反向动画**:visionOS 1.1 有限,`AnimationPlaybackController` 不干净支持反向播放。
- **NavMesh/寻路**:仅 WWDC26(2026)加;之前无。布料同。
### 结论
RealityKit 现有足够原语:绑骨 USDZ、blend shape、blend tree、全身 IK、场景 mesh 物理碰撞、(新)navmesh 寻路。**两件须自建**:(1) 动画状态机/行为层 (2) 逐帧 neural-pose→关节目标 桥。其余原生或 Swift 可写。

## 8. 意图/动作 → 动作映射层
文献收敛于**LLM 内离散动作 token**:
- **UniMo**(AAAI 2025;arXiv:2601.12126)—LLM 词表加 `<Motion_0>` 等 token+CoT。
- **LaMoGen**(2026;arXiv:2603.11605)—语言↔动作符号接口。
- **Motion-Agent/MotionLLM**(HKUST)—对话框架桥文本↔动作。
- **Being-M0**(ICML 2025)—百万级动作数据大动作模型。
- **Action-Token Decomposition RL**(UCL 2024)—token 级 RL 监督。
### 实操推荐(混合,非纯 LLM-as-controller)
1. LLM 吐"动作 token+参数"(`SIT{target=recliner}`、`GESTURE{intent=explaining}`、`LOCOMOTE{target=desk}`)。
2. Swift 侧小**行为树/有限状态机**把 token 翻成 (a) `AnimationLibrary` 基础片段 (b) 目标世界姿态。
3. **MotionLCM/生成模型** 工作站填中间帧。
4. **affordance 拟合(§1)** 把终止姿态 snap 到真实家具。
5. **RealityKit IK+物理** 接地。
避免让 LLM 每帧吐每个关节(延迟+非确定)。

## 9. 可行性(工作站 96GB + Mac 32GB)
### 工作站(RTX Pro 6000 Blackwell 96GB,24/7)
VRAM 对任一单模型充裕(批/流式推理)。预算(粗体=已核):
- **MotionLCM(文→动作):实时,一/少步 LCM——已核验**。每意图或每秒跑。
- **MoMask/MoMask++(文→动作)**:4090 上每片段几十 ms,Pro 6000 上微不足道。
- **Audio2Face-3D v3.0**:实时流——已核 NVIDIA 声称;移植 Apple Silicon,工作站 GPU 轻松。
- **HOSIG/CHOIS/PhySIC**:**非实时**(扩散,每片段秒级)。**每意图一次,缓存**。
- **PHC/PHC++(RL 控制器)**:MuJoCo/Isaac 600–1200Hz 仿真;单策略轻。成本在集成非推理。
- **手势(Streaming Rolling Diffusion)**:流式设计,每块低几十 ms。
### Mac(M1 Max 32GB)— RealityKit 渲染+IK+轻推理
- **90Hz 渲染**:RealityKit 原生;M1 Max 对单绑骨 USDZ 角色+blend shape+IK+场景碰撞绰绰有余。
- **Audio2Face-3D**:已核移植 Apple Silicon—工作站宕机时可**本地**跑(CoreML/ONNX-Runtime-CoreML EP)。
- **轻姿态/接触优化**:SDF 接触+穿透 QP(小 SMPL-X 参数集)M1 Max GPU via Metal/MPS 舒适。但完整扩散模型(HOSIG、CHOIS)本地交互速率**不现实**。
### 解耦 tick 模型(推荐)
- **认知→动作 tick:10–30Hz**(工作站:意图+lip 向量+手势块),经低延迟 socket(Thunderbolt 桥或 10GbE)流到 Mac。
- **90Hz 渲染环(Mac)**:插值流来的关节目标+跑 `IKComponent`+物理接触+blendshape 应用。
- **可接受端到端延迟(非对抗陪伴)**:50–120ms(真人对话伙伴类似)。

## 10. 综合 — 推荐架构 + 最难点 + 首读
### 推荐 action→animation 架构(原生 Swift+RealityKit,神经推理在工作站)
```
LLM 认知 ─吐─> 动作 token + 参数 + 语音文本 + 音频
        │
   ┌────┴──────────────────────────────────┐
   ▼                                        ▼
[RTX Pro 6000 工作站,24/7]            [M1 Max Mac—RealityKit 宿主]
 • Audio2Face-3D v3.0 →52-blendshape流@30–60Hz   • 90Hz 渲染环(USDZ rig)
 • MotionLCM/MoMask++ →每意图动作片段            • 行为树+自建动画状态机
 • HOSIG/CHOIS →粗坐/躺姿态目标(每意图)         • IKComponent(全身IK)+look-at
 • 手势(Rolling Diffusion)→上身手势层          • BlendTreeAnimation 混 idle↔action
 • PhySIC 式接触+穿透 QP →拟合SMPL-X到真实家具SDF  • 物理:SimulationBody vs 真实
   │                                              SceneReconstruction mesh(WWDC26)
   └────流 关节目标+52 blendshape float+根变换────┘
                    (10–30Hz,Thunderbolt/10GbE)
                          ▼
        90Hz RealityKit 插值+IK+物理沉降
                          ▼
        Vision Pro:角色坐/躺/趴在真实 mesh 上
```

### 小脑:连续动作 VLA 作为运动生产者(结论 12)
除上面 A/C 通道外,结论 12 引入**小脑 = 连续动作 VLA**(π0/GR00T,在你 rig 关节空间微调):吃实时感知 + 大脑意图 → 输出**连续 rig 关节轨迹** → 经运动解析器写入 rig。它不是替代 A/C,而是新增一条"感知→关节"的通路(更连续、更抗扰动、信息密度高)。微调前(Phase-1)可缺省,走 A/C 兜底。动作空间/接口契约见 `contracts-v1.md` 契约 C。

### 运动解析器(motion resolver port)
统一 rig 写入口与仲裁层。输入多源:小脑 VLA 轨迹 / 库片段 / IK 目标 / 生成式动作;输出:最终 rig 状态(@90Hz)。仲裁规则:按通道 + 优先级 + 冲突(如手在抓握时压制 idle fidget);**A↔C 占比是可调旋钮**。这是一个抽象 port——实现可从"纯 A"起步,逐年把通道迁到 C/小脑。

### 4 个最难点(排)
1. **affordance 拟合、mesh 贴合的就坐/躺姿态(针对你*特定*家具)**。无 2026 端到端。须组合 HOSIG/CHOIS(粗姿态)→PhySIC 式接触 QP(对 ARKit mesh 的 SDF)→RealityKit 物理沉降。最大坑是保守(略膨胀)ARKit 碰撞壳——预留每家具接触偏移。
2. **Swift 里动画状态机+神经片段混合**。RealityKit 有 blend tree+IK 无原生状态机。你写行为树+状态图层选片段、请神经填充、逐帧解过渡。
3. **延迟预算解耦**。认知/动作合成别上 90Hz 线程。Mac 须持关节目标+blendshape 向量小环形缓冲并 90Hz 插值。工作站推理突发(10–30Hz),Mac 平滑。
4. **手势重定向到就坐角色**。共话语手势训于站立说话者。下身须 IK 钉躺椅,手势须过滤避免穿椅臂/桌。
### 验收测试 — 坐真实躺椅/躺真实床/趴真实桌
- **无 2026 方法端到端解决**。最接近:HOSIG(AAAI 2026)粗生成 → PhySIC(SIGGRAPH Asia 2025)接触+穿透目标 → ARKit `SceneReconstruction` mesh(现参与物理,WWDC26)→ RealityKit `IKComponent` 精细接地。
- **今天可建**。估工:躺椅最难;通了,床(躺)/桌(趴)复用同管线换接触区集。
### 首读(最新在前)
1. HOSIG(AAAI 2026,arXiv:2506.01579)https://arxiv.org/abs/2506.01579
2. MoLingo(CVPR 2026);MotionHiFlow(CVPR 2026);Streaming Co-Speech Gestures Rolling Diffusion(AAAI 2026)
3. MoMask++/SnapMoGen(NeurIPS 2025);InterMimic(CVPR 2025);TokenHSI(CVPR 2025)
4. PhySIC(SIGGRAPH Asia 2025)https://yuxuan-xue.com/physic/ https://github.com/YuxuanSnow/Phy-SIC
5. **Audio2Face-3D v3.0**(NVIDIA,2025-09 开源)https://github.com/NVIDIA/Audio2Face-3D
6. ExBody2(arXiv:2412.13196)https://exbody2.github.io/
7. MotionLCM(ECCV 2024,arXiv:2404.19759)https://github.com/Dai-Wenxun/MotionLCM
8. OmniH2O(IROS 2024,arXiv:2406.08858)https://github.com/lecar-lab/human2humanoid
9. CHOIS(CVPR 2024,arXiv:2312.03913)https://lijiaman.github.io/projects/chois/
10. PHC/Universal Humanoid Motion Repr.(ICCV2023/ICLR2024)https://github.com/ZhengyiLuo/PHC
11. ASE(SIGGRAPH 2022)https://github.com/nv-tlabs/ASE
12. POSA(CVPR 2021)+PROX https://github.com/mohamedhassanmus/POSA
13. "Generating Sitting Poses"(arXiv:2308.12969,2024)—你"SITS"的真实身份 https://pure.mpg.de/rest/items/item_3607651_6/component/file_3613286/content
### 策展索引
- github.com/Zilize/awesome-text-to-motion(T2M)
- github.com/Kedreamix/Awesome-Talking-Head-Synthesis(talking-head/lip 同步)
- github.com/DirtyHarryLYL/HOI-Learning-List(HOI)

## 标注的不确定(构建前再核)
- **"SITS: Sit the Right Way"**—无此确切标题论文;最接近 **"Generating Sitting Poses"**(arXiv:2308.12969)。"SITS" 或为项目名/非正式。
- **"MotionCLM"**—未找到;疑为 **MotionLCM** 笔误。
- **InHabit arXiv 2604.19673**—搜索元数据出现;未能直 fetch arxiv 核确切 ID/作者。
- **BEAT2** 为独立数据集论文—未能定精确引用;"BEAT"(ECCV 2022)是已核基础。
- **PACER 作者**—原始是 **Rempe 等 2023**,非 "Pan 2024";Pan 2025 TokenHSI 用 PACER 作基线。
