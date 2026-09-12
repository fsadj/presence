# 世界模型能否简化你的架构?(2026-08,联网核验)

> 直接回答"世界模型能否折叠/简化当前多组件架构"。结论可能反直觉。来源文件号见文末;unconfirmed 已标。

## 一句话结论
**不能——至少今天不能,且不是通常设想的那种方式。** 2025–2026 文献最关键的发现:**前沿正在做相反的事——把显式记忆模块"外挂"到世界模型/VLA/VLM 之上**(EPM、MEM、3DLLM-Mem、SPMEM、HIMM,皆 2025–2026)。**你的 HOV-SG + Graphiti + Milvus 栈,恰好对齐了领域走向,而非落后。**

世界模型**真正能简化**的两处:
- **预判/前瞻变内置**(V-JEPA 2、DINO-WM、NavWorld、Cosmos 都能在"给定动作"下预测下一观测)——干净净赚,替掉"外挂预测层"。
- **感知→短程动作**可经统一 VLA/视觉推理模型(UniVR-34B/Emu3.5、π0.5、Gemini Robotics)部分合并——"VLM 眼 + LLM 脑"在**反应式短程通道**上可合。

世界模型**反而增加复杂度/不成熟**之处:跨会话持久记忆、可编辑性、可调试性、**多时间尺度推理(当前所有 WM 都是单时间尺度/短程/episode 内——直接和你"多时间尺度环"原则冲突)**、真实房间 grounding(多为仿真/桌面)、以及整个对话/人格/动作/渲染栈(碰不到)。

**净:WM 简化你 5 个 port 里的 ~1 个(预判),其余反而更复杂。保留架构;把 WM 窄采用为"前瞻服务",挂在已有契约后;"WM 当全脑"列为 2–3 年观察项。**

## 当前架构——哪些"可能塌缩"
| 当前组件 | 能否塌缩? | 原因 |
|---|---|---|
| 感知:立体相机+mesh+SAM3.1+VLM→3D→场景图 | 部分 | 潜/视频 WM(V-JEPA 2、DINO-WM)吃原始视频预测状态,不需手搭 lift 管线——但你会丢掉可编辑/可查询/锚点索引的表征 |
| 记忆:HOV-SG 场景图+Graphiti KG+Milvus/BGE-M3 | **不能(领域证实)** | 2025–26 是给 WM *加* 记忆,不是把记忆折进去。你会丢持久/可编辑/WorldAnchor 索引的信念 |
| 认知:LLM 脑+VLM"看"工具+屏幕 VLM | 部分(仅短程) | UniVR-34B/Emu3.5 式"视觉空间推理"可把眼+脑合并*为一个通道*;人格/对话/推理仍 LLM |
| 动作:motion resolver+Audio2Face→RealityKit | 不能 | WM 不生绑骨角色动作,它预测观测/动作 |
| 反射+行为基线 | 不能 | 本地快控正交 |
| 对话:ASR+LLM+TTS+barge-in | 不能 | 语音 I/O 与 WM 无关 |
| 5 个可插拔契约 | **不变——这是你的优势** | 正好让你能增量接入 WM |

## 世界模型全景(最新在前)
每项:**预测什么 / 开源+规模 / 真实落地? / 持久记忆? / 成熟度 / 时间**。

### A. 综述与"统一大脑"之争
- **NeurIPS 2025 Workshop "Embodied World Models for Decision Making"**(2025-12)。含观点文 *"A Unified World Model is the cornerstone for integrating perception, reasoning, and decision-making"*(Yipeng Xu)——**注意是"观点",即统一论是有待证明的愿景,非已验证**。Chelsea Finn(PI)讲"Long-Term Autonomy";Philip Ball 讲 **Genie 3**。https://embodied-world-models.github.io/
- **"A Comprehensive Survey on World Models for Embodied AI"**(Li 等,arXiv:2510.16732;v1 2025-10 → **v3 2026-06-25**)。三轴分类;点名的开放挑战**正是你的风险**:**长程时间一致性+误差累积**、实时算力权衡、缺物理一致性度量。https://arxiv.org/abs/2510.16732 ;https://github.com/Li-Zn-H/AwesomeWorldModels
- **"A Survey of Embodied World Models"**(Shang 等,2026,清华 FIB Lab)。https://www.preprints.org/manuscript/202604.0928
- **"Embodied AI: From LLMs to World Models"**(清华,2025;arXiv:2509.20021)。**"Understanding World Models"**(清华 FIB Lab,ACM CSUR 2025)。**WHALE**(周志华组,NeurIPS 2025 wm workshop,可扩展 WM)。

