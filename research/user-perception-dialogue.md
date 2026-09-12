# 多模态感知用户 + 屏幕感知 + 对话/TTS 延迟 — 给 Vision Pro 角色(2026-08,串行,优先最新)

> 角色"感知用户、实时反应、且看你敲代码"的决策级调研。含屏幕感知这一新颖通道。

## 一句话结论
- **可行**。你这套硬件(RTX Pro 6000 Blackwell 96GB + M1 Max 32GB + Vision Pro)是 2026 中**自托管、低延迟、全隐私反应式角色的甜点配置**。单 96GB Blackwell 卡可同时驻留 70B 级对话 LLM + 屏幕 VLM + 流式 ASR + 流式 TTS,KV-cache 余量充足。
- **对话环路是已解决模式,不是研究问题**。2026 业界共识(OpenAI "GPT-Live" 2026-07 后)是**混合流式管线 + 全双工 barge-in**,目标**端到端 300–600ms**,打断处理 **~200ms 感知天花板**。主流 "Dualplex":实时 LLM 推理路径 + 独立的高质量流式 TTS。
- **真正新且难的是你的屏幕感知通道**——让角色*看 Mac 上你写代码*。模型已有(UI-TARS-1.5、OmniParser、OS-Atlas、基于 Qwen2.5-VL 的 grounded agent),但它们为**操作**屏幕而生,不是**常驻陪伴式感知+评论**。改造 = 只跑便宜的"感知/理解"半边,**事件触发 + 周期**而非每帧,把活动态融合进对话上下文 + 身体/注视 IK(看向虚拟屏幕)。
- **推荐栈**:Mac/Vision Pro 端侧感知(MediaPipe/CoreML 面部+注视+手势、Apple Speech ASR、ScreenCaptureKit)→ LAN 流到**工作站**(vLLM 对话 LLM + Qwen2.5-VL/InternVL 屏幕 VLM + FlashTTS/Faster-IndexTTS-2 流式 TTS)→ 音频/状态回传。barge-in 检测本地(Silero VAD + 置信加权 partial),中断延迟独立于网络。
- **中文端到端强**:SenseVoice/FireRedASR/Qwen3-ASR(ASR)、Qwen3-Omni/GLM-4-Voice/Step-Audio 2(对话+语音)、IndexTTS-2/CosyVoice2/Spark-TTS(TTS)皆 ZH 优先/强且可自托管。

## 1. 流式 ASR & 对话轮次(最新在前)
### 流式 ASR 2026 榜
- **NVIDIA Canary-1B-v2 / Parakeet-TDT-1.1B**(CC-BY-4.0)英文 ASR 领先 HF Open ASR Leaderboard **~6.7–6.9% avg WER**;Canary-1B-v2 **超 Whisper-large-v3 且快 ~10×**(arXiv:2509.14128)。Parakeet-TDT 单 A100 ~**50× 实时**,低延迟流式 + VAD 首选。
- **Canary-Qwen 2.5B、IBM Granite Speech、Qwen3-ASR** 是 2026 最新混合 ASR-LLM,**WER 超 Whisper**(Qwen3-ASR 对中英尤相关)。
- **Whisper Large v3 Turbo**(~0.8B,MIT)仍是最部署的*多语*模型(~99 语言,~7.8% WER)。
- **中文**:**SenseVoice**(阿里/FunASR)CER ~8.0% 超 Whisper(普/粤);**FireRedASR**(开源,工业级普通话+方言+英);**Paraformer**(FunASR)强非自回归流式。
- **端侧**:**Moonshine Tiny/Base**(27M/61M);**Distil-Whisper Large v3** 保 ~99% 质量快 ~6×。
- **Apple Speech 框架**(端侧,免费,M1 Max & visionOS)零依赖基线——无流式 WER 公开数,但摩擦/延迟最低、内置中文;需 Whisper 级精度时本地配 faster-whisper/whisper.cpp。
- 流式/端侧权威基准:2026 arXiv 综述 "Pushing the Limits of On-Device Streaming ASR"(arXiv:2604.14493),50+ 配置(PDF 未能直 fetch,经搜索摘要核验)。

