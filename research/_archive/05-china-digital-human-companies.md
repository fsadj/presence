# 国内数字人 / 虚拟人公司 — Vision Pro 合作分析(2026-07/08)

> 目标用户:Apple Vision Pro 开发者(原生 Swift,具身AI认知 + AR 渲染)。4 家深度核验。

## 按 Vision Pro 契合度排名
| 排名 | 公司 | 对 Vision Pro 最佳资产 | iOS SDK | Unity/Unreal | 关键风险 |
|------|------|------------------------|---------|--------------|----------|
| 1 | **硅基智能 Guiji(Duix)** | Duix-Mobile 开源端侧 avatar SDK | **是**(原生)| 否(移动原生)| 2D 视频 avatar,非 3D mesh |
| 2 | **魔珐科技 Xmov(星云 Xingyun)** | 3D 具身AI avatar,SSML 动作控制 | 声称 | 声称(Unity+Unreal)| 云渲染串流模型;SDK alpha |
| 3 | **商汤 SenseTime** | SenseMARS 面部追踪/AR(补充 ARKit)| 是(SenseAR/SenseME)| 是(SenseAR Unity SDK)| 无 avatar 动画 SDK 本身 |
| 4 | **世优科技 Shiyoo** | 动捕硬件 + BVH/FBX 数据管线 | 否 | 经 Puppeteer 管线 | 无公开 SDK;硬件/服务商 |

**关键发现:4 家都没有公开的 visionOS SDK 或已宣布的 Apple Vision Pro 合作。所有 visionOS 路径都需自定义集成。**

---

## 1) 商汤科技 SenseTime
- 中文:商汤科技有限公司。2014-10 成立于上海。**港股上市(0020.HK)**,2021-12-30。仍运营。~2,472 人。已故汤晓鸥(创始人)、现 CEO 徐立。
- **签名产品**:如影 **SenseAvatar**(SaaS,数字人视频生成,**非 SDK**);**SenseMARS 火星混合现实平台**(最接近——含虚拟化身、特效引擎、三维重建);**SenseMARS 特效引擎 SDK**(美颜/贴纸/肢体特效,基于人脸关键点/手势/背景分割);**SenseME**(移动终端 SDK);**SenseAR**(AR SDK,**有 Unity SDK** 下载 openar.sensetime.com/sdks);SenseNova/秒画/琼宇/格物。
- **给 Vision Pro 带来什么**:面部追踪、AR 特效、手势、SLAM——**补充而非替代** ARKit/RealityKit。SenseAvatar 是视频输出 SaaS,无实时渲染 SDK。**无专门 3D avatar 动画 SDK**。SenseSpace/SenseThings 是场景/物体 3D 重建,非角色生成。
- **SDK/授权/visionOS**:iOS SDK 是(商汤 SDK,经声网插件也用);Unity SDK 是;Unreal 未确认;**visionOS 无**。商用 SDK,per-app 授权,business@sensetime.com。
- **HuggingFace**:https://huggingface.co/SenseTime — 5 个模型全是 Deformable-DETR(目标检测),**无 avatar/数字人/3D 模型**。SenseNova 在单独 collection。
- **结论**:契合度有限。CV/AR SDK 与 ARKit 重叠;无实时 3D avatar 动画 SDK。可作 ARKit 补充或认知层 LLM,非主 avatar 渲染伙伴。

