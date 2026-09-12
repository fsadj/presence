# 竞品 / 相关工作定位(2026-08)

> 最相似现存系统逐个对比:相似处 / 你有·它没有(差异化)/ 它强·你要追。给立项、对外讲故事(高校/投资人)用。已并入 open-vla 发现的 SOLAMI/ViBES 等最新先例。

## 最相似系统对比

| 系统 | 与你相似 | ✅ 你有·它没有(差异化) | ⬆ 它强·你要追 |
|---|---|---|---|
| **阿里 MNN-TaoAvatar**〔china-industry / open-vla〕| **最接近**:3DGS 全身 talking avatar + Qwen LLM + ASR/TTS/A2BS,端侧,**Vision Pro 90fps demo** | 感知**真实房间**(mesh+企业摄像头)非只渲染;**持久真实房间记忆**(物体恒常、坐*你的*躺椅);**屏幕感知**;多时间尺度架构;**原生 Swift/RealityKit**(它是 3DGS/MNN、Android 打包);跨会话陪伴人格;**VLA 锚定动作**(open-vla) | 3DGS 写实渲染质量;**已跑通 Vision Pro demo**;管线已集成 |
| **SOLAMI(CVPR2025)**〔open-vla〕| **首个端到端社交 VLA,VR 里 3D 自主角色**,语音+肢体驱动——**几乎就是你的产品,你的模板** | **真实房间 AR grounding**(它是 VR 合成);跨会话持久记忆;贴真实家具 affordance;屏幕感知;原生 Swift | 已发表顶会概念验证 + SynMSI 合成数据;你尚在立项 |
| **ViBES(CVPR2026)**〔open-vla〕| 对话 agent + 行为智能 3D 虚拟身体,LLM 吐语义 token 驱动表达(共话语手势/talking-head/text-to-motion) | 真实房间感知;持久记忆;affordance;原生 AR 陪伴 | 已验证"重定义 token 为行为词表"范式(open-vla 路线) |
| **DeepMind SIMA/SIMA2**〔open-vla〕| VLA 但动作=**虚拟世界键鼠**(非机器人)——"VLA 给虚拟身体"的概念许可 | **真实房间 grounding**;AR 空间陪伴;持久记忆;对话人格;真实家具 affordance | 跨大量 3D 游戏泛化、Gemini 级推理广度(闭源、游戏域) |
| **Meta Horizon 具身 LLM NPC**〔open-vla, user-perception-dialogue〕| LLM 驱动 NPC,会对话+比划+表情 | **真实房间**(AR)非虚拟世界;感知**真实用户**(注视/情绪/屏幕);真实家具交互;住进*你的*空间 | 产品级规模部署、社交多用户(域是虚拟社交) |
| **NVIDIA ACE(Audio2Face-3D+Riva)**〔animation-affordance〕| "AI 脸+库身体"标杆管线 | AR 空间+真实房间感知+记忆+affordance+屏幕感知+陪伴人格;ACE 只是脸/语音管线 | **脸/语音业界最强**(你本就把 Audio2Face-3D 当组件) |
| **魔珐 Xmov(星云/镜JING)**〔china-industry-and-niche〕| "具身AI 3D 数字人",文→语音+表情+手势+身体,**有 Vision Pro demo** | **真实房间感知**(非屏/全息柜);持久记忆;affordance;原生 AR 陪伴 | 成熟数字人平台、SDK/API、3000+ avatar |
| **OK-Robot / 3D-Mem / HOV-SG**〔scene-memory〕| **开放词表持久 3D 场景记忆+物体恒常**——正是你的场景图设计 | 它是**机器人**(物理),你是虚拟角色;你加 AR 渲染+陪伴人格+affordance 姿态(坐/躺)+屏幕感知 | **真机验证的场景记忆栈**(你的记忆设计借它) |
| **Generative Agents(Stanford Smallville)**〔open-vla〕| LLM agent 情景+语义记忆/反思/规划/人格——你的认知/记忆原则同源 | **真实房间空间 grounding**(它是 2D 文本世界);AR 具身;真实感知;affordance | 记忆/反思/人格的**认知架构**(你借其思想) |
| **世界模型线(Dreamer/V-JEPA2/Cosmos/UniVR-34B)**〔scene-memory, world-model-verdict〕| 预测式规划、统一感知+推理——你的"未来大脑/ForesightService"候选 | **跨会话持久记忆**(它们全无);真实房间 grounding;陪伴人格/对话;**多时间尺度**(它们单时间尺度) | **预判/前瞻能力**(你窄采用为 ForesightService) |
| **物理控制器线(PHC/ExBody2/OmniH2O/ASE)**〔animation-affordance〕| 全身物理控制、抗扰动——"精确操控"的另一条路(animation-affordance) | 你锚定 VLA(非物理仿真桥);AR 真实房间;对话人格 | 物理可信度/接地(若你要精确物理控制,这是开源底座) |