### B. 经典"预测式规划"
- **V-JEPA 2**(Meta/LeCun 系;blog+arXiv:2506.09985,2025-06)。**预测潜嵌入(非像素)**;1.2B、**开源**;>100 万小时视频+62h 机器人数据→真机零样本,未见物 pick-and-place 65–80%。**对你的硬限制**:**单时间尺度**(Meta 明说分层/多时间尺度 JEPA 是未来工作——**直接和你的多时间尺度原则冲突**)、短程 MPC 每步重规划、**无跨会话持久记忆**(每片段预测器)、仅桌面操控。https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/ ;https://github.com/facebookresearch/jepa-wms
- **DreamerV3**(Hafner,**Nature 2025**,DOI 10.1038/s41586-025-08744-2;原 arXiv:2301.04104,1700+ 引)。**潜空间 RSSM 预测**;开源;单一配置通 150+ 任务,比 model-free 省 10–100× 样本。**DreamerV4 已宣布**(可扩展 WM,细节 unconfirmed)。**硬限制**:**仿真训练**、RSSM **episode 间重置(无持久记忆)**、是控制非真实房间感知。https://arxiv.org/abs/2301.04104
- **DINO-WM**(ICML 2025)。在**冻结 DINOv2 潜空间**预测(不重建像素);新环境零样本规划;小/高效;仿真+导航,短程。https://dino-wm.github.io/
- **Navigation World Models (NWM)**(Bar 等,**CVPR 2025**,304 引)。可控**视频生成**(条件于过往观测+导航动作)= 像素空间"视觉前瞻";偏闭源;仅导航,分钟级。https://www.amirbar.net/nwm/

### C. 世界基座/视频世界预测器
- **NVIDIA Cosmos 3**(~2026 技术报告)。**联合预测 language+image+video+audio+action**(policy/forward/inverse dynamics)="世界-动作模型"。变体:**Cosmos3-Edge 2B、Nano 16B、Super(在 Qwen3-VL 32B 上,~64B)**;**Cosmos-Predict2.5(2B/14B,开源)**。HF 开源。**经机器人数据真实落地,但本质是合成数据/仿真/策略生成器;无持久 agent 记忆**——报告自己承认"物理理解需时间持久状态、绑对象/agent 的空间 grounding、affordance 推理",即**承认你架构已填的缺口**。96GB 可载(尤其 2B/14B/16B)。https://research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf
- **Genie 2(2024-12)→ Genie 3(2025)**(DeepMind)。**预测像素**(从图/提示生可控可玩 3D 环境);**闭源**;是**仿真/训练环境生成器**,非感知 agent;短游玩、无持久记忆、合成世界。
- **Sora 2**(OpenAI,2025)。更物理准确/可控视频;闭源;"Sora 当世界模拟器"论**有争议**(Duality.ai:视觉逼真但不功能忠实)。LucidSim 是机器人相关衍生。

### D. 统一具身基础大脑
- **UniVR-34B**(字节/北交大,arXiv~2607.12800,**2026-07**)在 **Emu3.5 34B**(BAAI,arXiv:2510.26583,2025-10)。**跨视觉+语言 next-token 预测**(Emu3.5="原生多模态即世界学习者")。UniVR 加 **VR-GRPO**(强化"在视觉空间推理",生视觉推理轨迹替代文本 CoT)做长程规划+细粒度动力学。**34B、开源**(ByteDance/UniVR-34B-Planning),96GB 可载。**最接近开源"统一感知+推理脑"**;仍是研究 demo,无持久房间记忆、无 rig 控制。https://huggingface.co/ByteDance/UniVR-34B-Planning
- **Gemini Robotics 1.5→2 + ER(具身推理)1.5→ER 2**(DeepMind,2025)。VLA:视觉+语言→电机控制;ER 变体=空间推理"高层脑"。**闭源、仅 API**(你工作站跑不了)。真实落地、跨本体。统一脑论题的参考架构,但不可部署。https://deepmind.google/models/gemini-robotics/
- **π0 / π0.5**(Physical Intelligence,arXiv:2410.24164 + 2504.16054,945+ 引)。VLM+流匹配动作专家→**连续动作@50Hz**;π0.5 经异质共训做开放世界泛化;**开源**;真实落地。**关键**:π0 是**反应式策略**(感知+语言→动作),**非**下一状态世界模型——**无内部状态/预判/记忆**。它正好回答"直接从感知+状态预测动作"(Q3),但不回答 Q1/Q2/Q4。

