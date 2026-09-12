# 开源 VLA 驱动虚拟角色 — 选型与适配(2026-08,联网核验)

> ⚠️ **修订(2026-08,用户决定)**:**首选路径从"离散 token 重定义(π0-FAST)"改为"连续动作 VLA"**——即 **π0 / π0.5 / GR00T N1.7(开源)在你 rig 的关节空间微调**(把虚拟 rig 的关节向量当动作空间,在 rig 关节轨迹数据上训)。原因:① 连续信息密度高、无量化损失,贴合"高精度/活感"目标;② **完美契前沿主线**(GR2/π0-flow/GR00T 皆连续,下一代直接继承);③ open-vla 原本"机器人形状"反对**只在复用预训练输出时成立**——train-on-your-rig 即消解;④ 数据侧更顺(木偶/仿真产连续关节轨迹,免 token 化)。**离散 token 降为 Phase1 轻量 fallback。** 采用 **大脑/小脑双系统**(大脑=具身推理 LLM 管语义泛化;小脑=连续 VLA 管运动泛化)。**仍需微调**(所有基础 VLA 上新本体都要),但模型/感知/跨本体机制均现成。详见 architecture-log 结论12 + 记忆。下文为原始调研(离散为主选的论述保留作参考)。
>

> 用户已决定**锚定 VLA**(非传统片段库+物理仿真桥)。本文给当下开源可用 VLA + 如何适配虚拟角色(机器人动作形状 vs 动画 rig 的适配是你要做的重活)。来源见文末;unconfirmed 已标。

## 一句话结论
**锚定离散 token VLA——首选 π0-FAST(次选 OpenVLA-OFT)——当 Brain(感知→动作 token),并把动作 token 词表重定义为"动画动作词表"(gesture/posture/gaze/emotion/locomotion),而非机器人电机指令。不要锚定连续动作 VLA**(π0 流匹配、GR00T、RDT-1B、CogACT)——它们 7-DoF/关节力矩输出是机器人形状,映射到骨骼/blendshape rig 最损最不可行。

三个支撑发现:
1. **SOLAMI(CVPR 2025)**= 首个"端到端社交 VLA,沉浸式交互 3D 自主角色(VR)",语音+肢体驱动。**几乎就是你的产品**,验证此方向是已发表前沿、非空想。**当你的模板。** ViBES(CVPR 2026)= 对话 agent + 行为智能 3D 虚拟身体,LLM 吐语义 token 驱动表达。
2. **离散动作 token 天生可重定义**。OpenVLA 把 256 个 Llama token 重用为动作 token;π0-FAST 的 FAST tokenizer 经 DCT+BPE 把**任意**连续 chunk 压成离散 token;SpatialVLA"自适应动作网格"为跨本体离散化。换动作空间有强先例(UniAct CVPR2025、LAP、X-VLA)。**把"机器人电机 token"换成"动画动作 token"架构上是被支持的。**
3. **VLA 不做人格/对话** → VLA 不能当全脑。VLA 进 Brain port 产**动作意图/动作 token**,旁挂一个对话/人格 LLM。MotionResolver 变成**薄的、确定性的 token→rig 解码器**。

**硬件无虑**:RTX Pro 6000 96GB 装**所有**开源 VLA(π0 全量微调 >70GB 也够)。OpenPI 自带**远程推理服务器(websocket)**——正好对应你 Mac(客户端)→ 工作站(服务器)的分工。

**诚实差距**:前沿适配;**没有开源 VLA 开箱输出 rig 控制**;离散 token 重定义最可行,但需自建(感知→动画动作)数据集——真正的活。

