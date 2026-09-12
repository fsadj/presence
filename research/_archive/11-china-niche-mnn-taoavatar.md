# 国内利基升级版 · MNN-TaoAvatar 等(2026-07)

> 与 08(visionOS SDK+利基)互补。本篇升级关键发现:**MNN-TaoAvatar 是唯一干净命中"AR + 端侧多模态AI + 虚拟角色"的中国开源项目**,且已在 Apple Vision Pro 演示。

## TL;DR
1. **截至 2026-07,无任何中国 avatar 厂商发布 visionOS 原生 SDK**(相芯/商汤/魔珐/腾讯/阿里/百度皆 iOS/Android/Unity,**无一 visionOS/xrOS/RealityKit**)。真实可利用缺口。
2. **你精确利基的唯一最佳契合:阿里 MNN-TaoAvatar**—开源、端侧、多模态 3D talking avatar(3DGS+ASR+LLM+TTS+A2BS+神经渲染),论文标题即 "...for Augmented Reality",**已在 Apple Vision Pro 演示**。
3. **最接近的商业"具身AI+数字人"平台:魔珐星云(Xmov)**—"全球首个具身智能 3D 数字人开放平台",SDK/API,云-边端渲染架构(百元级芯片可跑),但未 AR/visionOS 化。
4. HuggingFace 可用开源资产:**腾讯(Hunyuan3D-*、HunyuanVideo-Avatar)、美团(LongCat-Video-Avatar)、字节(LatentSync)**。影眸 Rodin、商汤如影 **仅商业、无开源权重**。

## A) 中国厂商 avatar SDK 平台(均无 visionOS)
| 厂商 | 产品 | 文档 SDK 平台 | visionOS? |
|---|---|---|---|
| 相芯 FaceUnity | AvatarX/AR 特效 | Android,iOS,Unity,Web,Win/Mac,鸿蒙 | 否 |
| 商汤 | SenseME/SenseAR Effects | Android,iOS | 否 |
| 魔珐 Xmov | 魔珐星云 SDK | Android,iOS,鸿蒙(边端渲染)| 否 |
| 腾讯云 | 智能数智人客户端渲染 SDK | Android,iOS | 否 |
| 阿里云 | 虚拟数字人 SDK(云渲染 RTC)| iOS,Android | 否 |
| 百度 | 3D 数字人交互 SDK | Android(iOS 可能)| 否 |
| 火山引擎(字节)| 虚拟数字人 | 云管线(2D/3D)| 否 |
| 讯飞 | 虚拟数字人平台 | SaaS/API | 否 |

**中国 avatar 技术在 Vision Pro 上的公开演示:仅一个——阿里 MNN-TaoAvatar**(C 节)。**WWDC 2024/25/26 无中国 avatar/数字人 SDK 合作伙伴提及**(中国 app 上 Vision Pro 的有微信/钉钉/携程/淘宝/高德/招行;visionOS 26/27 加了 Foveated Streaming 云渲染)。
**含义**:visionOS 机会开放。阻力最小路径:(a) 移植云渲染管线(腾讯/阿里/百度经 RTC 拉视频流—visionOS 今天经 AVKit 可用),或 (b) 用 RealityKit + 开源模型管线(如 MNN-TaoAvatar)做原生端侧渲染。

