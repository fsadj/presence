# VLA / Qwen-VLA — 给 Vision Pro 虚拟角色(2026-08,串行调研)

> 用户重点:VLA > 纯 LLM;尽量新。结论已含 2026 最新(Qwen-VLA、SIMA 2、TurboVLA、OpenVLA-OFT、GR00T N1.7、Gemini Robotics-ER 1.5)。

## 一句话结论
1. **Qwen-VLA 真实存在**(官方 GitHub `QwenLM/Qwen-VLA`,arXiv 2605.30280,2026-05):~5B(Qwen3.5-4B 视觉-语言主干 + 1.15B DiT 流匹配动作解码器),开源,真机 ALOHA 上超 GR00T N1.6 和 π0.5。
2. **机器人 VLA 对你的虚拟角色"形状错了"**。Qwen-VLA/π0/GR00T/OpenVLA 都吐*连续末端/关节轨迹*(7-DoF 位姿、关节角或 chunked action 向量)。你的"本体"是 Blender rig + RealityKit IK + navmesh + blendshape——这些连续力矩/位姿输出没有自然映射。硬塞是倒退。
3. **正确范式(领域已在收敛)**:**VLM"大脑"吐结构化动作 token(意图/手势/注视/情绪/导航/语音)+ 一层薄的 action→animation**。这正是阿里 **TaoAvatar**(Qwen2.5 LLM + ASR/TTS/音→blendshape + 神经渲染,明确 Apple-Vision-Pro 兼容)与 DeepMind **SIMA/SIMA 2**(动作是虚拟世界的键鼠,非机器人关节)的架构。
4. **推荐 build**:感知(Vision Pro LiDAR + 场景 mesh + passthrough)→ **RTX Pro 6000 上跑 Qwen3-VL/Qwen2.5-VL** VLM 吐小结构化 JSON/"action-token"流(手势/注视目标/情绪/导航点/语音)→ RealityKit 侧 action mapper 以 90Hz 驱动 IK/navmesh/blendshape/TTS。你得到 VLA *范式*(感知→推理→动作)却避开机器人形状的动作空间。完整机器人 VLA 仅留给"未来要物理仿真角色操控物体"时。

## Qwen VLA — 最新事实(2026 中)
官方 repo(github.com/QwenLM/Qwen-VLA)+ arXiv 2605.30280 确认:
- **名**:Qwen-VLA(Qwen-VLA-Base / Qwen-VLA-Instruct)。
- **架构**:Qwen3.5-4B 视觉-语言主干 + **1.15B DiT 流匹配动作解码器**(总 ~5B+)。
- **动作表示**:**连续动作+轨迹**(流匹配);统一框架跨操控、导航、第一人称动作建模、轨迹预测。
- **本体处理**:"Embodiment-aware prompt conditioning"—一套权重,改文本 prompt 切换本体,无 per-platform 输出头。
- **训练**:大规模动作预训练 → 多模态继续预训练 → SFT → RL。
- **开放**:repo 称代码+权重"领先开源 VLA"。**License/HF 权重路径部分 unconfirmed**(Qwen 家族几乎都是 Apache-2.0;用前在 huggingface.co/Qwen 核)。
- **真机 ALOHA(域内成功率%)**:Qwen-VLA-aloha(带预训练)**83.6** vs π0.5 71.6 vs GR00T N1.6 28.6。
- **ALOHA OOD**:Qwen-VLA **76.9** vs π0.5 41.5 vs GR00T N1.6 25.4。
- **仿真(Instruct)**:LIBERO 97.9、RoboCasa-GR1 56.7、Simpler-WidowX 73.7、RoboTwin-Hard 87.2、R2R-SR 57.5、RxR-SR 59.6。
- **关键概念**:Qwen-VLA 动作解码器产**机器人末端/关节的连续轨迹**,非离散动画指令。这是它**非** Blender 虚拟人即插大脑的核心原因。

**相关 Qwen 生态**:
- **Qwen-Robot Suite / Qwen-RobotManip**—阿里 2025-01 早期具身套件。
- **RynnVLA-001**(阿里 DAMO)—视频生成为底座的 VLA。
- **WorldVLA**—~2025-09 开源。
- **TaoAvatar**(阿里/PixelAI,arXiv 2503.17032,2025-03)—*非*机器人 VLA,而是全身 3D talking avatar(SMPL-X + 3DGS),端侧 **Qwen2.5** LLM 大脑 + ASR + TTS + 音→blendshape(A2BS)+ 神经渲染。开源移动 demo `alibaba/MNN/apps/Android/Mnn3dAvatar`,项目页明确列 **Apple Vision Pro**。这是"Qwen 驱动虚拟角色"最接近的现存物。