## 1. 开源 VLA 全景(最新在前;尺寸均适配 96GB)
| 模型 | 机构/来源 | 时间 | 规模 | 开源+许可 | 动作表示 | 真实落地? | 速度 | 成熟度 |
|---|---|---|---|---|---|---|---|---|
| Xiaomi-Robotics-1 | 小米(中) | 2026-07 | 大 | 论文;开源待定 | VLA 基座,scaling laws | 真机操控 | — | 研究 |
| InternVLA-A1 | 上海AI Lab(中) | 2026-01 | ~7B 级 | 是(HF InternRobotics) | MoT(理解+视觉前瞻+动作);称超 π0 | 真(12 真机任务) | — | 研究 |
| Xiaomi-Robotics-0 | 小米(中) | 2026-02 | **4.7B** | **是(开源)** | 实时执行,**80ms 推理** | 真 | 80ms(~12Hz) | 近可部署 |
| GR00T N1.7 | NVIDIA | 2025–26 | **3B** | **是(HF nvidia/GR00T-N1.7-3B)+商用** | **连续** 流匹配 DiT 动作头 | 真+仿真(Isaac) | **16ms 头/120Hz 环** | 产品轨 |
| DAXIAO ACE-Brain-0 | 达闼(中) | 2026-03 | — | **是(开源)** | 通用基座 | — | — | 早期 |
| cVLA | TUM 等 | 2025-07 | 小 | 是(权重+数据) | 相机空间 2D 轨迹 | 真 | 高效 | 研究 |
| π0.5 | Physical Intelligence | **2025-09** | ~3B 级 | **是(openpi)+ 开数据** | FAST tokenizer(离散)做数据;repo 有流匹配头 | 真(1万+小时) | 快 | 近可部署 |
| RoboBrain 2.0 | BAAI(中) | CVPR2025 | **3B** | 是(HF BAAI/RoboBrain2.0-3B) | MLLM"脑"(感知/推理/规划,非纯运动 VLA) | 真 | — | 研究 |
| InternVLA-M1 | 上海AI Lab(中) | 2025-10 | 7B 级 | 是(HF) | 空间 grounding+动作 | 真 | — | 研究 |
| LeVERB(基准) | — | 2025-06 | — | 开基准 | **人形全身控制**,sim-to-real,VLA 闭环,150+任务 | 仿真+真 | — | 研究基准 |
| SmolVLA | Hugging Face | 2025-06 | **450M** | 是(MIT 式,开源) | 离散 token,LeRobot | 真(社区) | 消费级 GPU | 可用/紧凑 |
| **SOLAMI** ⭐ | NTU 等 | **CVPR2025** | VLM 基 | 项目开;研究 | **社交 VLA→VR 里 3D 自主角色动作**;语音+肢体 | 合成(SynMSI) | 实时 VR demo | **概念证明(你的模板)** |
| **OpenVLA-OFT** ⭐ | Stanford/Berkeley/TRI | 2025-02 | 7B | 是(开源) | **离散动作 token chunk**(25 步),并行解码;256 重用 Llama token | 真(Open X) | **比 OpenVLA 快 26–43×** | 可用 |
| GR00T N1 | NVIDIA | 2025-03 | 3B | 是+商用 | 连续流匹配,双系统(S1 动作/S2 推理) | 真+仿真 | 120Hz | 产品轨 |
| SpatialVLA | 多机构(中关联) | 2025-01 | 7B 级 | 是(开源) | **自适应动作网格**——离散化连续动作统一本体;110万 episode | 真 | — | 研究 |
| **π0-FAST** ⭐ | Physical Intelligence | 2025-01 | ~3B | **是(openpi)** | **自回归,FAST 离散 token**(DCT+BPE 压连续 chunk) | 真(1万+小时) | 快 | 近可部署 |
| π0 | Physical Intelligence | 2024-10 | ~3B | **是(openpi)** | **连续** 流匹配动作专家 | 真(1万+小时) | 推理>8GB;全量FT>70GB | 近可部署 |
| CogACT | 微软 | 2024-11 | 7B+300M | 是 | **连续** 扩散动作 transformer | 真(Open X) | — | 研究 |
| RDT-1B | 清华(中) | 2024-10 | **1.2B** | 是(HF) | **连续** 扩散 transformer,预测 64 步 | 真(100万+episode) | — | 研究(ICLR2025) |
| OpenVLA | Stanford/Berkeley/TRI | 2024-06 | **7B** | **是(MIT)** | 离散动作 token(256 bin/DoF,重用 Llama);单步 | 真(97万 demo) | 慢(单步) | 成熟基线 |
| TinyVLA / MiniVLA(1B) | 多/SAIL | 2024/2025 | 紧凑/1B | 是 | 离散 token | 真 | 快 | 研究 |
| Octo | Berkeley 等 | 2024-05 | **27M/93M** | 是 | **连续** 扩散 readout | 真(80万 Open X) | 快 | 可用/紧凑 |