## 看似像但不同类(对照,非竞品)
- **屏基数字人**(商汤如影/腾讯 IVH/硅基 Duix〔china-industry-and-niche〕):2D/屏/全息,非 AR 空间;不感知真实房间。
- **Apple Spatial Personas**:visionOS 原生 AR avatar,但是**用户自己的替身**,非 AI 驱动独立陪伴。
- **姿态生成研究**(HOSIG/PhySIC/坐姿生成〔animation-affordance〕):只解"贴家具姿态"一片,非完整角色。
- **GR2/Gemini Robotics**(open-vla):闭源、物理机器人 VLA——北极星,非可用组件。

## 裁决:你的差异化与风险

**没有任一现存系统同时具备:AR 空间 + 真实房间 grounding + 持久记忆 + 贴真实家具 affordance + 屏幕感知 + 陪伴人格 + 原生 Swift + VLA 锚定动作。** 各块分别存在(TaoAvatar=avatar+脑管线;SOLAMI/ViBES=社交 VLA 驱动虚拟角色;OK-Robot/3D-Mem=场景记忆;SIMA=虚拟具身 VLA;ACE=脸;世界模型=预判),**没人组合成"真实房间 AR 陪伴角色 + VLA 动作"**。

- **你的差异化(护城河)**:真实房间 grounding + 持久物体恒常 + 贴真实家具坐/躺/趴 + 屏幕感知 + 多时间尺度活感架构 + VLA 锚定的统一感知→动作。**组合创新,非单点。**
- **你的风险**:正因是组合,集成难;每块对标的最强者都更成熟(渲染→TaoAvatar、脸→ACE、记忆→OK-Robot、预判→世界模型、VLA 精确→GR2/π0[闭源/机器人形状])。**你的赢法不是某块超越,而是"组合 + 真实房间 VLA 陪伴"这个没人占的 niche。**
- **最该学的模板**:**SOLAMI**(CVPR2025)和 **MNN-TaoAvatar**——前者是范式先例(社交 VLA→虚拟角色),后者是工程先例(已上 Vision Pro)。

## 一句话定位(对外讲)
> "我们做的是**首个'真实房间 AR 具身陪伴角色'**——用 VLA 把感知-认知-动作闭环放进一个虚拟身体,在 Vision Pro 上感知你的真实房间和你、记住跨会话、坐在你的真家具上、看着你写代码。最接近的是 SOLAMI(VR 合成)和 MNN-TaoAvatar(无真实房间感知);我们的增量是**真实房间 grounding + 持久记忆 + affordance + 屏幕感知 + 原生 Swift**,组合上没人做过。"

## 关联文件
差异化细节见各通道:VLA〔open-vla〕、场景记忆〔scene-memory〕、动画/affordance〔animation-affordance〕、屏幕感知/对话〔user-perception-dialogue〕、世界模型/预判〔world-model-verdict〕、契约〔contracts-v1〕、架构/可进化〔architecture-log〕。