### E. 国内/因果世界模型
- **"Embodied AI Agents: Modeling the World"**(arXiv:2506.22355,2025)——主张建 WM 是具身推理核心,并提**学用户的内心世界模型**;直接相关"看着你、和你对话的陪伴"。王亦洲(PKU)"主动因果世界模型"以**综述级主题 + UnrealZoo(ICCV 2025)** 形式出现,非单一地标论文。https://arXiv:2506.22355
- **DreamWorld**(SJTU,含严骏驰合作者)——视频生成里的统一 WM(细节 unconfirmed)。**Spirit v1.5**(清华系)——RoboChallenge 真机榜 #1。
- **清华 LEAP / BAAI ORCA**——**未能作为具名模型核验**(unconfirmed,或内部/命名错配)。清华 FIB Lab 综述是可靠可引产出。

### F. "记忆即世界模型"/预测式记忆——**对你 Q1 的决定性一类**
2025–26 共识:**WM/VLA 长程记忆不足;解法是给它们"外挂"结构化记忆模块**。这与"把记忆折进 WM"相反。
- **EPM "Planning with an Embodied Learnable Memory"**(**ICLR 2026**)—VLM 式可学习记忆用于长程移动操控。https://openreview.net/forum?id=79BOATBal9
- **MEM: Multi-Scale Embodied Memory for VLA**(Physical Intelligence,Finn 组)—给 VLA 加多尺度记忆。https://www.pi.website/download/Mem.pdf
- **3DLLM-Mem: 长程时空记忆**(**NeurIPS 2025**)—动态记忆管理/融合,含 3DMEM-BENCH。
- **SPMEM: 带长期空间记忆的视频世界模型**—几何 grounding 的长期记忆(正治视频 WM 漂移/误差累积)。https://spmem.github.io/
- **HIMM: 类人长期记忆建模**—显式情景+语义记忆给具身 MLLM agent。arXiv:2602.15513
- **Pred-EQA "Predict Before You Explore"**(**CVPR 2026**)—预测式规划+专用记忆做长程具身 QA。
> **含义**:你的显式、可编辑、WorldAnchor 索引的场景图+Graphiti 时序 KG+向量库,**正是研究者在重新发明的外挂模块**。你没落后,你在他们收敛的路径上前置。

## 逐组件:WM 能吸收吗?
| 当前组件 | 可吸收? | 最佳 WM | 得 | 失/险 |
|---|---|---|---|---|
| 立体相机+mesh+SAM3.1+VLM→3D 场景图 | 部分(未成熟) | V-JEPA 2、DINO-WM、Cosmos-3 | 原始视频→潜状态,免手搭 lift | 可查询、WorldAnchor 索引、物体级可编辑;仅仿真/桌面 |
| HOV-SG 分层开放词表 3D 场景图 | **不能** | (无—领域是给 WM 加记忆) | — | 持久信念、语义可编辑、可调试 |
| Graphiti 时序 KG | **不能** | — | — | 时间/事件推理、人格连续 |
| Milvus/BGE-M3 向量检索 | 不能(仅短程 in-context) | EPM/MEM/3DLLM-Mem 式 | 短程 in-context 检索或缩向量库角色 | 跨会话规模检索、精确召回 |
| LLM 脑(推理+人格) | 不能(人格/推理留) | UniVR-34B/Emu3.5(眼+脑合*视觉*通道) | 一个模型"看+推" | 人格、对话、长程规划、可控 |
| VLM"看"工具 | 是(部分) | UniVR-34B、Cosmos-3、V-JEPA-2+readout | 折进统一预测/推理器 | 专精、可换 |
| 屏幕 VLM | 不能 | — | — | 屏幕是房间 WM 没有的模态 |
| 规划/"动作 token"发射 | 部分(短程) | π0.5、Gemini-Robotics、V-JEPA-2 MPC | 感知→动作一步(短程) | 长程意图、手势/姿态/情绪通道 |
| **用户/场景预判(今天外挂)** | **是——干净净赚** | V-JEPA 2、DINO-WM、NavWorld、Cosmos | 内置前瞻替外挂预测器 | 无大险—纯赚,做成服务 |
| motion resolver→RealityKit | 不能 | — | — | WM 不产绑骨动作 |
| Audio2Face-3D / 反射层 / 对话 / 渲染 | 不能 | — | — | — |