**注**:"Red Hawk" 非 VLA(是 Montclair State 人形机器人平台,命名混淆)。**Being-H0.5** NVIDIA 博客提及 2026 开源 VLA,细节未能一手核(unconfirmed)。**中国开源 VLA**(你中文环境)强项:InternVLA-A1/M1、RoboBrain 2.0、RDT-1B、Xiaomi-Robotics-0/1、SpatialVLA、DAXIAO ACE-Brain-0。

## 2. 动作表示分类——哪种最适配虚拟 rig
- **连续动作 VLA(流匹配/扩散)**:π0、GR00T、RDT-1B、CogACT、Octo。输出连续 7-DoF 末端/关节,条件于机器人**本体感知(关节角)**。**适配 rig:最差**——要造假本体感知 + 把关节/末端轨迹 retarget 到拓扑/DoF 不同的人形 rig,损且脆,模型没见过 rig 语义。**避免锚定。**
- **离散 token VLA(分箱/DCT)**:OpenVLA/OFT(256 Llama token 重用)、π0-FAST(FAST:DCT+BPE 压任意连续 chunk)、SpatialVLA(动作网格)。**适配 rig:最好**——token 是本体无关占位符,词表是设计选择非物理约束。可**把词表重定义为动画动作词典**(gesture/gaze/posture/emotion/微 locomotion/时序)。**关键杠杆:FAST 能 token 化你训练的任何连续信号**——若你的"动作"是 rig 参数(blendshape 权重、IK 目标曲线、片段混合权重),FAST 一样 token 化。这是 VLA 架构→rig 最干净的桥。
- **混合"通用动作"/跨本体(重定义动作的先例)**:UniAct(CVPR2025,学本体无关动作、快速适配新本体)、LAP、X-VLA(软提示跨本体)、"Actions as Language"。社区正把 VLA 动作头与特定本体解耦。**你的"新本体=虚拟 rig"是此趋势的合法一例。**
**裁决**:离散 token VLA(π0-FAST 或 OpenVLA-OFT)+ 重定义词表 最适配 rig。连续 VLA 最多用其感知/推理骨干,不用其动作头。

## 3. 适配层方案(按实用性排)
**Rank1(推荐):VLA 当脑 + 重定义动作 token 词表**。取 π0-FAST/OFT 架构,把机器人动作 token 头/词表换成你设计的动画动作词表,在(感知→动画动作)数据上微调。MotionResolver = 确定性 token→rig 解码(IK/blendshape/片段混合)。先例:SOLAMI、ViBES、Galatolo 2025、FAST/UniAct/LAP 证明 token 空间可换。**为何最佳**:保留 VLA 强感知+语言推理;免损 retarget;动画动作远低于关节力矩维且语义化。
**Rank2:VLA 当脑(仅感知+推理)+ 轻量控制器**。VLA/VLM 只产意图(JSON),手写/小 ML 控制器映射到 rig。最低险、最快出 demo,但丢了"统一感知→动作"训练(基本=VLM+规则)。好 fallback/Phase1。
**Rank3:retarget 连续 VLA 动作→rig 关节**。最损最脆(DoF 不匹配、造假本体感知、没训过 rig 语义)。仅当你要某预训练连续 VLA 的操控技能时考虑。
**Rank4(你要避开的):sim2rig**。VLA 在仿真人形里跑、流关节到 RealityKit。先例:LeVERB、AGILE、GR00T-Dreams。重(全仿真在环)、延迟,是"传统笨重"路径。注意:这是让 VLA 原生控制人形身体最成熟的方式——但原生控制的是**仿真机器人**身体,非动画 rig。

## 4. 已在仿真/虚拟身体上跑的 VLA(先例——你立论最强处)
- **SOLAMI(CVPR2025,arXiv 2412.00174)** ⭐——首个端到端**社交** VLA,VR 里沉浸交互 3D 自主角色,语音+肢体驱动,合成数据 SynMSI。**几乎就是你的产品**。学它、扩它。https://solami-ai.github.io/
- **ViBES(CVPR2026,arXiv 2512.14234)**——对话 agent + 行为智能 3D 虚拟身体,LLM 吐**语义 token 驱动表达**(共话语手势、talking-head、text-to-motion)。= "重定义 token 为行为词表"的已验证范式。
- **LeVERB(2025-06,arXiv 2506.13751)**——首个 sim-to-real 就绪、视觉-语言闭环的**人形全身控制**基准,150+ 任务。证 VLA 能控整个人形(仿真),不只手臂。
- **AGILE**(社区 Isaac-Lab):3 个 VLA 控仿真人形(移动+操控)。
- **Galatolo 等 2025**(PMC12122315)——token 级同时生文本+手势,开销极小。
- **NVIDIA ACE**(工业,非开源 VLA):数字人生成栈——产品化方向。**Figure Helix**(闭源):通用 VLA,概念可用于 avatar 但不开源。
**解读**:"VLA 驱动虚拟/社交身体"子领域存在且在顶会发表(CVPR2025/2026)。你锚定此处不疯——但你在**研究前沿**,非即插即用。