### VAD/端点/barge-in/全双工轮次(2026 SOTA)
- **"自然"的延迟预算**:人际轮间 ~200ms(日 −7ms 重叠,丹麦 ~469ms)。**>500ms** 感觉卡;**>800ms** 开始抢话;**>1s** 接触中心数据可测挂断。**ITU-T G.114** 单程 150ms 最优。
- **~200ms barge-in 窗**:用户打断时,系统有 ~200ms 完成:检测语音起始→取消外发 TTS→清空在途音频缓冲→重评上下文→起新响应("Dualplex")。
- **组件预算(优化栈典型)**:网络入 30–60ms(共址)/100–200ms(跨区);流式 ASR 150–300ms;端点检测(VAD+静音阈值)50–150ms;LLM 首 token 150–400ms(实时模型)/300–1000ms(标准);**TTS 首音频 TTFA:Cartesia Sonic ~90ms(SSM 架构)、ElevenLabs <100ms、标准神经 TTS 200–500ms**;网络出 30–100ms。
- **三种架构模式**:(1) 顺序级联 STT→LLM→TTS(800–2000ms,无 barge-in,太慢);(2) 端到端 S2S(300–600ms,模型原生 barge-in,语音控制有限:OpenAI Realtime/Gemini Live/Moshi);(3) **混合流式**(partial 转录→流式 LLM token→流式 TTS 并行,500–900ms,STT 层 barge-in,全语音控制)+ 2026 "Dualplex" 变体(实时 LLM 路径 + 独立 premium TTS→300–600ms 高音质)。
- **barge-in 最难工程**:声学回声消除 AEC(角色别听到自己 TTS);**置信加权 partial 转录**(避免低置信 STT 假打断);**音素边界 mid-utterance TTS 取消**(需 TTS 暴露缓冲位置元数据);打断后上下文管理;中英文中途热切。
- **2026 新研究**:arXiv:2604.21406(全双工 SDS 交互)、arXiv:2607.07148(解耦全双工对话动力学,显式建模 barge-in 后让话、backchannel 生产、轮启延迟)。VAD:**Silero VAD** 仍是开源事实标准。

## 2. 情绪/表情/注视/手势/韵律感知(最新在前)
- **FER**:**MediaPipe FaceLandmarker**(478 关键点 + **52 blendshapes** 近 FACS)实时跑 M1 Max/Vision Pro,实用基线;Apple 原生 **Vision/ARKit 面部追踪**等效无第三方依赖。**BlendFER-Lite**(Frontiers Neurorobotics 2025,arXiv:2501.13432)—LSTM over blendshapes 分类情绪,**直接可部署**。blendshape→FACS AU 映射成标准桥(ScienceDirect 2026)。**EmoNet-Face**(NeurIPS 2025)—40 类情绪基准。**HSEmotion** 仍被引但无大更新。工具:**Py-Feat**、iMotions+Affectiva。
- **注视**:**GazeOnce**(CVPR2022,多人实时参考)、**L2CS-Net**(arXiv:2203.03339,细粒度单人头偏航/俯仰,实时)。**GAZE 2026 @ CVPR** 活跃场地。**CHI 2026 综述**(103 研究)确认注视+语音是空间 grounding 经典配对——正是你角色所需。**Vision Pro 已原生追踪用户眼睛**——可能直接读注视目标(屏幕 vs 角色 vs 他处)而无需 GazeOnce。
- **手势/韵律/多模态融合**:visionOS **HandTracking** 原生关节;MediaPipe Holistic/GestureRecognizer 跨平台 fallback。语音情绪 SER 现由**图+SSL 语音编码器**融合文本主导。多模态融合(face+voice+text+gaze)2025–2026 SOTA 经 transformer/时序融合;**关键**:在*决策/状态层*融合(每帧"用户情感态"向量)而非原始传感器层——便宜且抗模态缺失。

## 3. 屏幕感知 — 新颖通道(重点)
三子问题:**(a) 截屏 (b) 理解 (c) 让角色物理地看并反应**。

### (a) 截屏:Mac ScreenCaptureKit → 工作站 VLM
- **ScreenCaptureKit**(macOS/visionOS/iOS)高性能截屏;Mac 截**自己屏幕**流到 RTX 工作站(Vision Pro 看不到 Mac 屏,但 Mac 能截自己)。
- **LAN 传输**:社区共识(Reddit r/VisionPro 2025-12;KhaosT gist)低延迟本地串流首选 **NDI**;WebRTC 或 gRPC/WebSocket+JPEG/WebP 亦可。VLM 输入**不需 60fps**—1–5fps 压缩 1080p/4K 够,**GbE 带宽微不足道**(~1–5MB/s,8–40Mbps)。Mac→工作站跳几乎免费。