## 2) 硅基智能 Guiji AI(Duix)
- 中文:硅基智能(南京),2017 成立,司马华鹏。**已递表港 IPO(2025-11,2026 中重新递表)**,"数字人第一股",估值 31.5 亿元(2025-06),腾讯 16.59% 为最大机构股东。**勿与硅基流动 SiliconFlow 混淆**(不同公司)。
- **签名产品**:**Duix-Mobile**(开源移动 SDK,实时交互 AI avatar,**iOS+Android** 原生,<120ms,带 barge-in;https://github.com/duixcom/Duix-Mobile);**Duix-Avatar**(~10 秒视频克隆+lip-sync);**Duix.HeyGem**(talking-head 视频生成);**Duix-Reface**(实时换脸);DUIX 云 SaaS/API;DUIX.ONE 多模态大模型。GitHub 组织 https://github.com/duixcom 。
- **给 Vision Pro 带来什么**:**4 家中端侧 avatar SDK 最强**,唯一有公开文档的原生 iOS SDK 且**完全端侧**(无云依赖)——契合 Vision Pro 本地/低延迟偏好。设计上**接入你自己的 LLM/ASR/TTS**,契合"你出大脑、Guiji 出脸"的具身认知架构。
- **SDK/授权/visionOS**:iOS **明确支持**(README 列 iOS/Android/平板/车机/**VR**/IoT/大屏);Unity/Unreal 未提及(移动原生);**visionOS 无**,但 iOS 代码库是 4 家中最自然的 visionOS 移植起点。"开源 SDK"但**确切 license README 未显示(unconfirmed)**;定制 avatar 需邮件 support@duix.com(暗示生产用商用双授权)。**用前核 LICENSE**。
- **HuggingFace**:无 org 页(在 GitHub/Gitee)。
- **技术注意**:Duix-Mobile 渲染的是**2D 拟真 talking-head(视频神经渲染),非可控行架的 3D mesh**。适合"对话窗口脸",不能给 Vision Pro 空间环境里一个全关节 3D 身体。延迟数据基于骁龙 8 Gen 2,**Apple Silicon(M2)是否开箱支持 unconfirmed**。
- **结论**:**端侧 avatar 交互 SDK 成熟度最高,但是 2D 视频、非 3D 空间**。若 Vision Pro 角色是"说话的脸"而非全身空间 avatar,契合;iOS SDK 是全清单中最具体起点。

## 3) 世优科技 Shiyoo(4utech.com)
- 中文:世优(北京)科技股份有限公司,2015-03-18(公司实体),纪智辉(brief 的"纪铮"未确认;**CUAV 是另一家无人机飞控公司 cuav.net,疑为混淆**)。国家级专精特新小巨人,披露融资 ~2 亿元,B 轮 >1 亿(2023-12)。
- **签名产品**:**Puppeteer 虚拟工场**(2012 起核心平台,动捕+面捕+实时渲染引擎,Unity/Unreal 管线);**世优波塔 BOTA**(AI 数字人 agent,3D/2.5D,面向数字大屏/全息柜/网页);**波塔 Web SDK**(**唯一公开 SDK**,15 分钟集成);**UCM-2 Pro 动捕服/UCG-2 Pro 手套/UCF-2 Pro 头盔**(UCFace 2.0,32 参数,<25ms);**UME 光学动捕**(成都棚);AI 直播系统。已建 2000+ 数字人。
- **给 Vision Pro 带来什么**:**4 家中唯一深度动捕硬件+生产管线**。若需高质量 mocap 数据(BVH/FBX)驱动 avatar,UCM/UCG/UCF + Puppeteer 能产。生产服务导向,非开发者 SDK 导向。
- **SDK/授权/visionOS**:公开 SDK 仅波塔 Web SDK(JS,大屏);**无 iOS/Android/Unity/Unreal 公开 SDK**(动捕数据经 Puppeteer 进 Unity/Unreal 作生产管线输出 BVH/FBX,非运行时 SDK);visionOS 无。模式:**定制方案交付 + SaaS**。
- **结论**:**非 SDK 伙伴**。价值在动捕硬件 + 内容生产服务。若要(a)买动捕服给 Vision Pro 角色动画,或(b)外包数字人资产,联系他们。波塔 Web SDK 面向柜台/大屏,非空间计算。

## 4) 魔珐科技 Xmov(星云 Xingyun)
- 中文:魔珐(上海)信息科技,2018 成立,柴金祥,上海徐汇。披露融资 ~1.3 亿美元(2022-04 B+C),早期投资人红杉中国/晨兴/沈向洋。1000+ 企业客户。
- **签名产品**:**魔珐星云 Xingyun**(2025-10-29 发布,"全球首个具身智能 3D 数字人开放平台",理念"语言驱动身体"——LLM 输出动作参数驱动 3D 人或机器人,非预渲染视频;xingyun3d.com);**魔珐有言 Youyan**(文生 3D 视频);有光/有灵;虚拟人 IP "Ada/Ava"(中国首例虚拟人法律纠纷中心);**Xingyun litesdk(xmovAvatar.js)** Web SDK(`speak(ssml)`/`speakWithAction`/`think`,动作语义 Hello/Agree/Think... 经 `<ue4event>` SSML 标签)。
- **技术能力**:52 面部 blendshape、全身骨骼驱动、呼吸 idle;**<500ms 端到端**(ASR 流式+LLM 流式+<100ms TTS+<50ms avatar 驱动);barge-in;3000+ 超写实 3D avatar 库、图生 3D;千万级并发;信创私有化(飞腾/鲲鹏、麒麟/统信、等保三级+商密)。
- **给 Vision Pro 带来什么**:**4 家中最完整的 3D avatar 渲染+动画故事**:文本→语音+表情+手势+身体动作实时。SSML 动作触发引擎无关,优雅驱动任何 rig。图生 3D 可供角色资产。
- **SDK/授权/visionOS**:Web SDK **确认**(CDN JS,多 Vue3 demo);**iOS/Android/Unity/Unreal 声称但官方文档未能直接核验**(社区教程称"统一 API 支持 Web/iOS/Android/Unity/Unreal",官方称"多终端 一次开发到处运行",但**唯一直接可检的是 Web JS SDK**;原生/Unity/Unreal 下载链接未定位——**按"声称,待与魔珐核"处理**)。渲染架构**本质云串流**(连 gateway 下载资源 Canvas/视频流渲染);社区称"端侧渲染 SDK/免显卡/百元芯片"但 Apple 平台端侧 unconfirmed。付费(per-call);visionOS 无。
- **结论**:**4 家中 3D avatar 技术最佳,但集成风险高**。若声称的 Unity SDK 真存在,路径:星云 Unity SDK → Unity visionOS(PolySpatial)→ Vision Pro。**承诺前向魔珐核实 Unity SDK**。云串流默认架构对空间计算延迟不利,追问 Apple Silicon 的"端侧渲染"选项。

---

## 横切发现
### 没有人有的
1. **4 家都没有 visionOS SDK**。每条路径都需定制。
2. **没有任何中国虚拟人厂商宣布 Apple Vision Pro 合作**(中英双语激进搜索)。
3. **没有原生 Swift SDK**——iOS 支持全是 Objective-C 或桥接。

### 最佳合作架构(给原生 Swift + 具身认知 + AR 渲染开发者)
- **认知层(LLM)**:SenseNova(商汤,HuggingFace 开源)或任意 OpenAI 兼容模型经星云/Duix SDK(都接受可插拔 LLM)。
- **端侧 avatar 渲染(若 2D 说话脸够)**:**Guiji Duix-Mobile iOS SDK**——今天唯一能用的 iOS SDK。移植:iOS→visionOS。
- **带身体动画的 3D 空间 avatar**:**魔珐星云**是唯一可信选项——**前提 Unity SDK 为真,先核**。退路:用星云 text→action API 作服务,把动作数据在你自己的 Vision Pro rig 上用 RealityKit 渲染。
- **角色资产创建/3D 生成**:魔珐图生 3D,或外包世优 2000+ avatar 库。
- **动捕(若需真人表演驱动)**:**世优 UCM/UCG/UCF 动捕硬件** + Puppeteer → BVH/FBX → 导入你的 RealityKit/Unity rig。4 家都无程序化 mocap SDK。
- **面部/AR CV(补充)**:SenseMARS Effects SDK——但 visionOS 上 ARKit 多半已覆盖。

### 授权现实核查
4 家都是中国商用厂商。**除 Guiji 的 Duix 仓库(GitHub)外,没有一家对生产级 avatar 技术提供干净的宽松开源授权**(且 Duix 也要核 LICENSE;定制 avatar 需商用联系)。预期 **per-call 计费(星云)、商用 SDK 授权(商汤)、或项目制(世优)**。美国出口管制与中国数据驻留法规也会影响——**商汤尤其曾在美国实体清单**。

## 来源(关键 URL)
- 商汤:sensetime.com/cn 、SenseAvatar、SenseMARS Effects、SenseAR Unity tutorial、HuggingFace org、HKEX listing
- 硅基:github.com/duixcom、Duix-Mobile、Duix-Avatar、Gitee Duix.ai、财联社 IPO、百度百科 司马华鹏
- 世优:4utech.com、波塔 Web SDK、Baidu Baike、B-round pedaily、36氪 pitchhub
- 魔珐:xmov.ai、xingyun3d.com、corec.cc 发布报道、VR陀螺 1.3 亿美元、cnblogs SDK 教程、CSDN 全栈教程

## unconfirmed 项
- 商汤 Unreal SDK 是否存在
- 硅基 Duix-Mobile 确切开源 license(LICENSE 未直接检视)
- 硅基 Duix-Mobile Apple Silicon(M2)运行时支持
- 魔珐星云 iOS/Android/Unity/Unreal SDK 可用性(社区声称,官方未直接核)
- 魔珐 Apple 平台端侧(非云)渲染
- 世优创始人名(纪智辉 跨源确认;"纪铮"/"CUAV" 未能坐实——CUAV 似为无关无人机飞控公司 cuav.net)
- 世优成立年(公司注册 2015 vs 叙述源的"2010")
**承诺合作前直接联系各家核 visionOS/Apple Silicon SDK 可用性。**