## 5. VLA 在你架构里坐哪(Brain vs MotionResolver)
你定义了 **Brain**(Context→Decision)和 **MotionResolver**(Decision→rig)两个 port。
**推荐:VLA = Brain 的感知核心;MotionResolver = token→rig 解码器**。具体:
- **Brain port 有两个协作模型**:(i) **对话/人格 LLM**(VLAs 不做人格/开放对话,必须分开);(ii) **VLA(π0-FAST/OFT,微调)**——吃同一感知流(房间/用户/对话上下文),输出**动画动作 token**=Decision 的运动意图部分。Decision = {LLM 的对话文本/语音} + {VLA 的动作 token chunk}。VLA 可条件于 LLM 的对话输出→手势与语音对齐。
- **MotionResolver port** = 薄、近乎确定性:动画动作 token → IK 目标/blendshape/片段混合 → RealityKit rig。**ML 不在这里**,它是 VLA 重定义路径需要的适配器。
**别把 VLA 放 MotionResolver 后台**:那等于 Brain 吐符号决策再让 VLA 翻成动作——但 VLA 端到端训于原始感知,浪费其强项(接地感知)。VLA 价值在感知→动作,属上游吃原始传感器。
**架构红利**:OpenPI 自带**远程策略服务器(websocket)**——工作站跑 VLA 服务器;Vision Pro app(Mac/M1)当瘦客户端流观测、收动作 token。**正是 PI 设计的部署模式。**

## 6. 具体起步选型 + 适配计划 + 时间线 + 差距
### 起步 VLA:**π0-FAST**(主)、**OpenVLA-OFT**(备)
- **π0-FAST**:FAST tokenization 是重定义词表最干净机制(可 token 化任意连续 chunk);OpenPI 成熟微调管线 + **远程推理服务器**匹配你硬件分工;全量微调 >70GB→96GB 够;2025-09 发布,权重+数据开源;强 VLM 感知骨干。
- **OpenVLA-OFT**(备):纯离散 token chunking 更简单(256 重用 Llama token + 25 步 chunk),比 OpenVLA 快 26–43×,MIT,大微调社区;骨干略旧。
### 适配层(真正的活)
1. **设计动画动作词表**(核心创造/工程决策):token 意指分类{gesture-id, gaze-target, posture, emotion, locomotion-step} 或 FAST-chunk 的连续 rig 参数(blendshape/IK 曲线/片段混合权重,短时序)。起步:几百 token 分类 + FAST-chunk 连续参数做细动作。
2. **建数据集**(最难):记录/自动标注 {VP 深度+RGB + 企业相机 + 房间 mesh + 用户态 + 对话转录} → {角色该吐的动画动作 token}。冷启动:(a) 程序化/authoring——标你已有 Blender 动画及触发上下文;(b) 木偶/遥操——手驱 rig 录;(c) 合成——经仿真人形(LeVERB/AGILE 式)或 LLM 辅助标注。目标起步几百小时(OpenPI 预训练 1万+;你是微调非预训练)。
3. **微调** π0-FAST(单 GPU LoRA >22.5GB;工作站全量 >70GB)在 LeRobot 格式的(感知→动画动作)数据上。OpenPI `examples/libero` 管线是模板——把动作映射(`Inputs/Outputs` 策略类)换成你的 token 词表。
4. **MotionResolver**:动画动作 token → RealityKit 的确定性解码器(IK 目标、blendshape、片段混合)。复用你已有 rig 工作;几乎无 ML。
5. **接远程推理服务器**:工作站跑 VLA 服务器;VP 流观测、5–15Hz 收动作 token chunk(动画动作远低于 120Hz 电机控制——此路径大优势)。
### 时间线(单人/小团队,前沿)
- 0–2 月:复刻 SOLAMI/ViBES 思路;工作站起 OpenPI 推理;定 v1 动画动作词表;远程服务器接一个最简 VP 场景。
- 2–4 月:建数据管线(authoring+遥操+合成);首次 LoRA 微调;端到端 感知→token→rig 跑简单行为(坐/看/手势)。
- 4–8 月:扩数据集;全量微调;加对话-LLM 条件做语音对齐手势;处理房间 grounding(坐*真实*躺椅=感知→把角色放对位的动作 token)。
- 8–12 月:打磨、鲁棒、人格集成、实时约束。研究级陪伴远早于消费级可出货。
### 诚实差距(对比物理机器人部署)
- π0/GR00T 在真 Franka/人形上是**受支持、近可部署**路径(1万+小时匹配真机数据)。你的虚拟角色部署**零匹配预训练数据**——每个开源 VLA 都训于机器人。数据集与动作空间差距你自己扛。
- **无开源 VLA 开箱输出 rig 控制**。最接近的(SOLAMI、ViBES)是合成/有限数据的概念证明,非产品栈。
- 连续 VLA(π0/GR00T/RDT)对 rig 基本不可用(除非重损 retarget);离散 token VLA 可适配但微调负担归你。
- "VLA 当脑 + 重定义 token 词表"是最可辩护的赌注——因其绕开机器人形状问题——但这是**押一个方向,非现成工具链**。2026+ 开源 VLA(尤其中国 InternVLA-A1、Xiaomi-Robotics-1、RoboBrain-2.5)将改善跨本体与推理,逐步缩小你的适配层。**现在押离散 token 重定义架构,正好让你吸收这些改进。**

