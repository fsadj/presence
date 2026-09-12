# 中国数字人 / 产业 + visionOS SDK + 利基(合并版)

> Vision Pro 虚拟角色项目视角(原生 Swift,具身AI认知 + AR 渲染)。
> 状态标注:**✅**=联网核验、**⚠️**=待核;**纠错/勘误**标注随正文保留。
> 本文为目标结构合并文档。

## 来源映射

本文档合并自以下原文件:

| 原文件号 | 路径 | 主要贡献 |
|---|---|---|
| 05 | `research/_archive/05-china-digital-human-companies.md` | 数字人公司画像 1-4(商汤 / 硅基智能 / 世优 / 魔珐)+ 横切发现 |
| 06 | `research/_archive/06-china-digital-human-companies-batch2.md` | 数字人公司画像 5-8(网易伏羲 / 腾讯 / 影眸 / 数字王国)+ 纠错 |
| 08 | `research/_archive/08-china-visionos-sdks-and-niche.md` | visionOS SDK 矩阵 + HuggingFace org + AR/具身/虚拟人利基(初版) |
| 11 | `research/_archive/11-china-niche-mnn-taoavatar.md` | 利基升级版:MNN-TaoAvatar 为中心 + SDK 表/HF 表扩展版 |

去重处理:05 与 06 公司画像互补无重复;08 与 11 在 SDK 表 / HF 资产 / 利基三处大量重叠,**取 11 的扩展版为基,补充 08 独有的 URL 与旁系条目**。

---

## 1. 数字人公司画像(8 家,去重)

按 Vision Pro 契合度排名(来自 05 的 1-4 名 + 06 的 5-8 名):

| 排名 | 公司 | 对 Vision Pro 最佳资产 | iOS SDK | Unity/Unreal | 关键风险 |
|------|------|------------------------|---------|--------------|----------|
| 1 | **硅基智能 Guiji(Duix)** | Duix-Mobile 开源端侧 avatar SDK | **是**(原生)| 否(移动原生)| 2D 视频 avatar,非 3D mesh |
| 2 | **魔珐科技 Xmov(星云 Xingyun)** | 3D 具身AI avatar,SSML 动作控制;**已演示 Vision Pro** | 声称 | 声称(Unity+Unreal)| 云渲染串流;SDK alpha,原生 SDK 待核 |
| 3 | **商汤 SenseTime** | SenseMARS 面部追踪/AR(补充 ARKit)| 是(SenseAR/SenseME)| 是(SenseAR Unity SDK)| 无 avatar 动画 SDK 本身 |
| 4 | **世优科技 Shiyoo** | 动捕硬件 + BVH/FBX 数据管线 | 否 | 经 Puppeteer 管线 | 无公开 SDK;硬件/服务商 |
| 5 | **网易伏羲 NetEase Fuxi** | 玉言角色 LLM + 智能捏脸 + 4D 面扫 | 经云信 | 否 | B2B/合作制;已转机器人 |
| 6 | **腾讯 IVH(智影已死)** | 云 IVH iOS/Unity/UE SDK | 是 | 是 | 消费数字人战略收缩 |
| 7 | **影眸 Deemos(Hyper3D)** | Rodin 文/图→3D avatar 生成 | 否(云 API)| 是(原生插件)| 无运行时动画 SDK;"浙大 spinoff"是错的 |
| 8 | **Digital Domain 数字王国** | Douglas/Zoey 拟真数字人 VFX | 否 | Unreal 系 | 无 SDK/API,仅合作 |

**关键发现(横切):8 家都没有公开的 visionOS SDK 或已宣布的 Apple Vision Pro 合作。没有任何中国虚拟人厂商宣布 Apple Vision Pro 合作(中英双语激进搜索)。没有原生 Swift SDK——iOS 支持全是 Objective-C 或桥接。**

### 1.1 商汤科技 SenseTime ✅
- 中文:商汤科技有限公司。2014-10 成立于上海。**港股上市(0020.HK)**,2021-12-30。仍运营。~2,472 人。已故**汤晓鸥**(创始人)、现 CEO **徐立**。
- **签名产品**:如影 **SenseAvatar**(SaaS,数字人视频生成,**非 SDK**);**SenseMARS 火星混合现实平台**(最接近——含虚拟化身、特效引擎、三维重建);**SenseMARS 特效引擎 SDK**(美颜/贴纸/肢体特效,基于人脸关键点/手势/背景分割);**SenseME**(移动终端 SDK);**SenseAR**(AR SDK,**有 Unity SDK** 下载 openar.sensetime.com/sdks);SenseNova/秒画/琼宇/格物。
- **给 Vision Pro 带来什么**:面部追踪、AR 特效、手势、SLAM——**补充而非替代** ARKit/RealityKit。SenseAvatar 是视频输出 SaaS,无实时渲染 SDK。**无专门 3D avatar 动画 SDK**。SenseSpace/SenseThings 是场景/物体 3D 重建,非角色生成。发过《AI 数字人白皮书》定义 5 阶段。
- **SDK/授权/visionOS**:iOS SDK 是(商汤 SDK,经声网插件也用);Unity SDK 是;Unreal 未确认;**visionOS 无**。商用 SDK,per-app 授权,business@sensetime.com。SDK 平台:Android/iOS(SenseME/SenseAR Effects)。
- **HuggingFace**:https://huggingface.co/SenseTime — 5 个模型全是 **Deformable-DETR**(目标检测),**无 avatar/数字人/3D 模型**。SenseNova 在单独 collection;**SenseNova U1 Lite 系**(多模态 LLM,2025 末开源)。如影 SenseAvatar **不开源**(商业 SaaS)。
- **结论**:契合度有限。CV/AR SDK 与 ARKit 重叠;无实时 3D avatar 动画 SDK。可作 ARKit 补充或认知层 LLM,非主 avatar 渲染伙伴。
- ⚠️ **未核验**:搜索摘要称"商汤 2025 Tech Day 演示 SenseAvatar 跑在 Apple Vision Pro"。fetch 一手源(商汤 Facebook Tech Day 2025 帖)后**只宣布 SenseNova V6 和 SenseCore 2.0——无 Vision Pro 提及**。该主张按**未核验/疑似搜索摘要幻觉**处理。商汤 Unreal SDK 是否存在亦 unconfirmed。