### (b) 理解:屏幕理解模型(把 GUI agent 从"操作"改"看")
2024–2026 GUI agent VLM 爆发,直接可改造。它们为*定位 UI 元素并点击*而生,但感知半边(什么 app/活动/代码错误/窗口)正是看屏陪伴所需。
- **UI-TARS-1.5**(字节,开源,2 周 2.7万 star;arXiv:2501.12326)—"All-in-One"原生 GUI agent VLM,*只感知截图*并推理 UI 元素/布局/文本/图标。**UI-TARS Desktop**(2026)字面"看你屏幕理解 UI"的开源 GUI agent。**屏幕理解最强单选**;可提示其*描述*而非*操作*。
- **OmniParser**(微软)—纯视觉屏幕解析→结构化元素。便宜预处理/grounding 阶段。
- **OS-Atlas**(OS-Copilot,ICLR2025,arXiv:2410.23218,330+ 引)—GUI grounding 基础动作模型,接受任意图尺寸。
- **SeeClick**(ACL2024,596 引)—视觉 GUI grounding,创 ScreenSpot 基准。
- **GUI-Actor**(微软,NeurIPS2025)—**Qwen2.5-VL** 骨干上坐标无关 grounding;**GUI-G1**(NeurIPS2025)R1 式 RL grounding。**Qwen2.5-VL 是主导骨干**—单个 Qwen2.5-VL-72B(装得下 96GB)同时给你屏幕理解+通用视觉。
- **Claude/GPT computer-use** API 专有替代(不自托管),质量天花板参考。
- **改造模式(关键架构洞见)**:屏幕 VLM**周期(每隔几秒)+ 活动变化触发**(切窗、构建事件、空闲超时)运行,产紧凑**活动 token**("VSCode 调 Python,42 行报错"/"看文档"/"空闲 3 分")注入对话上下文—*非每帧*。这保持 VLM 成本可预测,命中 <500ms 对话预算。

### (c) 让角色物理地看屏幕
- **3D 定位屏幕**:用户配置锚点放虚拟 Mac 窗口(RealityKit world/page anchor),或世界 mesh 平面检测。配置锚点最简最稳。
- **朝向**:RealityKit **IK rig + 约束**(Apple "Character control, skeletons, IK")。devforums 线程 797407"mix Animation and IKRig"直接对点*播放 idle 时看向位置*。"Placing entities using head and device transform"样本给头变换作 look-at 源。WWDC25 session 287/317 最新 API。
- **行为状态机**:"陪伴-注视你工作"=融合(a)idle 呼吸/眨眼 (b)默认 look-at-IK 指向屏幕锚 (c)轮次让出时周期瞥用户脸 (d)检测到屏幕事件时微反应(点头/挑眉:构建成功/失败)。用上面活动 token 作触发。
- **常驻先例**:**VisionClaw**(arXiv:2604.03486,2026)—智能眼镜常驻 agent,Gemini 作上下文感知助手—最接近你"常驻感知屏幕/上下文"的已发表先例。

## 4. 反应式对话 & 流式 TTS(最新在前)
### 流式 TTS 2026(可自托管、ZH 强)
- **FlashTTS**(ASLP-lab,arXiv:2606.09141,2026)—Multi-Token Prediction 快流式 TTS,**~325ms 首包**,零样本克隆+多说话人。原生流式,强选。
- **Faster IndexTTS-2**(arXiv:2607.21042,2026)—TensorRT-LLM 加速 IndexTTS-2 + **分块流式降 TTFA**;批量推理提吞吐。**IndexTTS-2**(arXiv:2506.21619)最富表达开源 ZH TTS(首个 AR 零样本 TTS + 精确自然时长控制)但*非*原生流式。
- **Cartesia Sonic/Sonic-Turbo**—~**90ms/~40ms TTFA**(SSM 架构),商业低延迟领先;驱动 Realtime TTS Arena。
- **Qwen3-TTS**(阿里,2026-01 开源)—语音克隆+设计,10 语言含中;经 **vLLM-Omni**(OpenAI 兼容端点)服务。
- **CosyVoice 2**(阿里)—强 ZH 零样本克隆+表达,部分流式。**Spark-TTS**(0.5B)小快中;**GPT-SoVITS**—1 分钟少样本克隆(ZH/JP/KR/EN)但"挑剔";**F5-TTS**—非自回归 flow-matching+DiT,克隆优但**无原生流式**(不宜作主对话 TTS)。StyleTTS2 实时基准中不突出,被上者取代。
- **Apple AVSpeechSynthesizer + 端侧神经语音**(visionOS)—免费、首音频零延迟(本地)、零网络,但表达/可控性弱于开源模型。**最佳作即时 fallback/backchannel**("mhm"、点头),神经 TTS 预热时顶上。
- **Arena**(Artificial Analysis Speech Arena):**Simba 3.2** 总 TTS Elo 领先(1229);**Inworld Realtime TTS-2**(2026-05)领 Realtime TTS Arena;**Step-Audio R1.1** 领 Full-Duplex Bench。