## B) HuggingFace 中国公司(数字人/3D/avatar 模型)
| Org | URL | 相关公开模型 | 注 |
|---|---|---|---|
| **腾讯** | huggingface.co/tencent | Hunyuan3D-1/2/2.1/Part(文图→3D mesh)、**HunyuanVideo-Avatar**(音→avatar 视频,2025-05)、HunyuanCustom | 最强开源 3D+avatar 阵容。GitHub github.com/Tencent-Hunyuan |
| **影眸 Deemos** | huggingface.co/Deemos | "None public yet"。RodinHD 仅论文 | Rodin/Hyper3D **商业 API,无开源权重** |
| **阿里 alibaba-pai** | huggingface.co/alibaba-pai | ~124 模型;EMO 仅论文(GitHub 仅论文链接,无实现);Wan2.1-Fun-* 视频 | EMO 权重**未发布**。阿里真·开源 avatar 资产是 **MNN-TaoAvatar**,权重在 **ModelScope**(非 HF) |
| **字节** | huggingface.co/ByteDance | **LatentSync**(音驱 lip-sync,开源 U-Net+SyncNet+Whisper ckpt);OmniHuman-1/1.5 研究/API 无开源;GauHuman CVPR 2024 论文 | LatentSync 是可用开源资产 |
| **商汤** | huggingface.co/SenseTime | SenseNova U1 Lite 系(多模态 LLM,2025 末开源)、检测模型 | 如影 SenseAvatar **不开源**(商业 SaaS)|
| **美团** | huggingface.co/meituan-longcat | **LongCat-Video-Avatar/-1.5**(音驱 avatar 视频;AT2V/ATI2V;商业级;基于 13.6B LongCat-Video)| 近期发布的强开源 avatar 模型。GitHub github.com/meituan-longcat/LongCat-Video |
| **网易伏羲** | 无突出 HF avatar org | 研究:Make-A-Character(文→3D lifelike avatar,网易+密歇根)、FreeAvatar(arXiv 2409.13180)| 论文,无可下载 HF 权重 |

**其他**:`weihaox/awesome-digital-human`(github.com/weihaox/awesome-digital-human)是上述最佳策展索引。

## C) "AR + 具身AI + 虚拟人"中国利基
诚实答案:**干净命中三者之一的项目极少**。中国"具身AI"偏物理机器人(人形);"数字人"多屏基(直播/客服)。**唯一清晰融合 AR 渲染 + 端侧多模态AI + 虚拟角色**:

