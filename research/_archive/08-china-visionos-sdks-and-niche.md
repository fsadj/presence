# 国内数字人/虚拟人厂商 for Apple Vision Pro — visionOS SDK + HuggingFace + 利基(2026-07)

> Vision Pro 虚拟角色项目。所有条目已溯源;**[unconfirmed]** = 未直接确认。利基真正空白处会明说。

## A) 中国厂商的原生 iOS/visionOS/Unity/Unreal avatar SDK
### 标题结论
**截至 2026-07,没有任何中国厂商公开发布或宣布 visionOS 原生 avatar/动画 SDK。** 最接近:(1) 会以 iPad/iOS 兼容模式跑在 Vision Pro 上的 iOS SDK;或 (2) 内容级 Vision Pro demo(非 SDK)。**WWDC 2024/25/26 零提及中国 avatar 合作伙伴**;无任何中国厂商的公开 RealityKit/PolySpatial 集成。

### 最清晰的 Vision Pro demo(非 SDK):魔珐科技 Xmov
Xmov 在 app/内容层在 Apple Vision Pro 上演示 3D 虚拟人。中文产业媒体直引:"魔珐科技用苹果 Vision Pro 为我们打开了 3D 虚拟人应用的新场景。在 Vision Pro 中,与虚拟人的面对面交互由屏幕过渡到空间,我们与虚拟人的'隔阂'消失了"。
- 源:https://mp.ofweek.com/ce/a956714267417 , https://t.cj.sina.cn/articles/view/6012711797/16662b375001018852
- Xmov 旗舰虚拟人 agent 名 **"镜 JING"**(2023-05 发布的虚拟人 3.0 智能体):https://column.iresearch.cn/b/202306/962263.shtml
- 开发者 SDK 品牌 **魔珐星云(Xingyun)**——公开文档面向 **Web/大屏/服务端 API**,**无公开 iOS 或 Unity SDK**。https://xingyun3d.csdn.net/column/6941161c20df62166aaba626

### 有 iOS/Unity avatar SDK 的中国厂商(经兼容跑 visionOS,非 visionOS 原生)
| 厂商 | SDK/产品 | 平台(公开文档)| visionOS 原生? |
|---|---|---|---|
| **相芯 FaceUnity** | **AvatarX SDK**(Cubic 引擎+Unity)| iOS,Android,Windows,Mac,Unity | 否 https://www.faceunity.com/avatarxsdk.html |
| **腾讯云** | Tencent Avatar SDK(腾讯云视立方)+ 智能数智人 iVH | iOS,Android,Web | 否 https://cloud.tencent.com/document/product/1662/85737 |
| **阿里云** | **万相 Wanxiang Digital Human**—纯端侧音驱 iOS SDK + 云渲染 iOS SDK | iOS(RTC),Web | 否 https://help.aliyun.com/zh/avatar/avatar-application/developer-reference/digital-people-conversation-ios-sdk-local-avatar-only |
| **火山引擎(字节)** | 虚拟数字人平台 | 2D/3D 生产管线,需商务联系 | 否 https://www.volcengine.com/product/avatar |
| **讯飞 iFlytek** | 虚拟数字人平台 + 超拟人交互 SDK | Web SDK 2.0 公开;iOS 未公开文档 | 否 https://virtual-man.xfyun.cn/ |
| **魔珐 Xmov** | 魔珐星云 SDK | Web,大屏,服务端 API | 无 SDK;有 app 级 Vision Pro demo |
| **PICO(字节 VR)** | PICO Avatar SDK | PICO VR 设备 | 否(封闭 VR 生态)|

### Unity→visionOS 路径(因 FaceUnity AvatarX 用 Unity 而相关)
Unity **PolySpatial** 让 Unity 场景经 RealityKit 跑在 visionOS。这是中国 Unity 系 avatar 技术(如 AvatarX)进 visionOS 最 plausible 的集成路径,但**无中国厂商发布 PolySpatial 移植**。https://unity.com/cn/blog/engine-platform/unity-support-for-visionos
**A 小结**:原生 Swift/RealityKit Vision Pro app,**当前无法授权到成品中国 avatar SDK**。要么 (a) 基于中国厂商云渲染 RTC 管线(阿里万相、腾讯 iVH),(b) 经 Unity PolySpatial 集成 Unity 系 SDK(FaceUnity AvatarX),(c) 向魔珐定制移植。Xmov Vision Pro demo 证明**内容**可达,但 **SDK 未上架**。

---

## B) 中国公司 HuggingFace 组织页