## 与 contracts-v1 的关键汇合(重要)
**contracts-v1 的"动作 token 词表"就是 VLA 重定义后的动作空间**——你之前定的细粒语义动作 token(gesture/gaze/posture/emotion/...)正是 π0-FAST 要重定义成的"动画动作 token"。**两条决定(细粒动作词表 + 锚定 VLA)天然合流**:VLA 吐 file-18 式 token,MotionResolver 据此驱动 rig。这把 contracts-v1、architecture-log 结论4、本文件焊在一起。

## 来源(关键)
- OpenPI(π0/π0-FAST/π0.5)https://github.com/Physical-Intelligence/openpi ;π0 https://arxiv.org/html/2410.24164v1 ;π0.5 https://www.pi.website/download/pi05.pdf ;FAST https://arxiv.org/abs/2501.09747
- GR00T N1 https://arxiv.org/abs/2503.14734 ;N1.7 https://github.com/Nvidia/Isaac-GR00T + https://huggingface.co/nvidia/GR00T-N1.7-3B ;OpenVLA https://github.com/openvla/openvla ;OFT https://openvla-oft.github.io/ + https://github.com/moojink/openvla-oft ;RDT-1B https://github.com/thu-ml/RoboticsDiffusionTransformer ;CogACT https://cogact.github.io/ ;Octo https://octo-models.github.io/
- InternVLA-M1 https://arxiv.org/html/2510.13778v1 + A1 https://arxiv.org/html/2601.02456v1 ;RoboBrain2.0 https://github.com/FlagOpen/RoboBrain + https://huggingface.co/BAAI/RoboBrain2.0-3B ;SpatialVLA https://github.com/SpatialVLA/SpatialVLA ;SmolVLA https://huggingface.co/blog/smolvla ;TinyVLA https://arxiv.org/abs/2409.12514 ;MiniVLA https://ai.stanford.edu/blog/minivla/ ;Xiaomi-Robotics-0 https://github.com/XiaomiRobotics/Xiaomi-Robotics-0 + 1 https://arxiv.org/html/2607.15330v2 ;cVLA https://arxiv.org/html/2507.02190v2
- 跨本体/动作空间适配先例:UniAct(CVPR2025)https://openaccess.thecvf.com/content/CVPR2025/papers/Zheng_Universal_Actions_for_Enhanced_Embodied_Foundation_Models_CVPR_2025_paper.pdf ;LAP https://arxiv.org/html/2602.10556v2 ;X-VLA https://github.com/2toinf/X-VLA ;"Actions as Language" https://openreview.net/forum?id=sFO9d6XSlf ;Motion-Focused Latent Action https://arxiv.org/html/2606.18955
- 虚拟/仿真/社交身体先例:**SOLAMI(CVPR2025)** https://arxiv.org/abs/2412.00174 + https://solami-ai.github.io/ ;**ViBES(CVPR2026)** https://arxiv.org/html/2512.14234v2 ;LeVERB https://arxiv.org/html/2506.13751v1 ;Galatolo 2025 https://pmc.ncbi.nlm.nih.gov/articles/PMC12122315/ ;NVIDIA ACE https://www.nvidia.com/en-us/use-cases/digital-humans/ ;MDPI 2026 综述 https://www.mdpi.com/2813-2084/5/2/20
- OpenPI 硬件/许可:推理>8GB;LoRA FT>22.5GB;全量FT>70GB(A100/H100 级——RTX Pro 6000 96GB 合格);远程推理经 websocket 策略服务器(serve_policy.py)
- 2026 VLA landscape https://www.roboticscenter.ai/vla-models/best-2026/ ;NVIDIA world/action-models 博客 https://developer.nvidia.com/blog/pretrained-to-imagine-fine-tuned-to-act-the-rise-of-world-action-models/