## 当前 vs 简化后架构
**当前(5 port):** 感知→场景图;MemoryService(HOV-SG+Graphiti+Milvus);Brain(LLM 人格+推理,用 VLM"看"+屏幕 VLM)→动作 token→MotionResolver→RealityKit+Audio2Face;反射层;对话。

**WM 简化(现实、窄采用):**
- 加一个 **ForesightService(新、窄)**——V-JEPA-2/DINO-WM/Cosmos-3 包一层,输入"当前观测潜+候选动作",输出"预测下一潜/视频"。**预判从此内置——纯赚**。Brain 调它做前瞻。
- MemoryService **不变**(WM 是记忆的*客户端*,非替代):检索到的 HOV-SG/Graphiti 上下文条件化预测器;读侧加"预判预取"(EPM/MEM 模式)。
- Brain 可选 v2:用 UniVR-34B/Emu3.5 式视觉推理 WM 作"看+短程规划"统一模型(实现换,契约不变),人格/对话仍 LLM,A/B 比对。
- 其余(GroundingModel 部分、MotionResolver、反射、对话、渲染)基本不动。

**塌缩的**:外挂预判层→吸收进 ForesightService(挂 Brain/Grounding port 后);"VLM 眼"可并入视觉推理 WM(一个通道)。
**不塌缩(强折会丢的)**:跨会话持久信念、可编辑、可调试、人格/对话、动作 rig、反射、渲染。

## 诚实裁决 + 成熟度风险
**今天 WM 简化你的架构吗?** 边际——只在一处(预判),且是干净低险净赚。其余要么用不上,要么**增加**脆弱。
过度采用的风险:
1. **无当前 WM 有跨会话持久记忆**(V-JEPA 2、DreamerV3、Cosmos-3、π0.5 皆单 episode/单片段)。2025–26 的修法是*外挂记忆*——你已有。
2. **单时间尺度**。V-JEPA 2 明把分层/多时间尺度 JEPA 列为未来工作;你的设计原则是多时间尺度环。**直接替换破坏你的核心原则**。
3. **真实房间 grounding 薄**。SOTA 是桌面操控(DROID)或仿真(DreamerV3)。感知真实躺椅+编码用户+对话的角色,远超已验证 grounding。
4. **误差累积/漂移**——arXiv:2510.16732 点名的头号开放挑战;SPMEM 存在*正是因为*视频 WM 漂移。你的可编辑场景图是可调试的解药。
5. **不可调试黑箱**。你的场景图+KG 可检视/可编辑;潜 WM 不可。对"跨会话记忆"的陪伴,可调试是产品要求。
6. **对话/人格碰不到**。WM 不会对话/持人格——LLM 脑无论如何得留。
**简化值不值成熟度险?** 对*预判*:值,低险,现在做(挂 port)。对*塌缩记忆/脑*:不值——会用一个工作、可调试、对齐前沿的系统换一个目前需要*更多*组件的研究 demo。