### 1.2 硅基智能 Guiji AI(Duix)✅
- 中文:硅基智能(南京),2017 成立,**司马华鹏**。**已递表港 IPO(2025-11,2026 中重新递表)**,"数字人第一股",估值 31.5 亿元(2025-06),**腾讯 16.59% 为最大机构股东**。**勿与硅基流动 SiliconFlow 混淆**(不同公司)。
- **签名产品**:**Duix-Mobile**(开源移动 SDK,实时交互 AI avatar,**iOS+Android** 原生,<120ms,带 barge-in;https://github.com/duixcom/Duix-Mobile);**Duix-Avatar**(~10 秒视频克隆+lip-sync);**Duix.HeyGem**(talking-head 视频生成);**Duix-Reface**(实时换脸);DUIX 云 SaaS/API;DUIX.ONE 多模态大模型。GitHub 组织 https://github.com/duixcom 。
- **给 Vision Pro 带来什么**:**8 家中端侧 avatar SDK 最强**,唯一有公开文档的原生 iOS SDK 且**完全端侧**(无云依赖)——契合 Vision Pro 本地/低延迟偏好。设计上**接入你自己的 LLM/ASR/TTS**,契合"你出大脑、Guiji 出脸"的具身认知架构。
- **SDK/授权/visionOS**:iOS **明确支持**(README 列 iOS/Android/平板/车机/**VR**/IoT/大屏);Unity/Unreal 未提及(移动原生);**visionOS 无**,但 iOS 代码库是 8 家中最自然的 visionOS 移植起点。"开源 SDK"但**确切 license README 未显示(unconfirmed)**;定制 avatar 需邮件 support@duix.com(暗示生产用商用双授权)。**用前核 LICENSE**。
- **HuggingFace**:无 org 页(在 GitHub/Gitee)。
- **技术注意**:Duix-Mobile 渲染的是**2D 拟真 talking-head(视频神经渲染),非可控行架的 3D mesh**。适合"对话窗口脸",不能给 Vision Pro 空间环境里一个全关节 3D 身体。延迟数据基于骁龙 8 Gen 2,**Apple Silicon(M2)是否开箱支持 unconfirmed**。
- **结论**:**端侧 avatar 交互 SDK 成熟度最高,但是 2D 视频、非 3D 空间**。若 Vision Pro 角色是"说话的脸"而非全身空间 avatar,契合;iOS SDK 是全清单中最具体起点。

### 1.3 世优科技 Shiyoo(4utech.com)✅
- 中文:世优(北京)科技股份有限公司,2015-03-18(公司实体),**纪智辉**。国家级专精特新小巨人,披露融资 ~2 亿元,B 轮 >1 亿(2023-12)。
- ⚠️ **纠错**:brief 中的"纪铮"未确认;**CUAV 是另一家无人机飞控公司 cuav.net,疑为混淆**。成立年公司注册 2015 vs 叙述源的"2010"亦待核。
- **签名产品**:**Puppeteer 虚拟工场**(2012 起核心平台,动捕+面捕+实时渲染引擎,Unity/Unreal 管线);**世优波塔 BOTA**(AI 数字人 agent,3D/2.5D,面向数字大屏/全息柜/网页);**波塔 Web SDK**(**唯一公开 SDK**,15 分钟集成);**UCM-2 Pro 动捕服/UCG-2 Pro 手套/UCF-2 Pro 头盔**(UCFace 2.0,32 参数,<25ms);**UME 光学动捕**(成都棚);AI 直播系统。已建 2000+ 数字人。
- **给 Vision Pro 带来什么**:**8 家中唯一深度动捕硬件+生产管线**。若需高质量 mocap 数据(BVH/FBX)驱动 avatar,UCM/UCG/UCF + Puppeteer 能产。生产服务导向,非开发者 SDK 导向。
- **SDK/授权/visionOS**:公开 SDK 仅波塔 Web SDK(JS,大屏);**无 iOS/Android/Unity/Unreal 公开 SDK**(动捕数据经 Puppeteer 进 Unity/Unreal 作生产管线输出 BVH/FBX,非运行时 SDK);visionOS 无。模式:**定制方案交付 + SaaS**。
- **结论**:**非 SDK 伙伴**。价值在动捕硬件 + 内容生产服务。若要(a)买动捕服给 Vision Pro 角色动画,或(b)外包数字人资产,联系他们。波塔 Web SDK 面向柜台/大屏,非空间计算。

### 1.4 魔珐科技 Xmov(星云 Xingyun)✅
- 中文:魔珐(上海)信息科技,2018 成立,**柴金祥**,上海徐汇。披露融资 ~1.3 亿美元(2022-04 B+C),早期投资人**红杉中国/晨兴/沈向洋**。1000+ 企业客户。
- **签名产品**:**魔珐星云 Xingyun**(2025-10-29 发布,"全球首个具身智能 3D 数字人开放平台",理念"语言驱动身体"——LLM 输出动作参数驱动 3D 人或机器人,非预渲染视频;xingyun3d.com);**魔珐有言 Youyan**(文生 3D 视频);有光/有灵;虚拟人 IP "**Ada/Ava**"(中国首例虚拟人法律纠纷中心);旗舰虚拟人 agent **"镜 JING"**(2023-05 发布的虚拟人 3.0 智能体);**Xingyun litesdk(xmovAvatar.js)** Web SDK(`speak(ssml)`/`speakWithAction`/`think`,动作语义 Hello/Agree/Think... 经 `<ue4event>` SSML 标签)。
- **技术能力**:52 面部 blendshape、全身骨骼驱动、呼吸 idle;**<500ms 端到端**(ASR 流式+LLM 流式+<100ms TTS+<50ms avatar 驱动);barge-in;3000+ 超写实 3D avatar 库、图生 3D;千万级并发;信创私有化(飞腾/鲲鹏、麒麟/统信、等保三级+商密)。云-边端分离架构:**云端生成语音+动作参数,边端 AI 渲染**,**端到端 <1.5s,千万级并发,百元级芯片(RK3566/3588)可跑**;经 SDK 或 API 接入,面向屏(移动/平板/PC/TV/大屏/全息舱)、人形机器人、"任意终端"。
- **Vision Pro demo(✅ 已确认,非 SDK)**:Xmov 在 app/内容层在 Apple Vision Pro 上演示 3D 虚拟人。中文产业媒体直引:"魔珐科技用苹果 Vision Pro 为我们打开了 3D 虚拟人应用的新场景。在 Vision Pro 中,与虚拟人的面对面交互由屏幕过渡到空间,我们与虚拟人的'隔阂'消失了"。
  - 源:https://mp.ofweek.com/ce/a956714267417 , https://t.cj.sina.cn/articles/view/6012711797/16662b375001018852
  - JING 介绍:https://column.iresearch.cn/b/202306/962263.shtml
  - 发布报道(量子位 2025-10-30):https://www.qbitai.com/2025/10/347284.html
- **SDK/授权/visionOS**:Web SDK **确认**(CDN JS,多 Vue3 demo);**iOS/Android/Unity/Unreal 声称但官方文档未能直接核验**(社区教程称"统一 API 支持 Web/iOS/Android/Unity/Unreal",官方称"多终端 一次开发到处运行",但**唯一直接可检的是 Web JS SDK**;原生/Unity/Unreal 下载链接未定位——**按"声称,待与魔珐核"处理**)。SDK 平台文档列:Web,大屏,服务端 API(Android/iOS/鸿蒙边端渲染声称)。渲染架构**本质云串流**(连 gateway 下载资源 Canvas/视频流渲染);社区称"端侧渲染 SDK/免显卡/百元芯片"但 Apple 平台端侧 unconfirmed。付费(per-call);visionOS 无。
- **结论**:**8 家中 3D avatar 技术最佳,但集成风险高**;也是最接近的商业"具身AI+数字人"平台。若声称的 Unity SDK 真存在,路径:星云 Unity SDK → Unity visionOS(PolySpatial)→ Vision Pro。**承诺前向魔珐核实 Unity SDK**。云串流默认架构对空间计算延迟不利,追问 Apple Silicon 的"端侧渲染"选项。屏基"虚拟具身"(其术语),非 AR/空间。无 visionOS 移植,但边端渲染架构概念上契合耦合/系留头显。
- ⚠️ **未核验**:iOS/Android/Unity/Unreal SDK 可用性(社区声称,官方未直接核);Apple 平台端侧(非云)渲染。

### 1.5 网易伏羲 NetEase Fuxi ✅
- 网易 AI 研究实验室,2017 成立。负责人**范长杰**(USTC 博士,~200 专利 ~60 顶会论文)。[百度百科](https://baike.baidu.com/item/网易伏羲)
- **当前战略(重要)**:已**大幅转向具身AI/工业机器人**(网易灵动 Lingdong:挖掘/装载机器人)与众包/agent。数字人工作仍在但不再是头条。产品见 fuxi.163.com:网易灵动(旗舰)、绘梦天工(30+ 游戏资产,含逆水寒/永劫无间)、有灵众包、有灵智能体+AOP(Agent-Oriented Programming)、妙启AI对话。
- **签名模型/资产**:**玉言**(~11B 中文 LLM,deep encoder+shallow decoder,CLUE 登顶);**玉知**(VLM);**丹青**(图像生成);**易生诸相**(角色扮演/character LLM 底座);**4D 面部扫描**(逆水寒叶雪青);**智能捏脸**(照片→3D 面部参数,永劫无间/逆水寒);**智能动作/表情捕捉**(普通摄像头无标记动捕/表情捕捉);**语音驱动表情动画**(GDC 2021);玉言驱动逆水寒 AI NPC(并邀通义/文心/abab/月之暗面/豆包入园);获**国家科技进步奖**(数字人)。
- **给 Vision Pro 带来什么**:认知(玉言角色扮演 LLM);avatar 创建(智能捏脸、4D 面扫、语音驱动面部动画——对 AR avatar 直接有用);动捕(仅摄像头动捕/表情捕捉);实时交付见下云信。
- **访问/SDK/iOS-visionOS**:伏羲自身能力是 **B2B/合作制**,非交钥匙公开 SDK;AOP 是集成框架。**iOS 可访问的具体路径是网易云信(WXIN)**:实时对话 AI agent 驱动 AI 数字人(音视频),**跨平台 SDK 含 iOS**(NIM SDK v10.8.30,流式输出)。无原生 visionOS SDK。
- **HuggingFace**:https://huggingface.co/FUXI — 7 模型:Multi-modal_10B_CN、yuyan-11b/10b/dialogue、danqing-caption。数字人研究多私有;公开含 **FaceG2E**(文→3D 脸)。研究侧:**Make-A-Character**(文→3D lifelike avatar,网易+密歇根)、**FreeAvatar**(arXiv 2409.13180)——均为论文,无可下载 HF 权重。玉言权重 2022-23 曾部分上 HuggingFace(NetEase-Fuxi org),当前开源状态**unconfirmed**(已转机器人)。
- **结论**:认知(LLM)+面部动画+摄像头动捕强,但多内部/游戏 B2B。原生 iOS 唯一打包路径是云信实时 AI-agent + 数字人 SDK。

### 1.6 腾讯 IVH(搜狗血脉;智影已死)✅
- **历史**:搜狗分身是搜狗核心 AI,2018 首发**全球首个 AI 合成主播**(与新华社),克隆主播**邱浩/"雅妮"**,2019 全球首个站立 AI anchor(WaveRNN 语音+唇/表情合成)。**王小川**时任搜狗 CEO。
- **收购与状态**:腾讯 2021 私有化收购搜狗。avatar 技术被吸收、跨多个腾讯业务重组:

  | 产品 | 状态(2026)|
  |---|---|
  | 搜狗分身 | 已吸收,非独立品牌 |
  | **腾讯智影 Zhiying** | **已死**。2025-04-15 停注册,2025-06-30 停登录删数据,2025-07-01 起"升级维护中",官方账号 9 月注销。腾讯未给官方理由。 |
  | **腾讯云智能数智人 IVH** | **活跃——真正继任者**。cloud.tencent.com/product/ivh |
  | 腾讯妙思 Miaosi | 活跃(营销 AIGC)|
  | "奇妙数字人"(3000+ avatar,WAIC 2025)| 活跃 |

- **大信号**:腾讯另在**计划封禁直播电商 AI 数字人主播**——消费级数字人战略收缩,企业级 IVH 继续。
- **活跃 SDK:腾讯云 IVH(对 Vision Pro 最重要)**:**小样本克隆**(3 分钟视频+100 句语音→~24 小时克隆);集成 H5/Android/**iOS**,渲染引擎 **WebGL/Unity/Unreal**,协议 **RTMP/WebRTC**;**客户端渲染 SDK**(Android+iOS,端侧 lip-sync/渲染);云渲染 H5 SDK v5.5.0。另有 Tencent Avatar SDK(腾讯云视立方)+ 智能数智人 iVH:iOS/Android/Web(https://cloud.tencent.com/document/product/1662/85737 )。
- **结论**:搜狗血脉**仅存于腾讯云 IVH**。腾讯智影已没。Vision Pro 开发者的唯一腾讯路径(有真 iOS/Unity/UE SDK)是 IVH,但注意腾讯整体对消费数字人降温。

### 1.7 影眸科技 Hyper3D / Deemos ✅(含重要纠错)
- 影眸科技(上海)有限公司 / Deemos,2020 成立(注册 2020-06-24)。
- ⚠️ **纠错:非浙江大学 spinoff**。影眸在**上海科技大学(ShanghaiTech)**孵化——MARS Lab / Visual & Data Intelligence Center。创始人**吴迪(CEO)**、**张启煊(CTO,1999 生,上科大 2018 本→2022 硕)**、**张龙文**、**曾初啸**。2020 获上科大 IP 授权。无公开来源指向浙大。
- **投资**:**凯辉**、**上海国投先导**(领投)、**字节跳动**等;**光源资本** FA。ARR 数千万美元。入选**英伟达黄仁勋 CES 主旨** 3D 资产工作流——唯一入选的 3D 生成创业公司。
- **签名产品**(hyper3d.ai):**Rodin**(文/图生 3D 旗舰;Gen-1(1.5B DiT,PBR,四边拓扑)/Gen-1.5/Gen-2(~10B,BANG,2025-10)/**Gen-2.5**——全球首个**千万级多边形** 3D 生成,"think-then-generate" LLM 式推理);**Hyper3D** 平台;**ChatAvatar**(文/图→3D avatar/脸,2023 曾在 HF Space);OmniCraft、AI 纹理/HDRI 生成、3D mesh 编辑器、3D 模型搜索引擎。
- **Vision Pro 相关度**:**3D 资产/avatar 生成最佳**。站点明确列 VR/AR、角色设计、游戏、动画、元宇宙。输出生产级 mesh+UV+纹理+PBR。
- **插件/SDK/定价——DCC 覆盖强**:原生插件 **Blender/Unity/Unreal/Godot/Maya/3DS Max/ComfyUI**;REST API。定价:免费 1000 credit/月(个人);Pro ~$20/月;API ≈ **$0.30–0.50/生成**。亦在 fal.ai($0.40)、wavespeed($0.30)。**无原生 iOS/visionOS SDK**——是云生成服务 + DCC 插件,非运行时 avatar 动画引擎。Vision Pro 用法:服务端调 API 生成 mesh/纹理,再导入/动画(如 MetaHuman、ARKit blendshape);Unity 插件可喂 Unity→visionOS 管线。
- **HuggingFace**:org `Deemos` 与 `DEEMOSTech` 均**无公开权重模型**("None public yet"——**不在 HF 发权重**)。仅研究论文:*Strips as Tokens*、*Kinematify*。旧 ChatAvatar HF Space(2023)已失效。**RodinHD**(arXiv 2407.06938,GitHub RodinHD)是**学术研究论文,非商业 Rodin 模型**,研究用途条款。经 Spaces(Rodin)与 RodinHD 论文发布。
- **Rodin 授权**:**封闭/商用**。生产 Rodin 无开源权重。商用需 API 订阅或企业授权(具体转售条款 unconfirmed,读 ToS)。
- **结论**:生成高保真 3D avatar/资产出色,有原生 Unity/Unreal 插件。非运行时动画或 mocap 方案,无原生 iOS/visionOS SDK。**"浙大 spinoff"前提错——是上海科大**。

### 1.8 Digital Domain 数字王国 ✅(含领导层澄清)
- Digital Domain Holdings,好莱坞 VFX(1993,**James Cameron** 等),母公司**港股 547.HK**。大中华首家独立 VFX 工作室。办公室:LA/Vancouver/Montreal/**京沪深港台北**/Hyderabad。中国实体:數字王國朝霆(上海)。
- **领导层澄清**:**谢安 = Daniel Seah(SEAH Ang)**——同一人。2014 入任 CEO,后升**主席**,主导 MBO。(中文媒体用谢安,英文用 Daniel Seah——**非两人**。)
- 2018 收购 **3Glasses**(深圳 VR)。
- **数字人工作**:**邓丽君全息**(标志性,2013 周杰伦演唱会,后独立演唱会,AI 驱动版讨论过);**龚俊数字人**;**"Douglas"(2020)**——自主实时拟真数字人(POC,**Doug Roble** 肖像),ML 驱动:语言处理/表情/视觉追踪/换脸/语音复制(~30 分钟音频/10 分钟视频);**"Zoey"(2022)**——最先进自主数字人,Douglas 继任;早期 DigiDoug 实时角色。技术栈 **Unreal Engine + ML + 深度学习 + 虚拟制作**。拥有 **NUKE** 合成软件(2017 奥斯卡科技奖)。2023 在**香港科学园建 2600 万美元虚拟人研发中心**。
- **给 Vision Pro 带来什么**:VFX 级拟真数字人渲染 + 全息专长(最"好莱坞级"选项);自主数字人 know-how(Douglas/Zoey:感知/表情/对话——概念上接近具身AI角色);VFX/mocap 血统(表演捕捉、neural rendering)。
- **访问/SDK/授权——catch**:**无公开 SDK、无开源、无 API 产品**。接触是**合作/联合开发/定制**。Douglas 联系 dhginfo@d2.com。无 iOS/visionOS SDK。基于 Unreal——理论上可经 Unreal visionOS 支持 target,但 DD 自己不发此集成。邀请"投资、合作与客户工作"围绕 Douglas。
- **结论**:高端定制 VFX/拟真数字人伙伴,非即插 SDK。仅在预算与时间允许联合开发做超高保真 avatar 时相关。无交钥匙开发者产品。

---

## 2. avatar SDK 平台表(合并 08 + 11,取 11 扩展版)

### 标题结论
**【无中国厂商有 visionOS 原生 avatar SDK】**——截至 2026-07,没有任何中国厂商公开发布或宣布 visionOS 原生 avatar/动画 SDK(相芯/商汤/魔珐/腾讯/阿里/百度/字节/讯飞/PICO 皆 iOS/Android/Unity,**无一 visionOS/xrOS/RealityKit**)。最接近:(1) 会以 iPad/iOS 兼容模式跑在 Vision Pro 上的 iOS SDK;或 (2) 内容级 Vision Pro demo(非 SDK)。**WWDC 2024/25/26 零提及中国 avatar 合作伙伴**;无任何中国厂商的公开 RealityKit/PolySpatial 集成。这是真实可利用的缺口。

### 各厂 SDK 平台矩阵

| 厂商 | 产品 | 文档 SDK 平台 | Unity/Unreal | visionOS 原生? | 备注/URL |
|---|---|---|---|---|---|
| **相芯 FaceUnity** | AvatarX(Cubic 引擎+Unity)/AR 特效 | Android,iOS,Unity,Web,Win/Mac,鸿蒙 | Unity | 否 | https://www.faceunity.com/avatarxsdk.html ;开发者中心 https://www.faceunity.com/developer/ |
| **商汤** | SenseME/SenseAR Effects | Android,iOS | Unity(SenseAR) | 否 | https://www.sensetime.com/cn/product-business?categoryId=79 ;openar.sensetime.com/sdks |
| **硅基智能 Guiji** | Duix-Mobile(开源) | iOS,Android 原生(+平板/车机/VR/IoT/大屏) | 否 | 否 | https://github.com/duixcom/Duix-Mobile ;<120ms,完全端侧 |
| **魔珐 Xmov** | 魔珐星云 SDK | Web,大屏,服务端 API(Android/iOS/鸿蒙边端渲染**声称待核**)| 声称(Unity+Unreal,**待核**)| 否(有 app 级 Vision Pro demo)| https://xingyun3d.com |
| **腾讯云** | 智能数智人客户端渲染 SDK / Tencent Avatar SDK(腾讯云视立方)/ iVH | Android,iOS,Web | Unity/Unreal(WebGL/UE 渲染) | 否 | https://cloud.tencent.com/document/product/1662/85737 ;https://cloud.tencent.com/product/ivh ;RTMP/WebRTC;云渲染 H5 SDK v5.5.0 |
| **阿里云** | 万相 Wanxiang Digital Human(纯端侧音驱 iOS SDK + 云渲染 iOS SDK)/虚拟数字人 SDK | iOS(RTC),Android,Web | 否 | 否 | https://help.aliyun.com/zh/avatar/avatar-application/developer-reference/digital-people-conversation-ios-sdk-local-avatar-only |
| **百度** | 3D 数字人交互 SDK | Android(iOS 可能) | 否 | 否 | — |
| **火山引擎(字节)** | 虚拟数字人平台 | 云管线(2D/3D),需商务联系 | 否 | 否 | https://www.volcengine.com/product/avatar |
| **讯飞 iFlytek** | 虚拟数字人平台 + 超拟人交互 SDK | SaaS/API;Web SDK 2.0 公开(iOS 未公开文档) | 否 | 否 | https://virtual-man.xfyun.cn/ |
| **PICO(字节 VR)** | PICO Avatar SDK | PICO VR 设备 | 否 | 否(封闭 VR 生态) | — |
| **网易(经云信)** | AI-agent 数字人 SDK(NIM SDK v10.8.30) | iOS(跨平台) | 否 | 否 | 经网易云信 WXIN,B2B |

### 中国 avatar 在 Vision Pro 上的公开演示
**仅一个——阿里 MNN-TaoAvatar**(详见 §4)。中国 app 上 Vision Pro 的有微信/钉钉/携程/淘宝/高德/招行;visionOS 26/27 加了 Foveated Streaming 云渲染(均非 avatar SDK)。

### Unity→visionOS 路径(因 FaceUnity AvatarX 用 Unity 而相关)
Unity **PolySpatial** 让 Unity 场景经 RealityKit 跑在 visionOS。这是中国 Unity 系 avatar 技术(如 AvatarX)进 visionOS 最 plausible 的集成路径,但**无中国厂商发布 PolySpatial 移植**。https://unity.com/cn/blog/engine-platform/unity-support-for-visionos

**小结**:原生 Swift/RealityKit Vision Pro app,**当前无法授权到成品中国 avatar SDK**。要么 (a) 基于中国厂商云渲染 RTC 管线(阿里万相、腾讯 iVH),(b) 经 Unity PolySpatial 集成 Unity 系 SDK(FaceUnity AvatarX),(c) 向魔珐定制移植,或 (d) 用 RealityKit + 开源模型管线(如 MNN-TaoAvatar)做原生端侧渲染。Xmov Vision Pro demo 证明**内容**可达,但 **SDK 未上架**。

---

## 3. HuggingFace / ModelScope 资产(合并 08 + 11,取 11 扩展版)

| Org | URL | 相关公开模型 | 注 |
|---|---|---|---|
| **腾讯** | https://huggingface.co/tencent(collection hunyuan3d)| **Hunyuan3D-1/-2/-2.1/-Part**(文/图→3D mesh,PBR,可编辑部件)、**Hunyuan3D World Model 1.0**(首个开源 3D 世界模型)、**HunyuanVideo-Avatar**(音→avatar 视频,2025-05)、**HunyuanCustom** | **最强开源 3D+avatar 阵容**。GitHub https://github.com/Tencent-Hunyuan ;https://huggingface.co/tencent/Hunyuan3D-2.1 ;https://3d-models.hunyuan.tencent.com/ |
| **阿里 alibaba-pai** | https://huggingface.co/alibaba-pai | ~124 模型;**AgenticQwen-8B/30B-A3B**(agentic Qwen);Z-Image-Fun 图像;Wan2.2-Fun 视频;**RynnBrain 1.1 具身基础模型** | EMO 权重**未发布**(仅论文,GitHub 仅论文链接,无实现) |
| **阿里 3DAIGC(阿里 Tanji)** | https://huggingface.co/3DAIGC | **LHM-1B-HF**——大尺度可动画人重建,单图→可动画 3DGS avatar 秒级(ICCV 2025) | https://github.com/aigc3d/LHM |
| **阿里 TaoAvatar / Live Avatar** | 论文/项目页 | **TaoAvatar**(arXiv 2503.17032,CVPR 2025)— 实时、轻量 3DGS 全身 talking avatar,明确为移动端 AR 设计;**Live Avatar**(阿里-夸克,arXiv 2512.04677,ECCV 2026 Oral)— 流式实时音驱 avatar,与 **Qwen3-Omni** 集成做交互对话 agent | 权重在 **ModelScope**(非 HF)。https://pixelai-team.github.io/TaoAvatar/ ;https://github.com/Alibaba-Quark/LiveAvatar 。阿里真·开源 avatar 资产是 **MNN-TaoAvatar** |
| **ModelScope(阿里自家 hub)** | https://www.modelscope.cn/ | 平行 HF,有开源实时数字人对话 demo;MNN-TaoAvatar 模型在此 | https://modelscope.cn/collections/TaoAvatar-68d8a46f2e554a |
| **影眸 Deemos** | https://huggingface.co/Deemos | **"None public yet"**——不在 HF 发权重。RodinHD 仅论文(arXiv 2407.06938) | Rodin/Hyper3D **商业 API,无开源权重**。org `Deemos` 与 `DEEMOSTech` 均 0 模型 |
| **字节 `ByteDance`** | https://huggingface.co/ByteDance | **UniVR-34B-Planning**(VR 规划模型,图+文→文);Bernini-R(14B 图文生视频);Ouro-1.4B/2.6B;数据集 **VR-X-SFT-RL**(23.8 万行 VR 相关)、veAgentBench | 共 54 模型 |
| **字节 `ByteDance-Seed`** | https://huggingface.co/ByteDance-Seed | **LatentSync**(音驱 lip-sync,开源 U-Net+SyncNet+Whisper ckpt;https://huggingface.co/ByteDance/LatentSync );**UI-TARS-1.5-7B**(GUI agent);**Seed3D 1.0**(图→高保真仿真级 3D 资产,arXiv 2510.19944) | LatentSync 是可用开源资产;OmniHuman-1/1.5(https://omnihuman-lab.github.io/v1_5/ )研究/API 无开源;GauHuman CVPR 2024 论文 |
| **商汤** | https://huggingface.co/SenseTime | 仅 5 模型,全是 **Deformable-DETR**(检测);SenseNova U1 Lite 系(多模态 LLM,2025 末开源) | **HF 上零数字人/avatar 模型**;如影 SenseAvatar **不开源**(商业 SaaS)。SenseNova 在单独 collection |
| **美团** | https://huggingface.co/meituan-longcat | **LongCat-Video-Avatar / -1.5**(音驱 avatar 视频;AT2V/ATI2V;商业级;基于 13.6B LongCat-Video) | 近期发布的强开源 avatar 模型。GitHub https://github.com/meituan-longcat/LongCat-Video |
| **网易伏羲** | https://huggingface.co/FUXI | 7 模型:Multi-modal_10B_CN、yuyan-11b/10b/dialogue、danqing-caption、**FaceG2E**(文→3D 脸) | Make-A-Character(网易+密歇根)、FreeAvatar(arXiv 2409.13180)均为论文,无可下载 HF 权重 |

**策展索引**:`weihaox/awesome-digital-human`(https://github.com/weihaox/awesome-digital-human )是上述最佳策展索引。

---

## 4. AR + 具身AI + 虚拟人 利基(以 TaoAvatar 为中心)

诚实判定:严格交集(**AR 渲染 + 具身/多模态 AI + 虚拟角色,来自中国**)干净命中三者的项目极少。中国"具身AI"偏物理机器人(人形);"数字人"多屏基(直播/客服)。**唯一清晰融合 AR 渲染 + 端侧多模态AI + 虚拟角色**的项目是 MNN-TaoAvatar。

### Tier 1 — 直接命中

#### 4.1 阿里 MNN-TaoAvatar(淘宝 Meta 团队)— THE 答案【本项目最该学的中国工程先例】
- 全身拟真 3D talking avatar,基于 **3D Gaussian Splatting**,完整**端侧**多模态管线:**MNN-ASR + MNN-LLM(Qwen2.5-1.5B)+ MNN-TTS + A2BS(音→blendshape)+ MNN-NNR(神经渲染)**。骁龙 8 Gen 3 上 **60 FPS**。
- 论文:**"TaoAvatar: Real-Time Lifelike Full-Body Talking Avatars for Augmented Reality via 3D Gaussian Splatting"**(arXiv 2503.17032,CVPR 2025)。项目 https://pixelai-team.github.io/TaoAvatar/
- 发布文明确称"能在手机或 XR 设备上实现 3D 数字人的实时渲染以及 AI 对话",并展示"在 Android 手机及 **Apple Vision Pro 设备**上的体验效果"——**【已演示 Apple Vision Pro】**。https://m.thepaper.cn/newsDetail_forward_31039750
- 经阿里 **MNN** 框架开源:https://github.com/alibaba/MNN(apps/Android/MnnTaoAvatar;README /blob/master/apps/Android/MnnTaoAvatar/README_CN.md)。模型在 **ModelScope**:https://modelscope.cn/collections/TaoAvatar-68d8a46f2e554a
- **对 Vision Pro 项目意义**:MNN 跨平台 C++(**iOS/visionOS 受支持**),这是原生 visionOS 具身角色最可移植起点。开源发布是 Android 打包,需自己把管线移植进 RealityKit/visionOS。论文标题即 "...for Augmented Reality"。

#### 4.2 魔珐 Xmov / 魔珐星云(xingyun3d)— 最强商业"具身AI+数字人"平台
- 自称(量子位 2025-10-30)"**全球首个具身智能 3D 数字人开放平台**"——给 LLM/agent 一个身体:实时 文本→3D avatar 言语/表情/注视/手势/身体动作。创始人**柴金祥**。
- 云-边端分离:云端生成语音+动作*参数*,边端 AI 渲染——**端到端 <1.5s,千万级并发,百元级芯片(RK3566/3588)可跑**,信创芯片。经 **SDK 或 API** 接入;面向屏(移动/平板/PC/TV/大屏/全息舱)、人形机器人、"任意终端"。
- 源:https://www.qbitai.com/2025/10/347284.html ;https://xingyun3d.com
- **caveat**:屏基"虚拟具身"(其术语),**非 AR/空间**。无 visionOS 移植。但边端渲染架构概念上契合耦合/系留头显。**Vision Pro demo 已确认(内容级,非 SDK)**,旗舰 agent "镜 JING"(2023-05)。

#### 4.3 阿里 Live Avatar + Qwen3-Omni(ECCV 2026 Oral)— 最接近"具身多模态AI+虚拟角色"的开源栈
- 流式实时音驱 avatar,作者明确与 **Qwen3-Omni** 结合做"完全交互对话 agent"。https://github.com/Alibaba-Quark/LiveAvatar ,arXiv 2512.04677。

### Tier 2 — 旁系开源/商业资产(缺一根柱)

#### 4.4 美团 LongCat-Video-Avatar / -1.5
- 13.6B(基于 LongCat-Video),音频驱动 avatar 视频;AT2V/ATI2V;商业级。https://huggingface.co/meituan-longcat/LongCat-Video-Avatar-1.5 ;https://github.com/meituan-longcat/LongCat-Video 。**自建管线时的音驱 talking-face 开源层**。

#### 4.5 腾讯 HunyuanVideo-Avatar
- 音频→avatar 视频,2025-05。https://huggingface.co/tencent/HunyuanVideo-Avatar 。配合 **Hunyuan3D-2** 生成 3D avatar 资产 + 视频 avatar 退路。

#### 4.6 字节 LatentSync
- 音频驱动 lip-sync,开源 **U-Net + SyncNet + Whisper** ckpt。https://huggingface.co/ByteDance/LatentSync 。自建管线时的音驱 talking-face 开源层。**盯字节 UniVR-34B-Planning + VR-X-SFT-RL**(23.8 万行 VR 相关)——字节公开发 VR/XR 规划模型,暗示内部 VR-agent 努力。

#### 4.7 蚂蚁灵波 LingBot
- 严肃具身AI"机器人脑"栈:**LingBot-VA 2.0 世界动作模型、VLA、Vision/Depth 空间感知**。重机器人,非虚拟人。

#### 4.8 百度度晓晓 / Xiaodu(旁系,非 AR)
- 3D 数字人 AI 陪伴(多模态、情感/陪伴),手机 app。**非 AR/空间**。另有 DuMix AR 但未与数字人产品化为一体。"数字人家族"(度晓晓/希加加/林开开/叶悠悠)是 2D/手机屏/元宇宙(希壤)——**非 AR 具身**。小度 AI Glasses Pro + "超能小度"(2025-11)是 Meta-Ray-Ban 式语音/摄像头助手——**无具身 3D 虚拟角色**确认。

#### 4.9 Rokid(AR 眼镜 + 数字人技术栈,但非 Vision Pro)
- 核心列"数字人技术":30+ 骨骼/动作节点、多角色自适应、ASR/TTS/AIGC,定位 NPC + 办公对话。AR Studio 含 SLAM+3D 手势+6DoF+空间音频+数字人。https://www.rokid.com/zh-CN/technology 。**确是"AR+虚拟角色",但不在 Vision Pro**——是 Rokid 自家 AR 眼镜平台。注:Rokid 现售 AI 眼镜(2026 Style)是语音+摄像头智能眼镜(ChatGPT-5/Gemini),**出货产品上无 3D 虚拟角色陪伴**,只有语音助手。

#### 4.10 高校(清华/浙大/上交/智源)
- 强具身AI/机器人研究:**清华 EIR**(2025 末)+ **IIIS VAR** + **MARS Lab** + **AIR** + **KEG/智谱 CogAgent** GUI agent;**AIR+阿里云"可进化 agent"**;**ZJU 余姚具身**;**智源**。**但未发现具体开源高校项目结合 AR 渲染+具身AI+虚拟人角色**——中国学术具身工作主要是物理机器人/VLA。最清晰 AR+AI+avatar 工作是工业界 MNN-TaoAvatar。

### 否定项(必须保留)

- **Rokid 与 XREAL 不出货虚拟陪伴角色**。Rokid Glasses=语音 AI 助手(翻译/提词/导航/AI识物)+ AR 文字/信息叠加——无 3D avatar。**XREAL**=空间显示器(大屏视频),偏影音,未发现原生虚拟陪伴角色。两者均无具身虚拟人。[XREAL unconfirmed——大概率不存在为出货功能]。
- **百度是 2D 屏基非 AR**(度晓晓/希加加/林开开/叶悠悠;希壤元宇宙;小度 Glasses 是语音/摄像头助手)。
- **ShowUI / OS-Copilot / FRIDAY 非本利基**。**ShowUI**(CVPR 2025,4.2B VLA 用于 GUI 自动化)是 **ShowLab(邵岭 Mike Zheng Shou,NUS——新加坡,非大陆)** 的 GUI VLA agent,自动化屏幕 UI,无 AR 渲染、无虚拟角色。https://github.com/showlab/ShowUI 。**OS-Copilot / FRIDAY**(https://os-copilot.github.io/ )通用 OS 级 agent 开源框架,作者来自上海AI Lab+华东师大+Princeton+港大;一作**吴志勇**现**在字节 Seed**。GUI/电脑 agent,非 AR/虚拟人——"具身"指软件具身,非物理/AR 具身。
- **B 站"Vision Pro 数字人"视频**(如 BV18H1TBnEUR)讲的是 **Apple 原生 Persona**,非中国 avatar 技术。
- **网易伏羲、商汤、字节**发 avatar *研究*(Make-A-Character、FreeAvatar、OmniHuman)但无一打包成 AR+具身 agent+虚拟人产品。

### 利基判定小结
- 严格"AR 渲染 + 具身/多模态AI + 虚拟角色,来自中国,在 Vision Pro 上"利基**在产品级基本未被占据**。唯一确认的中国 Vision Pro + 虚拟角色 demo 是**魔珐 Xmov**(内容 demo,非产品化 SDK);唯一确认的"AR+端侧多模态AI+虚拟角色"开源项目是**阿里 MNN-TaoAvatar**(且已演示 AVP)。
- **最强开源中国栈**(映射本项目目标):**阿里 MNN-TaoAvatar(3DGS 身体)+ Live Avatar(实时流式)+ Qwen3-Omni(多模态大脑)**——组合即具身 Vision Pro 角色的组件,但集成需自己做。
- **最强商业中国伙伴**(要原生中国 avatar 技术):**魔珐 Xmov**(实测 Vision Pro demo、"镜 JING")或**相芯 FaceUnity**(Unity 系 AvatarX SDK,可经 Unity PolySpatial 上 visionOS)。
- **未发现任何 WWDC 2024/25/26 提及中国 avatar/数字人合作伙伴**。[unconfirmed——可能存在但搜索不可发现]。

---

## 5. 对接 / 讲故事定位(本项目相对中国玩家的差异化)

### 给 visionOS 开发者的伙伴/技术短名单
1. **阿里 MNN-TaoAvatar**——最佳开源基座,适配进原生 visionOS 角色(跨平台 MNN、3DGS 渲染、完整端侧多模态AI、已 AVP 演示)。**首推**。
2. **魔珐 Xmov(魔珐星云)**——最佳商业"具身AI 数字人"平台,要交钥匙云-边端 SDK 可合作(有开发者计划,明邀 SDK/API 集成)。
3. **腾讯 Hunyuan3D-2 + HunyuanVideo-Avatar**(开源)——生成 3D avatar 资产 + 视频 avatar 退路。
4. **相芯 FaceUnity**——成熟中国 avatar/AR 特效 SDK 厂商,移动/Unity 覆盖最广;最自然去*请求* visionOS 移植的伙伴(已支持 iOS+Unity,RealityKit/visionOS 桥接可行)。
5. **美团 LongCat-Video-Avatar / 字节 LatentSync**——自建管线时的音驱 talking-face 开源层。

**无一今日有出货 visionOS SDK**——印证合作机会真实且时间敏感。

### 最佳合作架构(给原生 Swift + 具身认知 + AR 渲染开发者)
- **认知层(LLM)**:SenseNova(商汤,HuggingFace 开源)或任意 OpenAI 兼容模型经星云/Duix SDK(都接受可插拔 LLM);或 Qwen3-Omni(多模态);腾讯混元(另述)。
- **端侧 avatar 渲染(若 2D 说话脸够)**:**Guiji Duix-Mobile iOS SDK**——今天唯一能用的原生 iOS SDK。移植:iOS→visionOS。
- **带身体动画的 3D 空间 avatar**:**魔珐星云**是唯一可信商业选项——**前提 Unity SDK 为真,先核**。退路:用星云 text→action API 作服务,把动作数据在你自己的 Vision Pro rig 上用 RealityKit 渲染。**开源退路:MNN-TaoAvatar 端侧 3DGS 管线**。
- **角色资产创建/3D 生成**:影眸 Rodin(最高保真,商业 API),或魔珐图生 3D,或外包世优 2000+ avatar 库,或腾讯 Hunyuan3D-2 / 阿里 LHM-1B-HF(开源)。
- **动捕(若需真人表演驱动)**:**世优 UCM/UCG/UCF 动捕硬件** + Puppeteer → BVH/FBX → 导入你的 RealityKit/Unity rig。8 家都无程序化 mocap SDK。
- **面部/AR CV(补充)**:SenseMARS Effects SDK——但 visionOS 上 ARKit 多半已覆盖。

### 授权现实核查
8 家都是中国商用厂商。**除 Guiji 的 Duix 仓库(GitHub)外,没有一家对生产级 avatar 技术提供干净的宽松开源授权**(且 Duix 也要核 LICENSE;定制 avatar 需商用联系)。预期 **per-call 计费(星云)、商用 SDK 授权(商汤)、或项目制(世优/Digital Domain)**。影眸 Rodin 封闭商用(API $0.30–0.50/生成)。美国出口管制与中国数据驻留法规也会影响——**商汤尤其曾在美国实体清单**。

### 差异化定位
中国玩家的共同缺口正是本项目的立足点:
1. **无 visionOS 原生**——本项目走原生 Swift/RealityKit 路线,即"第一个 visionOS 原生具身角色"。
2. **多屏基 / 云串流**——中国数字人多服务屏(直播/客服/大屏)或云渲染;本项目坚持**端侧 + 空间渲染**(呼应 MNN-TaoAvatar 的端侧路线,但走 Apple Silicon 原生而非骁龙)。
3. **具身 = 机器人,数字人 = 2D**——中国把"具身AI"等同于人形机器人、把"数字人"等同于屏基 IP;本项目占据二者严格交集(**AR 渲染 + 具身/多模态认知 + 虚拟角色**),这正是 MNN-TaoAvatar 之外几乎无人占据的利基。
4. **场景图即认知 / 物体恒常性**——中国玩家的 avatar 多为"说话的脸 + 动作触发",缺乏本项目设定的多时间尺度环 + 场景图认知 + 记忆=信念+实时核验架构;这是认知层而非渲染层的差异化。

---

## 附录:全部关键源 URL

- 商汤:sensetime.com/cn 、SenseAvatar、SenseMARS Effects、SenseAR Unity tutorial(openar.sensetime.com/sdks)、HuggingFace org、HKEX listing(0020.HK)、SenseME https://www.sensetime.com/cn/product-business?categoryId=79
- 硅基:github.com/duixcom 、Duix-Mobile、Duix-Avatar、Gitee Duix.ai、财联社 IPO、百度百科 司马华鹏;support@duix.com
- 世优:4utech.com、波塔 Web SDK、Baidu Baike、B-round pedaily、36氪 pitchhub
- 魔珐:xmov.ai、xingyun3d.com、corec.cc 发布报道、VR陀螺 1.3 亿美元、cnblogs SDK 教程、CSDN 全栈教程 https://xingyun3d.csdn.net/column/6941161c20df62166aaba626 、量子位 https://www.qbitai.com/2025/10/347284.html 、Vision Pro demo https://mp.ofweek.com/ce/a956714267417 、https://t.cj.sina.cn/articles/view/6012711797/16662b375001018852 、JING https://column.iresearch.cn/b/202306/962263.shtml
- 网易伏羲:fuxi.163.com、百度百科、云信 NIM SDK
- 腾讯:cloud.tencent.com/product/ivh 、cloud.tencent.com/document/product/1662/85737 、cloud.tencent.com/document/product/1240/118295 、HuggingFace https://huggingface.co/tencent 、3d-models.hunyuan.tencent.com 、GitHub https://github.com/Tencent-Hunyuan 、HunyuanVideo-Avatar https://huggingface.co/tencent/HunyuanVideo-Avatar
- 影眸:hyper3d.ai、HuggingFace https://huggingface.co/Deemos 、RodinHD arXiv 2407.06938
- 数字王国:dhginfo@d2.com、547.HK
- 相芯:faceunity.com/avatarxsdk.html、faceunity.com/developer/
- 阿里:help.aliyun.com/zh/avatar/...ios-sdk-local-avatar-only 、MNN https://github.com/alibaba/MNN 、TaoAvatar https://pixelai-team.github.io/TaoAvatar/ 、arXiv 2503.17032、发布文 https://m.thepaper.cn/newsDetail_forward_31039750 、ModelScope https://modelscope.cn/collections/TaoAvatar-68d8a46f2e554a 、LHM https://github.com/aigc3d/LHM 、Live Avatar https://github.com/Alibaba-Quark/LiveAvatar 、arXiv 2512.04677、ModelScope https://www.modelscope.cn/
- 字节:volcengine.com/product/avatar 、LatentSync https://huggingface.co/ByteDance/LatentSync 、OmniHuman https://omnihuman-lab.github.io/v1_5/ 、ByteDance org https://huggingface.co/ByteDance 、ByteDance-Seed https://huggingface.co/ByteDance-Seed
- 美团:https://huggingface.co/meituan-longcat/LongCat-Video-Avatar-1.5 、https://github.com/meituan-longcat/LongCat-Video
- 百度:virtual-man.xfyun.cn(讯飞)
- Unity PolySpatial:https://unity.com/cn/blog/engine-platform/unity-support-for-visionos
- OS-Copilot:https://os-copilot.github.io/
- ShowUI:https://github.com/showlab/ShowUI
- Rokid:https://www.rokid.com/zh-CN/technology
- 策展索引:https://github.com/weihaox/awesome-digital-human

## 附录:未核验项汇总(承诺合作前直接联系各家核)
- 商汤 Unreal SDK 是否存在;商汤 SenseAvatar 跑 AVP 主张(疑似搜索摘要幻觉)。
- 硅基 Duix-Mobile 确切开源 license(LICENSE 未直接检视);Duix-Mobile Apple Silicon(M2)运行时支持。
- 魔珐星云 iOS/Android/Unity/Unreal SDK 可用性(社区声称,官方未直接核);魔珐 Apple 平台端侧(非云)渲染。
- 世优创始人名(纪智辉 跨源确认;"纪铮"/"CUAV" 未能坐实——CUAV 似为无关无人机飞控公司 cuav.net);世优成立年(公司注册 2015 vs 叙述源的"2010")。
- 网易伏羲玉言 HuggingFace 当前开源状态(unconfirmed,实验室转机器人)。
- Deemos HuggingFace(无公开权重模型;RodinHD github 是独立学术产物,非商业 Rodin)。
- 影眸 Rodin 商用转售条款(unconfirmed,读 ToS)。
- WWDC 2024/25/26 是否存在未公开中国 avatar 合作伙伴(搜索不可发现)。