**未确认**:Being-H0.5 参数/许可;Xiaomi-Robotics-1 开源权重状态;DAXIAO ACE-Brain-0 规格——依赖前对一手源核。

---

## 附 A:早期 VLA 调研要点(原 `_archive/12-vla-qwen-vla.md`「VLA / Qwen-VLA」并入;离散路径历史背景)

> 本节为立项早期 VLA 调研(原独立文件,见 `_archive/12-vla-qwen-vla.md`),写于"离散 token VLA 当脑"路径尚为首选时。现已被上方**连续动作 VLA + 大脑/小脑**决定取代(见顶部修订横幅 + architecture-log 结论12)。保留作历史背景与若干仍有用的细节。

### Qwen-VLA(2026-05,arXiv 2605.30280)— 早期一手事实
- ~5B:Qwen3.5-4B 视觉-语言主干 + **1.15B DiT 流匹配动作解码器**;开源(repo `QwenLM/Qwen-VLA`)。
- 动作=**连续轨迹**(流匹配);"embodiment-aware prompt conditioning"——一套权重、改 prompt 切换本体、无 per-platform 输出头(对"虚拟 rig 当新本体"有启发)。
- ALOHA 域内成功率 **83.6** vs π0.5 71.6 vs GR00T N1.6 28.6;OOD **76.9** vs 41.5 vs 25.4(证连续 VLA 强泛化——正是当前连续路径的支撑)。
- License/HF 权重路径未一手核(Qwen 家族默认 Apache-2.0;用前在 huggingface.co/Qwen 核)。
- 相关:Qwen-Robot Suite(2025-01)、RynnVLA-001(DAMO)、WorldVLA、TaoAvatar(2503.17032,见上表/利基文件)。

### 动作映射的四类桥接(早期框架,仍有参考价值)
- **(a) 动作 token→动画指令(非力矩)**:token-VLA 本体无关,词表是设计选择——重定义为 `<gesture>`/`<gaze>`/`<walk_to>`/`<emotion>`/`<speak>`。→ 即上方"离散 token fallback"路径的本质。
- **(b) LLM/VLM 当动作控制器**:MotionGPT/AvatarGPT/OmniControl 把动作当"外语"token 化、文生动作——开环片段生成,非实时反应式感知→动作。
- **(c) 游戏/仿真 agent 吐结构化动作**:DeepMind **SIMA** 动作空间=3D 虚拟世界键鼠(非机器人关节)——VLA 面向*虚拟*动作空间最清晰的现存证明;Voyager/GITM/DEPS 同理(Minecraft 代码/API)。
- **(d) 已部署虚拟角色管线**:TaoAvatar(Qwen2.5+ASR+TTS+A2BS+神经渲染,Vision-Pro 目标)、Look2React(IEEE TVCG 2026,VR NPC 视觉推理选姿态)、Meta Horizon LLM NPC。