### 端到端 S2S(替代架构)
- **Moshi**(Kyutai,arXiv:2410.00037,662 引)—7.6B 联合语音-文本,双流("内心独白"文+音),**首个开源 sub-200ms 全双工**对话。
- **GLM-4-Voice**(清华/智谱,~9B)—端到端中英,实时,**情感表达丰富**,全可自托管。
- **Qwen3-Omni-30B-A3B-Instruct**(阿里,arXiv:2509.17765)—**原生全模态:文+图+音+视频**,流式文+语音输出。**战略有趣**:单模型原则可同时处理屏幕 VLM 任务*和*语音对话(它摄取图),简化栈—但单次成本高于专用拆分。
- **Step-Audio 2/2 Mini**(StepFun,arXiv:2507.16632;8B Mini)—全双工,**基准超 GPT-4o-Audio**。
- **MichiAI**(Reddit)—530M 全双工语音 LLM,flow matching **~75ms 延迟**,轻量选项。

### Backchanneling & 听众行为
端到端 S2S(Moshi/Step-Audio/GLM-4-Voice)产**模型原生 backchannel**("mhm"/"yeah")+ 重叠感知轮次—通往*听众*角色最便宜。级联架构中 backchannel 须**脚本化为独立短 TTS 事件**,由 VAD/轮次让出检测触发,角色看向用户 + 点头动画。研究锚:arXiv:2607.07148、EACL2026 findings(手势+注视+语音轮次,aclanthology 2026.findings-eacl.106)。