### 腾讯 — https://huggingface.co/tencent(collection hunyuan3d)
- **Hunyuan3D-1 / -2 / -2.1 / -Part**(文/图→3D,PBR,可编辑部件);**Hunyuan3D World Model 1.0**(首个开源 3D 世界模型)。https://huggingface.co/tencent/Hunyuan3D-2.1 ;https://3d-models.hunyuan.tencent.com/

### 阿里(多 org——中国 HF 数字人最富足迹)
**`alibaba-pai`**(124 模型,直接 fetch):**AgenticQwen-8B/30B-A3B**(agentic Qwen);Z-Image-Fun 图像;Wan2.2-Fun 视频;近期论文 **RynnBrain 1.1 具身基础模型**。
阿里 avatar/3D 人在**单独 org**:
- **3DAIGC(阿里 Tanji)**:**LHM-1B-HF**——大尺度可动画人重建,单图→可动画 3DGS avatar 秒级(ICCV 2025)。https://huggingface.co/3DAIGC/LHM-1B-HF , https://github.com/aigc3d/LHM
- **TaoAvatar**(arXiv 2503.17032,CVPR 2025)— 实时、轻量 3DGS 全身 talking avatar,**明确为移动端 AR 设备设计**。https://pixelai-team.github.io/TaoAvatar/
- **Live Avatar**(阿里-夸克,arXiv 2512.04677,ECCV 2026 Oral)— 流式实时音驱 avatar,与 **Qwen3-Omni** 集成做交互对话 agent。https://github.com/Alibaba-Quark/LiveAvatar
- **ModelScope**—阿里自家模型 hub(平行 HF),有开源实时数字人对话 demo。https://www.modelscope.cn/

### 影眸/Deemos(Hyper3D)— https://huggingface.co/Deemos
"models: None public yet"——**不在 HF 发权重**。经 Spaces(Rodin)与 RodinHD 论文发布。产品 hyper3d.ai:Rodin Gen-1(1.5B DiT,PBR,四边拓扑)、Gen-2(~10B,2025-10)、Gen-2.5。

### 网易伏羲 — https://huggingface.co/FUXI
7 模型:Multi-modal_10B_CN、yuyan-11b/10b/dialogue、danqing-caption。数字人研究多私有;公开含 **FaceG2E**(文→3D 脸)。

### 商汤 — https://huggingface.co/SenseTime
仅 5 模型,全是 **Deformable-DETR**(检测)。**HF 上零数字人/avatar 模型**;SenseAvatar/如影 专有(仅产品页)。发过《AI 数字人白皮书》定义 5 阶段。

### 字节(两个 org)
**`ByteDance`**(54 模型):**UniVR-34B-Planning**(VR 规划模型,图+文→文);Bernini-R(14B 图文生视频);Ouro-1.4B/2.6B;数据集 **VR-X-SFT-RL**(23.8 万行 VR 相关)、veAgentBench。
**`ByteDance-Seed`**:**UI-TARS-1.5-7B**(GUI agent);**Seed3D 1.0**(图→高保真仿真级 3D 资产,arXiv 2510.19944)。

---

## C) "AR + 具身AI agent + 虚拟人" — 利基(最高价值)
严格交集(AR 渲染 + 具身/多模态 AI + 虚拟角色,来自中国)**干净命中三者的项目极少**。最接近,按相关度降序:

### Tier 1 — 直接命中
1. **阿里 TaoAvatar(CVPR 2025)**— 最相关的中国项目。实时、轻量、**3DGS 全身 talking avatar,明确为移动端 AR 设备构建**。多模态驱动(音频/姿态/手势/表情),拓扑一致可 rig/动画。**最接近"AR+虚拟角色"的中国开源项目**。https://pixelai-team.github.io/TaoAvatar/ , arXiv 2503.17032
2. **魔珐 Xmov — Vision Pro demo + "镜 JING"**。已确认的中国 app Vision Pro + 3D 虚拟人 demo。JING 定位"虚拟人 3.0 智能体",多模态交互。**最强中国 Vision Pro + 具身虚拟角色 demo 证据**。非开源。
3. **阿里 Live Avatar + Qwen3-Omni(ECCV 2026 Oral)**。流式实时音驱 avatar,作者明确与 Qwen3-Omni 结合做"完全交互对话 agent"。**最接近"具身多模态AI+虚拟角色"的开源栈**。https://github.com/Alibaba-Quark/LiveAvatar , arXiv 2512.04677