### 早期算力结论(M1 Max 侧,补充上方 RTX 分析)
- Mac 经 **MLX** 可跑 2–4B Qwen2-VL/Qwen3-VL(M1 级 ~8–15 tok/s;MLX 在 Apple Silicon 上比 llama.cpp 快 ~1.8–3×,过 ~40k context 优势缩小);全端侧路径可行但紧,仍推荐 Mac 当渲染/感知前端 + 工作站当推理服务器。
- RTX Pro 6000 96GB 装所有开源 VLA(π0 全量微调 >70GB 也够);OpenPI 自带**远程策略服务器(websocket)**正好匹配 Mac 客户端→工作站服务器分工。
- TurboVLA(0.2B)报消费级 RTX 31ms/~32Hz、0.9GB VRAM——证小型专用动作策略 RTX 上实时可行。

### 早期"首读 5 项"(仍有效)
TaoAvatar、Qwen-VLA、SIMA/SIMA2、OpenVLA-OFT、TurboVLA + MotionGPT + "How Much Do LLMs Know about Human Motion?"(arXiv 2505.21531,界定 LLM/VLM 在*动作*层能/不能做什么)。

---

## 附 B:适配层 + 微调最小化 + VLA 演进赌注(2026-08,用户决定;当前立场)

### 核心张力(决定为何要多通道)
| | 世界感知 | 运动质量 |
|---|---|---|
| **极 C(运动生成 MotionLCM/MoMask)** | ❌ 弱(开环,文本/音频条件,不实时看世界) | ✅ 细腻自然(训在人形运动上) |
| **VLA(小脑 π0/GR00T)** | ✅ 强(视觉→动作闭环,实时反应) | ❌ 粗糙(机器人任务式;不碰细通道) |
- 极 C"眼盲但身段好";VLA"眼明但身段粗"。
- → **三者各补一块**:极 C(意图驱动细腻身体动作)+ VLA(感知驱动反应动作)+ 契约 A/MotionResolver(细通道:脸/注视/手/呼吸)。一个未来"成熟 VLA"(同时有感知+细腻运动)会把这三者统一。

### 适配层 = 对,采用(VLA 与 rig 解耦)
VLA 吐什么(EEF/关节/token)→ **适配层**归一化成标准动作空间(SMPL-X)→ 喂 MotionResolver。换 VLA 时**只重写适配器,不重训** → "换 VLA 只要重新适配"的兑现。
- 但适配层只解决**格式(身体映射)**,不解决**行为(动作分布)**:零微调 VLA 输出机器人任务式运动,非陪伴式。要陪伴行为仍需微调(下述压到极薄)。

### 微调最小化(三个杠杆,压到极薄)
1. **大量身体动作不用 VLA**——用运动生成(MotionLCM/MoMask/MotionGPT),文生运动、训在 AMASS/HumanML3D、人形原生、**零 VLA 微调**。共话语手势/姿态/走动/idle 交给它。
2. **VLA 只留给"感知→动作闭环"那一片**(反应式:你皱眉→调整、你走近→让位)。微调面从"全身"缩到"反应子集"。
3. **微调极薄 + 数据可移植**:只 LoRA action head(backbone 冻结);数据存标准 SMPL-X → 换未来任何 VLA = 同一份数据重 LoRA,**不重采**(贵的是数据一次性,换是重 LoRA)。

### VLA 演进赌注(用户立场,2026-08)
**赌**:等项目成熟时(数年),已有"成熟 VLA"(同时具备世界感知+细腻运动)可直接采用。
**为何稳健(hedge)**:Phase 1/A **完全不依赖 VLA**——零微调(极 C + 离散意图 + 库)就能跑通闭环、出可演示角色。架构本就为"换叶子"设计(适配层+标准骨骼+可移植数据)。成熟 VLA 来了→便宜插入(重适配 + 可选重 LoRA);没来→项目照样活,只是反应性弱些。
**趋势背书**:跨本体 VLA(RT-X/UniAct/CrossFormer)+ 通用 humanoid 策略正往"更少 per-embodiment 微调"走,与该赌注同向。今天极薄微调仍能提质量;2-3 年可能近零样本控标准人形。

### 落到阶段
- Phase 1 / A:**零微调**(离散意图 + 运动生成 + 库)。
- 小脑微调(M-6/7):**独立长 sub-track,默认缺省**;自采数据飞轮起来后再 LoRA,插进去替换 A/C。在那之前 A/C + 运动生成扛。