## VLA 全景(2023–2026)
| 模型 | 机构 | 规模 | 开源? | 动作表示 | 备注 |
|---|---|---|---|---|---|
| RT-1/RT-2/RT-X | Google | ~35B(RT-2)| 闭 | 离散动作 token | RT-2=首个会动作的 VLM;Open X-Embodiment 跨本体 |
| Gemini Robotics/-ER 1.5 | DeepMind | 未披露(Gemini 2.0)| 闭(API/合作)| 直驱电机+代码生成;"On-Device"低延迟变体 | ~2× 泛化;ER=具身推理/规划 |
| OpenVLA | Stanford/Berkeley/TRI | **7B** | 开(Apache-2.0)| 离散动作 token(逐步)| arXiv 2406.09246;A100 ~5Hz(优化 ~12Hz)|
| OpenVLA-OFT | 后续 | 7B | 开 | token chunks | **生成快 26×、延迟低 3×** |
| TurboVLA | — | **0.2B** | 开 | — | **消费级 RTX 31ms/~32Hz、0.9GB VRAM** |
| Octo | Octo Team | ~93M | 开 | 扩散策略 | 80万 Open X-Embodiment 轨迹 |
| π0/π0.5/π0-FAST | Physical Intelligence | ~2B(PaliGemma)| 开(`Physical-Intelligence/openpi`)| **连续流匹配动作 chunk** | 首个流匹配 VLA;强开放世界泛化 |
| RDT-1B | 清华 | **1B** | 开 | 扩散(预测后 64 动作)| 双臂操控 |
| CogACT | 微软 | 变 | 开 | 分离认知+动作通路 | 少样本适应 |
| GR00T N1/N1.5/**N1.7** | NVIDIA | **2B** | 开(`nvidia/GR00T-N1-2B`)| 双系统 VLA,人形动作 | arXiv 2503.14734;最新 N1.7 |
| Figure Helix | Figure AI | 未披露 | 部分/闭 | **单网高频连续全身上半身控制**(指/腕/躯/头)| 驱动 Figure 02/03 |
| SpaceVLA/Camera-Space VLA | 研究 | — | — | 相机坐标推理跨本体 | 新兴方向 |
| InternVLA A1/M1 | 上海AI Lab(InternRobotics)| — | 开 | VLA+推理+RL | 报道真机超 π0 与 GR00T N1.5;适配 Galbot/智元/青龙人形 |
| WholebodyVLA/GraspVLA/VeBrain | 上海AI Lab/PKU(王鹤)/智源 | — | 开(研究)| 全身/抓取中心 | 中国学术具身栈 |
| RoboBrain 2.0 | 智源 BAAI | — | 开 | 具身脑 | 中国旗舰具身 FM |
| AGIBOT/智元 全身 VLA | AGIBOT+HKU | — | — | 双足全身协调 | 走+操控 |
| **SIMA / SIMA 2** | DeepMind | Gemini 驱动 | 闭 | **3D 虚拟世界的键鼠**(非机器人关节)| **最接近你用例的先例**;arXiv 2512.04797 |

## VLA 给虚拟角色:动作如何映射到动画 rig
机器人 VLA 的"动作"=关节力矩/7-DoF 末端位姿。虚拟角色"动作"=blendshape 权重、IK 目标、注视射线、navmesh 航点、情绪标签、语音。桥接研究四类:
- **(a) 动作 token 映射到动画指令(非力矩)**。token-VLA(RT-2/OpenVLA)原则上本体无关:定义*离散动作 token 词表*并训练模型吐。对虚拟角色定义 `&lt;gesture:wave&gt;`、`&lt;gaze:user_face&gt;`、`&lt;walk_to:(x,y)&gt;`、`&lt;emotion:happy&gt;`、`&lt;speak:"..."&gt;`。arXiv 2505.21531("How Much Do LLMs Know about Human Motion?")显示 LLM 能做好*高层*计划→身体部位位置分解,但*精确*空间/DoF 定位弱—这正是下方留专门 motion/IK 层的原因。
- **(b) LLM/VLM 作动作控制器**。MotionGPT(NeurIPS 2023)把人体动作当"外语"token 化、文生动作;AvatarGPT/OmniControl/MotionLLM/T2M-GPT 皆语言→动作(姿态序列)。产可驱动 rig 的姿态序列,但是开环片段生成,非实时反应式感知→动作。
- **(c) 游戏/仿真 agent 吐结构化动作(最强先例)**。DeepMind **SIMA** 字面就是动作空间=3D 虚拟世界键鼠的 VLA—感知→推理→结构化虚拟动作,无机器人。**SIMA 2**(2025,Gemini 驱动)扩展。Voyager/GITM/DEPS 做 LLM 吐结构化动作(代码/API)于 Minecraft。这是 VLA 范式当"动作=给仿真身体的指令"时可行的最清晰现存证明。
- **(d) 已部署虚拟角色管线**。阿里 **TaoAvatar**(arXiv 2503.17032)是 Qwen 生态最直接工业先例:端侧 Qwen2.5 LLM + ASR + TTS + 音→blendshape + 神经渲染,实时,Apple-Vision-Pro 目标。**Look2React**(IEEE TVCG 2026)做 VR NPC 基于视觉推理动态选姿态+文本反应。**Meta Horizon** 出具身对话 LLM NPC。
- **RealityKit 实操模式**:把 rig + IK + navmesh + blendshape 当你的"本体"。动作空间定义为小 JSON schema(gesture id、gaze target=世界 Transform、locomotion target=navmesh 点、emotion 标签 viseme/blendshape 集、TTS 串)。VLM 每认知 tick 吐一条记录;RealityKit 90Hz 插值。这*正是*TaoAvatar 管线形状—他们只是用 3DGS 渲染而非 RealityKit。

## 可行性(M1 Max + RTX Pro 6000 96GB)
**RTX Pro 6000 96GB(PC)— 过配,良性**:
- 7B VLA(OpenVLA-7B)单 A100 ~5Hz 基线/~12Hz 优化;RTX Pro 6000 同级且 96GB,还能跑 13B–34B VLM 或 Qwen-VLA(~5B)全精度有余。
- OpenVLA-OFT 报生成快 26×/延迟低 3×。**TurboVLA(0.2B)报消费级 RTX 31ms(~32Hz)、0.9GB VRAM**—证明小型专用动作策略 RTX 上实时。
- π0/openpi(~2B)、GR00T N1(2B)、RDT-1B 皆轻松承载。
- **结论**:任何当前开源 VLA/VLM 舒适跑;1–10Hz *认知 tick*(与 90Hz 渲染解耦)延迟非问题。1–10Hz 是标准机器人 VLA 区间,对角色"思考"节奏完全可接受—人类对直接操控 <~100ms 延迟敏感,但角色*刻意*反应 200–500ms 读起来自然。

**M1 Max 32GB(Mac)— 经 MLX 可跑小 VLM/VLA,但更紧**:
- MLX-VLM 跑 Qwen2-VL-2B:M1 级 ~8–15 tok/s,M2 Max 15–20,M3 Max 20–25。
- mlxcel M1 Max 跑 3B:decode 200–700 tok/s(小模型)。
- MLX 在 Apple Silicon 上比 llama.cpp 快 ~1.8–3×;过 ~40k context 优势缩小。
- **结论**:2–4B Qwen-VL/Qwen3-VL(或量化 7B)可在 M1 Max 本地跑(全端侧路径),但把 Mac 当渲染/感知前端、RTX PC 当推理服务器(千兆/Wi-Fi 6E)会得到更丰富行为。Apple Core ML 也能导出小 VLM,但 MLX 是当下更好支持的路。

## 推荐架构 & 首读 5 项
```
[Apple Vision Pro] → passthrough + LiDAR + 场景 mesh + 手/眼/脸追踪
        │  (低带宽:降采样 RGB-D 裁剪、用户位姿、场景基元)
        ▼
[VLM"大脑"于 RTX Pro 6000 PC]   ← Qwen3-VL 或 Qwen2.5-VL(或 Qwen-VLA-Instruct
        │                         若要真连续轨迹输出)
        │  吐结构化动作记录(~1–5Hz):
        │  {gaze_target, gesture_id, emotion, locomotion_waypoint, speech_text, prosody}
        ▼
[Mac 上的 action→animation 映射器]
   • gaze_target  → RealityKit LookAt 约束
   • gesture_id   → 触发 Blender 烘焙动画片段 / IK 姿态目标
   • emotion      → blendshape 权重(ARKit blendshapes,Vision Pro 原生)
   • waypoint     → RealityKit navmesh Steering/Pathfinding
   • speech       → TTS(端侧)→ visemes → lip-sync blendshapes
        ▼
[RealityKit 90Hz 渲染角色]
```
**首读 5 项**:
1. **TaoAvatar**(arXiv 2503.17032)+ `alibaba/MNN/apps/Android/Mnn3dAvatar` 代码—最接近你目标系统的现存物,Qwen 生态,Vision-Pro 感知。学 LLM→A2BS→render 管线。
2. **Qwen-VLA**(arXiv 2605.30280,github.com/QwenLM/Qwen-VLA)—动作解码器设计与本体 prompt 条件思路(即便你多半不用其连续轨迹头)。
3. **SIMA / SIMA 2**(DeepMind blog;arXiv 2512.04797)—证明 VLA 可面向*虚拟*动作空间(键鼠)而非机器人关节;对你项目最清晰的概念许可。
4. **OpenVLA-OFT**(openvla-oft.github.io)与 **TurboVLA**(arXiv 2607.27205)—让近实时角色认知在 RTX 级 GPU 上可行的延迟预算。
5. **"How Much Do LLMs Know about Human Motion?"**(arXiv 2505.21531)+ **MotionGPT**(OpenMotionLab/MotionGPT)—界定 LLM/VLM 在*动作*层能/不能做什么,及何处必须交给专门 motion/IK 策略。

## 来源(关键)
- Qwen-VLA repo https://github.com/QwenLM/Qwen-VLA ;arXiv https://arxiv.org/abs/2605.30280 ;HF 论文 https://huggingface.co/papers/2605.30280 ;Qwen blog https://qwen.ai/blog?id=qwenvla
- 阿里云 Qwen-Robot Suite blog https://www.alibabacloud.com/blog/entering-the-physical-ai-era-introducing-the-qwen-robot-suite_603261 ;SCMP https://www.scmp.com/tech/big-tech/article/3357260/
- RynnVLA-001(DAMO)https://huggingface.co/blog/Alibaba-DAMO-Academy/rynnvla-001
- π0 blog https://www.pi.website/blog/pi0 ;openpi https://github.com/Physical-Intelligence/openpi ;论文 https://arxiv.org/html/2410.24164v1
- GR00T N1 论文 https://arxiv.org/abs/2503.14734 ;权重 https://huggingface.co/nvidia/GR00T-N1-2B ;Isaac-GR00T(N1.7)https://github.com/Nvidia/Isaac-GR00T
- OpenVLA 论文 https://arxiv.org/html/2406.09246v3 ;repo https://github.com/openvla/openvla ;OFT https://openvla-oft.github.io/ ;TurboVLA https://www.alphaxiv.org/abs/2607.27205
- Octo https://octo-models.github.io/ ;RDT-1B https://github.com/thu-ml/RoboticsDiffusionTransformer ;CogACT https://github.com/microsoft/CogACT
- Figure Helix https://www.figure.ai/news/helix
- Gemini Robotics blog https://deepmind.google/blog/gemini-robotics-brings-ai-into-the-physical-world/ ;论文 https://arxiv.org/html/2503.20020v1 ;ER 1.5 https://developers.googleblog.com/building-the-next-generation-of-physical-agents-with-gemini-robotics-er-15/
- SIMA blog https://deepmind.google/blog/sima-generalist-ai-agent-for-3d-virtual-environments/ ;SIMA 2 https://deepmind.google/blog/sima-2-an-agent-that-plays-reasons-and-learns-with-you-in-virtual-3d-worlds/ ;论文 https://arxiv.org/html/2512.04797v1
- Voyager https://voyager.minedojo.org/ ;LLM 游戏 agent 综述 https://arxiv.org/html/2404.02039v2
- MotionGPT https://github.com/OpenMotionLab/MotionGPT ;"LLMs Know about Motion?" https://arxiv.org/abs/2505.21531
- TaoAvatar 论文 https://arxiv.org/html/2503.17032v1 ;项目 https://pixelai-team.github.io/TaoAvatar/ ;MNN 3D Avatar demo https://github.com/alibaba/MNN/blob/master/apps/Android/Mnn3dAvatar/README.md
- 上海AI Lab 具身开源周(InternVLA A1)https://www.shlab.org.cn/news/5444209
- Look2React(IEEE TVCG 2026)https://www.computer.org/csdl/journal/tg/2026/05/11459367
- Meta Horizon 具身 LLM NPC https://developers.meta.com/horizon/blog/worlds/environment-generation-embodied-conversational-llm-npcs-genai-tools/
- MLX-VLM Mac 基准 https://byteiota.com/run-vision-ai-on-mac-with-mlx-vlm-free-gpt-4v-alternative/ ;MLX vs llama.cpp https://pub.towardsai.net/apples-mlx-runs-local-llms-3x-faster-than-llama-cpp-until-your-context-hits-40k-715ec441afbb
- VLA 综述 https://www.alphaxiv.org/abs/2405.14093v8

## 未确认项
- Qwen-VLA 本身确切 HF 权重路径与 license 串(Qwen 家族默认 Apache-2.0;repo 称开源,但未能直接 fetch `Qwen/Qwen-VLA` HF 卡)。在 huggingface.co/Qwen 核。
- Figure Helix、Gemini Robotics 及部分中国学术模型(InternVLA A1/M1、WholebodyVLA、GraspVLA、VeBrain、RoboBrain 2.0)的规模/license 单元格—确认存在且总体开源,但确切参数量在源中不一。