### Tier 1 — 直接命中
**1. 阿里 MNN-TaoAvatar(淘宝 Meta 团队)— THE 答案。**
- 全身拟真 3D talking avatar,基于 **3D Gaussian Splatting**,完整**端侧**多模态管线:MNN-ASR + MNN-LLM(Qwen2.5-1.5B)+ MNN-TTS + A2BS(音→blendshape)+ MNN-NNR(神经渲染)。骁龙 8 Gen 3 上 60 FPS。
- 论文:**"TaoAvatar: Real-Time Lifelike Full-Body Talking Avatars for Augmented Reality via 3D Gaussian Splatting"**(arXiv 2503.17032)。项目 https://pixelai-team.github.io/TaoAvatar/
- 发布文明确称"能在手机或 XR 设备上实现 3D 数字人的实时渲染以及 AI 对话",并展示"在 Android 手机及 **Apple Vision Pro 设备**上的体验效果"(https://m.thepaper.cn/newsDetail_forward_31039750)。
- 经阿里 MNN 框架开源:https://github.com/alibaba/MNN(apps/Android/MnnTaoAvatar;README /blob/master/apps/Android/MnnTaoAvatar/README_CN.md)。模型在 ModelScope。
- **对你意义**:MNN 跨平台 C++(**iOS/visionOS 受支持**),这是原生 visionOS 具身角色最可移植起点。开源发布是 Android 打包,需自己把管线移植进 RealityKit/visionOS。

**2. 魔珐 Xmov / 魔珐星云(xingyun3d)**—具身智能 3D 数字人开放平台。
- 自称(量子位 2025-10-30)"**全球首个具身智能 3D 数字人开放平台**"—给 LLM/agent 一个身体:实时 文本→3D avatar 言语/表情/注视/手势/身体动作。
- 云-边端分离:云端生成语音+动作*参数*,边端 AI 渲染—**端到端 <1.5s,千万级并发,百元级芯片(RK3566/3588)可跑**,信创芯片。
- 经 **SDK 或 API** 接入;面向屏(移动/平板/PC/TV/大屏/全息舱)、人形机器人、"任意终端"。
- 源:https://www.qbitai.com/2025/10/347284.html ;https://xingyun3d.com ;创始人柴金祥。
- **caveat**:屏基"虚拟具身"(其术语),非 AR/空间。无 visionOS 移植。但边端渲染架构概念上契合耦合/系留头显。

### Tier 2 — 旁系
**3. 百度度晓晓/Xiaodu**:3D 数字人 AI 陪伴(多模态、情感/陪伴),手机 app。**非 AR/空间**。另有 DuMix AR 但未与数字人产品化为一体。
**4. 蚂蚁灵波 LingBot**:严肃具身AI"机器人脑"栈(LingBot-VA 2.0 世界动作模型、VLA、Vision/Depth 空间感知)。重机器人,非虚拟人。
**5. 高校(清华/浙大/上交/智源)**:强具身AI/机器人研究(清华 EIR、AIR+阿里云"可进化 agent"、ZJU 余姚具身、智源)。**但未发现具体开源高校项目结合 AR 渲染+具身AI+虚拟人角色**—中国学术具身工作主要是物理机器人。最清晰 AR+AI+avatar 工作是工业界 MNN-TaoAvatar。

### 明确否定发现
- **Rokid 与 XREAL 不出货虚拟陪伴角色**。Rokid Glasses=语音 AI 助手(翻译/提词/导航/AI识物)+ AR 文字/信息叠加—无 3D avatar。XREAL=空间显示器(大屏视频),偏影音。两者均无具身虚拟人。
- **ShowUI/OS-Copilot 非此利基**。ShowUI 是 **ShowLab(邵岭 Mike Zheng Shou,NUS—新加坡,非大陆)** 的 GUI VLA agent,自动化屏幕 UI,无 AR 渲染、无虚拟角色。OS-Copilot 同。
- **B 站"Vision Pro 数字人"视频**(如 BV18H1TBnEUR)讲的是 **Apple 原生 Persona**,非中国 avatar 技术。
- **网易伏羲、商汤、字节**发 avatar *研究*(Make-A-Character、FreeAvatar、OmniHuman)但无一打包成 AR+具身 agent+虚拟人产品。

## 给 visionOS 开发者的伙伴/技术短名单
1. **阿里 MNN-TaoAvatar**—最佳开源基座,适配进原生 visionOS 角色(跨平台 MNN、3DGS 渲染、完整端侧多模态AI、已 AVP 演示)。
2. **魔珐 Xmov(魔珐星云)**—最佳商业"具身AI 数字人"平台,要交钥匙云-边端 SDK 可合作(有开发者计划,明邀 SDK/API 集成)。
3. **腾讯 Hunyuan3D-2 + HunyuanVideo-Avatar**(开源)—生成 3D avatar 资产 + 视频 avatar 退路。
4. **相芯 FaceUnity**—成熟中国 avatar/AR 特效 SDK 厂商,移动/Unity 覆盖最广;最自然去*请求* visionOS 移植的伙伴(已支持 iOS+Unity,RealityKit/visionOS 桥接可行)。
5. **美团 LongCat-Video-Avatar / 字节 LatentSync**—自建管线时的音驱 talking-face 开源层。

**无一今日有出货 visionOS SDK**—印证合作机会真实且时间敏感。

## 关键源
- FaceUnity 开发者中心 https://www.faceunity.com/developer/
- 商汤 SenseME https://www.sensetime.com/cn/product-business?categoryId=79
- 魔珐星云 https://xingyun3d.com | 量子位发布 https://www.qbitai.com/2025/10/347284.html
- 腾讯数智人 SDK https://cloud.tencent.com/document/product/1240/118295
- 阿里 MNN-TaoAvatar https://github.com/alibaba/MNN | 论文 https://arxiv.org/abs/2503.17032 | 发布文 https://m.thepaper.cn/newsDetail_forward_31039750 | ModelScope https://modelscope.cn/collections/TaoAvatar-68d8a46f2e554a
- 腾讯 HF https://huggingface.co/tencent | HunyuanVideo-Avatar https://huggingface.co/tencent/HunyuanVideo-Avatar
- 美团 HF https://huggingface.co/meituan-longcat/LongCat-Video-Avatar-1.5
- 字节 HF https://huggingface.co/ByteDance/LatentSync | OmniHuman https://omnihuman-lab.github.io/v1_5/
- 影眸 https://huggingface.co/Deemos(无公开模型)| Hyper3D https://hyper3d.ai
- 商汤 HF https://huggingface.co/SenseTime
- awesome-digital-human https://github.com/weihaox/awesome-digital-human