## 5. 可行性(工作站 96GB + Mac 32GB + Vision Pro)
### 工作站(RTX Pro 6000 Blackwell,96GB GDDR7 ECC,PCIe Gen5)
- **vLLM 官方支持该卡**(forum #1707);连续批+paged attention。
- **单 96GB 卡可装**:7–14B(Q4)4–9GB;**30B AWQ/FP8 ~18–20GB**;**70B(Llama3.3/Qwen2.5-72B)FP8 ~35–40GB,KV cache 余 ~50GB+**;VLM:Qwen2.5-VL-72B(FP8/Q4)、InternVL2.5-78B(Q4)、Llama-3.2-Vision-90B(Q4)皆可。
- **并发预算**:单卡可同时驻留 **70B 对话 LLM(FP8)** + **Qwen2.5-VL-7B/32B 看屏(常驻,便宜)** + **流式 ASR(Parakeet/SenseVoice)** + **流式 TTS(Faster-IndexTTS-2/FlashTTS)**,VRAM 细分。低并发(1 用户)求最高对话质量,Qwen3-235B-A22B 或 DeepSeek-V3 Q4(~65–70GB)临界但可行(上下文受限)。
- **延迟可达**:LLM TTFT 150–250ms(vLLM 本地无上游);VLM 屏幕理解 ~150–400ms/调用(节流 1–5fps 等效,事件触发);TTS TTFA 40–90ms(Cartesia 级)或 ~325ms(FlashTTS)。**全反应环路 <500ms**(barge-in 本地处理时舒适)。

### Mac(M1 Max 32GB)— 编排 + 端侧感知 + RealityKit 宿主
- MediaPipe/CoreML 面部关键点/blendshape/注视/手身体追踪 30–60fps 几乎零开销。
- Apple Speech ASR 端侧(免费、即时、ZH)作主 ASR 或低延迟首过后再工作站确认。
- ScreenCaptureKit 截自己屏流工作站。
- RealityKit 渲染环 + IK look-at 解算器。
- AVSpeechSynthesizer 神经 TTS 即时本地 fallback。

### Vision Pro
- 企业主摄像头 entitlement 解锁立体帧(`CameraFrameProvider` on `ARKitSession`)。L/R 同步,可用于视差/深度 + 用户感知。
- 头显**原生追踪用户眼/手/脸**—第 2 节感知多半无需跑 MediaPipe。
- RealityKit IK/约束 朝向配置的屏幕锚。

### 带宽 & 拓扑
- Mac→工作站 屏幕流:1080p/4K JPEG/WebP@1–5fps ≈ **8–40Mbps**—有线 GbE/Wi-Fi6 微不足道;LAN 单程 <5ms。
- 工作站→Mac/VP 音频回:流式 PCM/Opus ≈ <0.5Mbps。全反应环留在 **LAN,全隐私**—企业/隐私大优势。

## 6. 推荐架构 + 最难点 + 首读(最新在前)
### 架构
```
VISION PRO(M1 Max Mac 旁编排)
  ARKit CameraFrameProvider(企业)→ 立体帧 → 用户脸/注视/手
   原生 眼/手/脸 追踪(主用户态源)
   Apple Speech ASR(端侧,ZH)─┐
                              ├→ Silero VAD + 端点器(本地 ~50ms)
   ScreenCaptureKit(Mac 自己屏)┤     │ 置信加权 partial
   JPEG/WebP@1–5fps,事件触发   │     │
   └────────────────────────────┘     │
                                      ▼
              ───────── LAN(GbE,<5ms)─────────
        RTX Pro 6000 96GB 工作站(vLLM,常驻)
   对话 LLM        屏幕 VLM          流式 ASR         流式 TTS
   Llama3.3-70B    Qwen2.5-VL-32B   (refine/确认)   Faster-IndexTTS-2
   /Qwen3-30B-A3   (UI-TARS-1.5     SenseVoice(ZH)  /FlashTTS/Qwen3-TTS
   FP8              prompt)                          (ZH 表达)
        │活动 token(每隔几秒/事件)│
        ▼                          ▼
   [融合上下文:对话 + 用户情感 + 屏幕活动]
        └→ 流式 token → 流式 TTS → 音频回 Mac/VP
                 └► BARGE-IN:本地 VAD 在音素边界取消 TTS 缓冲(~200ms),独立于网络
   角色(RealityKit,on Mac/VP)
   默认 look-at IK 指向屏幕锚(idle"看你写代码");轮次让出瞥用户;屏幕事件点头/挑眉;
   backchannel 经 AVSpeechSynthesizer(即时)叠在神经 TTS 下。
```
**关键选择**:① 混合流式而非纯 S2S(保组件独立、全控角色语音、文本通道可调试、barge-in 本地);② 屏幕 VLM 周期+事件触发非每帧(使新颖通道可负担);③ barge-in/VAD 端侧(~200ms 太紧不能冒网络抖动);④ 中文:SenseVoice/FireRedASR→Qwen3-Omni/GLM-4-Voice 或 Qwen3 LLM→IndexTTS-2/CosyVoice2/Spark-TTS;⑤ 感知优先 visionOS 原生 API(眼/手/脸、ARKit),MediaPipe fallback。

### 最难点(排)
1. **屏幕感知环路节奏 + 上下文融合**:何时调 VLM(事件检测:切窗/终端输出突发/构建/空闲)、如何紧凑摘要屏态、如何注入 LLM 上下文不胀不破对话连贯。无现成系统。
2. **跨 Mac+工作站+VP 拓扑 sub-200ms barge-in** + 干净 AEC(角色自己 TTS 别触发假打断—共享声学空间里角色说话近用户时难)。
3. **角色注意分配/IK 融合**跨多 look-at 目标(屏/用户/手)不机械切换;让"看"有生命而非定睛。
4. **mid-utterance TTS 取消无 artifact**(音素边界)—需 TTS 暴露缓冲位置元数据(Faster-IndexTTS-2/Cartesia 有,多数无)。
5. **多语 + 中途 ZH↔EN 切换**(ASR/LLM/TTS 同时)无上下文重置。
6. **企业 entitlement 门控 + 隐私**(主摄像头帧不离 LAN)。
7. **尾延迟 p95 非均值**(400ms 均值负载下飙 1.2s 感觉坏)。每组件按 p95 预算。

### 首读(最新在前)
- FlashTTS(arXiv:2606.09141;github.com/ASLP-lab/FlashTTS);Faster IndexTTS-2(arXiv:2607.21042);Decoupling Conversational Dynamics(arXiv:2607.07148);Full-Duplex SDS(arXiv:2604.21406);VisionClaw 常驻眼镜 agent(arXiv:2604.03486);UAF 统一音频前端 LLM(arXiv:2604.19221);On-device 流式 ASR(arXiv:2604.14493);Qwen3-Omni(arXiv:2509.17765;github.com/QwenLM/Qwen3-Omni);Canary-1B-v2 & Parakeet-TDT(arXiv:2509.14128);Step-Audio 2(arXiv:2507.16632);IndexTTS-2(arXiv:2506.21619;github.com/index-tts/index-tts);EmoNet-Face(NeurIPS2025);GUI-Actor(NeurIPS2025,microsoft/GUI-Actor-7B-Qwen2.5-VL);OS-Atlas(arXiv:2410.23218;github.com/OS-Copilot/OS-Atlas);UI-TARS(arXiv:2501.12326;github.com/bytedance/ui-tars);Moshi(arXiv:2410.00037;github.com/kyutai-labs/moshi);SeeClick(ACL2024;github.com/njucckevin/SeeClick);L2CS-Net(arXiv:2203.03339);GazeOnce(CVPR2022)。

## 来源(关键)
**ASR**:presenc.ai/research/best-open-weight-speech-to-text-models-2026;marktechpost 2026-07;northflank 2026;gladia;artificialanalysis.ai/speech-to-text;HF Open ASR Leaderboard;arXiv 2509.14128;arXiv 2604.14493;funasr.com SenseVoice;github.com/FireRedTeam/FireRedASR
**轮次/barge-in**:vocaiq.ai dualplex-architecture-voice-ai-2026;futureagi 2026;arXiv 2604.21406;arXiv 2607.07148;GPT-Live 2026-07
**情绪/注视/手势/韵律**:MediaPipe face_landmarker;arXiv 2501.13432(BlendFER-Lite);py-feat.org;EmoNet-Face NeurIPS2025;L2CS-Net github;GazeOnce CVPR2022;GAZE2026@CVPR;CHI2026 gaze+speech review;Multi-Scale Temporal Fusion Sensors2025;SER 多模态融合 EAAI2026
**屏幕/GUI agent**:github.com/bytedance/ui-tars;arXiv 2501.12326;OmniParser 微软;arXiv 2410.23218 OS-Atlas;github.com/OS-Copilot/OS-Atlas;SeeClick github;microsoft.github.io/GUI-Actor;arXiv 2604.03486 VisionClaw
**Apple 截屏/摄像头/RealityKit**:ScreenCaptureKit 文档;accessing-the-main-camera;CameraFrameProvider;WWDC24 enterprise APIs;github.com/Waley-Z/visionos-main-camera;vision.engineer;griffinhurt 2025 立体;KhaosT NDI gist;RealityKit 骨骼/IK 文档;devforums 797407 mix Animation+IKRig;WWDC25 287/317
**TTS/S2S**:arXiv 2606.09141 FlashTTS;github.com/ASLP-lab/FlashTTS;arXiv 2607.21042;arXiv 2506.21619;github.com/index-tts/index-tts;github.com/RVC-Boss/GPT-SoVITS;Spark-TTS unsloth;gradium/inworld/deepgram 2026 基准;artificialanalysis TTS/S2S arena;qwen.ai qwen3tts-0115;vLLM-Omni docs;arXiv 2410.00037 Moshi;github.com/THUDM/GLM-4-Voice;HF Qwen3-Omni-30B-A3B-Instruct;arXiv 2507.16632 Step-Audio 2;MichiAI Reddit
**硬件可行**:NVIDIA RTX Pro 6000 Blackwell 页;bizon-tech;wiki.pulsedmedia RTX Pro 6000 模型适配表;petronellatech vLLM 多卡;vllm forum #1707;r/LocalLLaMA 单卡 96GB 讨论

## 未确认
arXiv 2604.14493 流式 ASR 基准具体 WER/延迟数来自搜索引擎摘要(PDF 因 arxiv 网络限制未能直 fetch),具体数字按指示性处理。专有模型 TTFA(Cartesia 40ms、Inworld Realtime TTS-2 领先)为厂商/arena 报告。Apple Speech 端侧流式 WER 未公开基准。
