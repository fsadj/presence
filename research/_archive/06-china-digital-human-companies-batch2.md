# 国内数字人/虚拟人公司 — 第 2 批(5-8)(2026-07/08)

> Vision Pro 开发者视角(原生 Swift,具身认知+AR 渲染)。

## 5) 网易伏羲 NetEase Fuxi
- 网易 AI 研究实验室,2017 成立。负责人**范长杰**(USTC 博士,~200 专利 ~60 顶会论文)。[百度百科](https://baike.baidu.com/item/网易伏羲)
- **当前战略(重要)**:已**大幅转向具身AI/工业机器人**(网易灵动 Lingdong:挖掘/装载机器人)与众包/agent。数字人工作仍在但不再是头条。产品见 fuxi.163.com:网易灵动(旗舰)、绘梦天工(30+ 游戏资产,含逆水寒/永劫无间)、有灵众包、有灵智能体+AOP(Agent-Oriented Programming)、妙启AI对话。
- **签名模型/资产**:玉言(~11B 中文 LLM,deep encoder+shallow decoder,CLUE 登顶);玉知(VLM);丹青(图像生成);**易生诸相**(角色扮演/character LLM 底座);**4D 面部扫描**(逆水寒叶雪青);**智能捏脸**(照片→3D 面部参数,永劫无间/逆水寒);**智能动作/表情捕捉**(普通摄像头无标记动捕/表情捕捉);**语音驱动表情动画**(GDC 2021);玉言驱动逆水寒 AI NPC(并邀通义/文心/abab/月之暗面/豆包入园);获**国家科技进步奖**(数字人)。
- **给 Vision Pro 带来什么**:认知(玉言角色扮演 LLM);avatar 创建(智能捏脸、4D 面扫、语音驱动面部动画——对 AR avatar 直接有用);动捕(仅摄像头动捕/表情捕捉);实时交付见下云信。
- **访问/SDK/iOS-visionOS**:伏羲自身能力是 **B2B/合作制**,非交钥匙公开 SDK;AOP 是集成框架。**iOS 可访问的具体路径是网易云信(WXIN)**:实时对话 AI agent 驱动 AI 数字人(音视频),**跨平台 SDK 含 iOS**(NIM SDK v10.8.30,流式输出)。无原生 visionOS SDK。玉言权重 2022-23 曾部分上 HuggingFace(NetEase-Fuxi org),当前开源状态**unconfirmed**(已转机器人)。
- **结论**:认知(LLM)+面部动画+摄像头动捕强,但多内部/游戏 B2B。原生 iOS 唯一打包路径是云信实时 AI-agent + 数字人 SDK。

## 6) Sogou 数字人 / Tencent 数字人 — 关键状态更新
- **历史**:搜狗分身是搜狗核心 AI,2018 首发**全球首个 AI 合成主播**(与新华社),克隆主播邱浩/"雅妮",2019 全球首个站立 AI anchor(WaveRNN 语音+唇/表情合成)。王小川时任搜狗 CEO。
- **收购与状态**:腾讯 2021 私有化收购搜狗。avatar 技术被吸收、跨多个腾讯业务重组:
  | 产品 | 状态(2026)|
  |---|---|
  | 搜狗分身 | 已吸收,非独立品牌 |
  | **腾讯智影 Zhiying** | **已死**。2025-04-15 停注册,2025-06-30 停登录删数据,2025-07-01 起"升级维护中",官方账号 9 月注销。腾讯未给官方理由。 |
  | **腾讯云智能数智人 IVH** | **活跃——真正继任者**。cloud.tencent.com/product/ivh |
  | 腾讯妙思 Miaosi | 活跃(营销 AIGC)|
  | "奇妙数字人"(3000+ avatar,WAIC 2025)| 活跃 |
- **大信号**:腾讯另在**计划封禁直播电商 AI 数字人主播**——消费级数字人战略收缩,企业级 IVH 继续。
- **活跃 SDK:腾讯云 IVH(对 Vision Pro 最重要)**:小样本克隆(3 分钟视频+100 句语音→~24 小时克隆);集成 H5/Android/**iOS**,渲染引擎 **WebGL/Unity/Unreal**,协议 **RTMP/WebRTC**;**客户端渲染 SDK**(Android+iOS,端侧 lip-sync/渲染);云渲染 H5 SDK v5.5.0。
- **结论**:搜狗血脉**仅存于腾讯云 IVH**。腾讯智影已没。Vision Pro 开发者的唯一腾讯路径(有真 iOS/Unity/UE SDK)是 IVH,但注意腾讯整体对消费数字人降温。

## 7) 影眸科技 Hyper3D / Deemos — "浙大 spinoff" 是错的
- 影眸科技(上海)有限公司 / Deemos,2020 成立(注册 2020-06-24)。
- **⚠️ 纠错:非浙江大学 spinoff**。影眸在**上海科技大学(ShanghaiTech)**孵化——MARS Lab / Visual & Data Intelligence Center。创始人**吴迪(CEO)**、**张启煊(CTO,1999 生,上科大 2018 本→2022 硕)**、张龙文、曾初啸。2020 获上科大 IP 授权。无公开来源指向浙大。
- **投资**:凯辉、上海国投先导(领投)、**字节跳动**等;光源资本 FA。ARR 数千万美元。入选**英伟达黄仁勋 CES 主旨** 3D 资产工作流——唯一入选的 3D 生成创业公司。
- **签名产品**(hyper3d.ai):**Rodin**(文/图生 3D 旗舰;Gen-1/Gen-1.5/Gen-2(BANG)/**Gen-2.5**——全球首个**千万级多边形** 3D 生成,"think-then-generate" LLM 式推理);**Hyper3D** 平台;**ChatAvatar**(文/图→3D avatar/脸,2023 曾在 HF Space);OmniCraft、AI 纹理/HDRI 生成、3D mesh 编辑器、3D 模型搜索引擎。
- **Vision Pro 相关度**:**3D 资产/avatar 生成最佳**。站点明确列 VR/AR、角色设计、游戏、动画、元宇宙。输出生产级 mesh+UV+纹理+PBR。
- **插件/SDK/定价——DCC 覆盖强**:原生插件 **Blender/Unity/Unreal/Godot/Maya/3DS Max/ComfyUI**;REST API。定价:免费 1000 credit/月(个人);Pro ~$20/月;API ≈ **$0.30–0.50/生成**。亦在 fal.ai($0.40)、wavespeed($0.30)。**无原生 iOS/visionOS SDK**——是云生成服务 + DCC 插件,非运行时 avatar 动画引擎。Vision Pro 用法:服务端调 API 生成 mesh/纹理,再导入/动画(如 MetaHuman、ARKit blendshape);Unity 插件可喂 Unity→visionOS 管线。
- **HuggingFace**:org `Deemos` 与 `DEEMOSTech` 均**无公开权重模型**(仅研究论文:*Strips as Tokens*、*Kinematify*)。旧 ChatAvatar HF Space(2023)已失效。**RodinHD**(arXiv 2407.06938,GitHub RodinHD)是**学术研究论文,非商业 Rodin 模型**,研究用途条款。
- **Rodin 授权**:**封闭/商用**。生产 Rodin 无开源权重。商用需 API 订阅或企业授权(具体转售条款 unconfirmed,读 ToS)。
- **结论**:生成高保真 3D avatar/资产出色,有原生 Unity/Unreal 插件。非运行时动画或 mocap 方案,无原生 iOS/visionOS SDK。"浙大 spinoff"前提错——是**上海科大**。

## 8) Digital Domain 数字王国
- Digital Domain Holdings,好莱坞 VFX(1993,James Cameron 等),母公司**港股 547.HK**。大中华首家独立 VFX 工作室。办公室:LA/Vancouver/Montreal/**京沪深港台北**/Hyderabad。中国实体:數字王國朝霆(上海)。
- **领导层澄清**:**谢安 = Daniel Seah(SEAH Ang)**——同一人。2014 入任 CEO,后升**主席**,主导 MBO。(中文媒体用谢安,英文用 Daniel Seah——非两人。)
- 2018 收购 **3Glasses**(深圳 VR)。
- **数字人工作**:邓丽君全息(标志性,2013 周杰伦演唱会,后独立演唱会,AI 驱动版讨论过);龚俊数字人;**"Douglas"(2020)**——自主实时拟真数字人(POC,Doug Roble 肖像),ML 驱动:语言处理/表情/视觉追踪/换脸/语音复制(~30 分钟音频/10 分钟视频);**"Zoey"(2022)**——最先进自主数字人,Douglas 继任;早期 DigiDoug 实时角色。技术栈 **Unreal Engine + ML + 深度学习 + 虚拟制作**。拥有 NUKE 合成软件(2017 奥斯卡科技奖)。2023 在**香港科学园建 2600 万美元虚拟人研发中心**。
- **给 Vision Pro 带来什么**:VFX 级拟真数字人渲染 + 全息专长(最"好莱坞级"选项);自主数字人 know-how(Douglas/Zoey:感知/表情/对话——概念上接近具身AI角色);VFX/mocap 血统(表演捕捉、neural rendering)。
- **访问/SDK/授权——catch**:**无公开 SDK、无开源、无 API 产品**。接触是**合作/联合开发/定制**。Douglas 联系 dhginfo@d2.com。无 iOS/visionOS SDK。基于 Unreal——理论上可经 Unreal visionOS 支持 target,但 DD 自己不发此集成。邀请"投资、合作与客户工作"围绕 Douglas。
- **结论**:高端定制 VFX/拟真数字人伙伴,非即插 SDK。仅在预算与时间允许联合开发做超高保真 avatar 时相关。无交钥匙开发者产品。

---

## 给 Vision Pro 开发者的横切总结(本批 4 家)
| 需求 | 本批最佳 |
|---|---|
| 3D avatar/资产**生成**(文/图→mesh)| **影眸/Hyper3D Rodin**(API + Unity/Unreal 插件;无 visionOS SDK——服务端调)|
| 高保真面部动画/动捕 | **网易伏羲**(语音驱动面部动画、摄像头动捕、智能捏脸)——B2B/合作;iOS 运行时经**网易云信** AI-agent SDK |
| 实时 iOS 数字人 SDK(交钥匙)| **腾讯云 IVH**(iOS SDK、Unity/UE、WebRTC;唯一活跃腾讯路径;腾讯智影已死)|
| 拟真 VFX/全息(定制)| **Digital Domain**(Douglas/Zoey;仅合作,无 SDK)|
| 具身AI认知(LLM)| 网易伏羲玉言角色模型;(腾讯混元另述)|

## 纠错/未确认
- 影眸 = **上海科大**孵化,**非浙大**(高置信,多源)。
- Daniel Seah **= 谢安**(同一人;解决 CEO 表面矛盾)。
- 玉言 HuggingFace 当前开源:**unconfirmed**(实验室转机器人)。
- 腾讯智影:**确认已死**(2025-07)。
- Deemos HuggingFace:**无公开权重模型**(Deemos 与 DEEMOSTech org 均 0 模型);RodinHD github 是独立学术产物,非商业 Rodin。