## 迁移路径(经你的 Brain/MemoryService port)
你的 port 化架构是决定性优势。增量迁移,每步是契约版本 bump,非重写:
1. **现在(低险、契约加法)**:加 **ForesightService**(新、窄)或作 Brain 内"predict"工具,包 **V-JEPA-2(1.2B)** 或 **DINO-WM**。输入当前观测潜+候选动作,输出预测下一潜/视频。Brain 用它做前瞻。**不动现有 port**。占 96GB 一角。实现那唯一干净的简化。
2. **Brain port v2(实验)**:`WorldModelBrain` 备选实现,吐*同一动作 token 契约*,内部用 **UniVR-34B/Emu3.5** 统一"看+短程规划"。人格/对话留 LLM。A/B 比短程反应。契约不变、实现换。
3. **MemoryService——别替换**:v2 = "PredictiveMemoryService":WM 成记忆*客户端*(检索 HOV-SG/Graphiti 条件化预测器),读侧加预判预取。EPM/MEM/3DLLM-Mem 模式直接映射。
4. **GroundingModel v2**:WM(Cosmos-3/DINO-WM)可吞部分 3D-lift+grounding——部分吸收候选,但保留显式锚点索引输出以便调试。
5. **PerceptionSource / MotionResolver——不动**。
**"WM 当全脑"可重访的信号**:(a) **分层/多时间尺度 JEPA** 发布;(b) WM 无外挂图即展示**跨会话持久记忆**(EPM/MEM 线并回基础模型);(c) **真实房间、对话式具身** grounding(非桌面);(d) Cosmos/V-JEPA 后继暴露**可编辑/可调试**状态。

## 最该盯的 5 个(未来简化候选)
1. **V-JEPA 2 + 分层/多时间尺度 JEPA 路线图(Meta)**——多时间尺度议程*正是你的设计原则*;分层 JEPA 出来时重评能否塌缩你的快/慢环。https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/
2. **UniVR-34B / Emu3.5(字节/BAAI)**——开源 34B、视觉空间推理、自回归 next-token WM;最接近可部署(96GB 可载)的"统一感知+推理"候选。
3. **MEM(PI)+ EPM(ICLR2026)+ SPMEM**——告诉你*何时*记忆在模型内成熟到可重评替换外部记忆栈。它们并回基础模型前,留你的图。
4. **NVIDIA Cosmos 3 及后继**——开源、全模态、预测视频+动作;若出持久状态/记忆增强版,最可能是"一个基础模型"基底。
5. **统一-WM-即-脑 论文**——NeurIPS 2025 workshop 观点文、**WHALE**(周志华组)、2026 具身 WM 综述。盯"统一"从*观点*变*已验证*。

## 来源(关键)
NeurIPS2025 Embodied World Models workshop https://embodied-world-models.github.io/ ;arXiv:2510.16732 综述 https://arxiv.org/abs/2510.16732 + https://github.com/Li-Zn-H/AwesomeWorldModels ;Shang 2026 综述 https://www.preprints.org/manuscript/202604.0928 ;arXiv:2509.20021 ;清华 FIB Lab World-Model CSUR2025;V-JEPA 2 https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/ + https://github.com/facebookresearch/jepa-wms ;DreamerV3 Nature2025/arXiv:2301.04104 + DreamerV4 https://danijar.com/project/dreamer4/ ;DINO-WM https://dino-wm.github.io/ ;NWM https://www.amirbar.net/nwm/ ;Cosmos 3 https://research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf ;Genie 2/3 DeepMind blog;Sora 2 + Duality 批评;UniVR-34B https://huggingface.co/ByteDance/UniVR-34B-Planning + Emu3.5 arXiv:2510.26583;Gemini Robotics https://deepmind.google/models/gemini-robotics/ ;π0 arXiv:2410.24164 + π0.5 arXiv:2504.16054 + https://huggingface.co/blog/pi0;arXiv:2506.22355;EPM ICLR2026 https://openreview.net/forum?id=79BOATBal9 ;MEM https://www.pi.website/download/Mem.pdf ;3DLLM-Mem NeurIPS2025;SPMEM https://spmem.github.io/ ;HIMM arXiv:2602.15513;Pred-EQA CVPR2026。

**未确认**:DreamerV4 细节、SJTU DreamWorld 细节、"清华 LEAP/BAAI ORCA"作为具名模型未能核验。