### Tier 2 — 旁系(缺一根柱)
4. **Rokid(AR 眼镜 + 数字人技术栈)**。核心列"数字人技术":30+ 骨骼/动作节点、多角色自适应、ASR/TTS/AIGC,定位 NPC + 办公对话。AR Studio 含 SLAM+3D 手势+6DoF+空间音频+数字人。**确是"AR+虚拟角色",但不在 Vision Pro**——是 Rokid 自家 AR 眼镜平台。https://www.rokid.com/zh-CN/technology 。注:Rokid 现售 AI 眼镜(2026 Style)是语音+摄像头智能眼镜(ChatGPT-5/Gemini),**出货产品上无 3D 虚拟角色陪伴**,只有语音助手。
5. **XREAL**。**未发现原生虚拟陪伴角色**。出货空间显示器;AI 助手依赖外设。[unconfirmed——大概率不存在为出货功能]。
6. **百度小度 AI Glasses Pro + "超能小度"(2025-11)**。多模态助手(视觉+语音),但是 Meta-Ray-Ban 式语音/摄像头助手——**无具身 3D 虚拟角色**确认。
7. **百度"数字人家族"(度晓晓/希加加/林开开/叶悠悠)**。多模态 AI 数字偶像,但**2D/手机屏/元宇宙(希壤)**——**非 AR 具身**。

### 值得知道的中国具身 agent 研究(非 AR+虚拟人)
8. **OS-Copilot / FRIDAY**—通用 OS 级 agent 开源框架。作者来自上海AI Lab+华东师大+Princeton+港大;一作吴志勇现**在字节 Seed**。**GUI/电脑 agent,非 AR/虚拟人**。"具身"指软件具身,非物理/AR 具身。https://os-copilot.github.io/
9. **ShowUI(CVPR 2025,showlab)**—4.2B VLA 用于 GUI 自动化。非 AR、非虚拟人。https://github.com/showlab/ShowUI
10. **清华具身AI 实验室**。EIR(2025 末)+ IIIS VAR + MARS Lab + AIR + KEG/智谱 CogAgent GUI agent。**无一公开结合 AR 渲染 + 虚拟人角色**——其具身是机器人/VLA。

### 利基判定(C)
- 严格"AR 渲染 + 具身/多模态AI + 虚拟角色,来自中国,在 Vision Pro 上"利基**在产品级基本未被占据**。唯一确认的中国 Vision Pro + 虚拟角色 demo 是**魔珐 Xmov**,且是内容 demo,非产品化 SDK 或开源项目。
- **最强开源中国栈**(映射你目标):**阿里 TaoAvatar(3DGS 身体)+ Live Avatar(实时流式)+ Qwen3-Omni(多模态大脑)**——组合即具身 Vision Pro 角色的组件,但集成需自己做。
- **最强商业中国伙伴**(要原生中国 avatar 技术):**魔珐 Xmov**(实测 Vision Pro demo、"镜 JING")或**相芯 FaceUnity**(Unity 系 AvatarX SDK,可经 Unity PolySpatial 上 visionOS)。
- **Rokid** 是唯一清晰出货"AR 眼镜 + 集成数字人技术"的中国公司——但在自家硬件,非 Apple。
- **未发现任何 WWDC 2024/25/26 提及中国 avatar/数字人合作伙伴**。[unconfirmed——可能存在但搜索不可发现]

### 两条标 **unconfirmed** 的线索(搜索摘要推断,未能对一手源核)
- 搜索摘要称"商汤 2025 Tech Day 演示 SenseAvatar 跑在 Apple Vision Pro"。fetch 一手源(商汤 Facebook Tech Day 2025 帖)后**只宣布 SenseNova V6 和 SenseCore 2.0——无 Vision Pro 提及**。该主张按**未核验/疑似搜索摘要幻觉**处理。

---

## 给开发者的下一步建议
1. **直接邮件魔珐与相芯商务团队**问 visionOS 原生或 Unity-PolySpatial 版本——两家都未公开文档,但都最接近出货。
2. **用开源阿里栈原型**:LHM(单图→3D 可动画 avatar)+ TaoAvatar(AR 目标实时渲染器)+ Live Avatar(Qwen3-Omni 对话环路)。三者皆阿里/Tanji/夸克且开放可得。
3. **勿对 OS-Copilot/ShowUI 过度投入**做此利基——优秀的中国 GUI agent 项目,但不解决 AR 或虚拟角色。
4. **盯字节 UniVR-34B-Planning + VR-X-SFT-RL**——字节公开发 VR/XR 规划模型,暗示内部 VR-agent 努力,可能浮现为相关工具。
