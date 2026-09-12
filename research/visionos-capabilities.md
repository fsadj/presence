# visionOS 核心能力:企业摄像头 + 锚点持久化

> 合并文档:Vision Pro 让虚拟角色"真正看见真实世界"和"记住/回到同一真实位置"所需的两大底层 visionOS 能力。全部对 Apple 一手文档 / WWDC24/25/26 / 社区实测核验;无臆造 API。

## 来源映射

本文档合并自以下原调研文件:

| 原文件号 | 路径 | 覆盖主题 |
|---|---|---|
| 01 | `/Users/brazion/project/vr/research/_archive/01-visionos-enterprise-camera-access.md` | 企业主摄像头访问门控、`CameraFrameProvider`、Foveated Streaming、分发约束、有无摄像头访问对比 |
| 04 | `/Users/brazion/project/vr/research/_archive/04-visionos-anchor-persistence.md` | `WorldAnchor`/`WorldTrackingProvider` 自动持久化、`ARWorldMap` 缺失、workaround、跨设备/会话限制 |

> 两份原文件的重叠点(已合并到单处,不重复):① main-camera entitlement `com.apple.developer.arkit.main-camera-access.allow` 的定义与"帧不含深度"——放在 A 节,B 节 §6 只引用结论;② 完整 ARKit data provider 列表(含 `CameraFrameProvider`/`CameraRegionProvider`/`SharedCoordinateSpaceProvider`)——A 节列企业 API 目录,B 节列持久化相关 provider。

---

## A. 企业摄像头权限(来自原 01)

### A.0 一句话结论

Vision Pro 的原始 RGB 摄像头访问是**企业级门控**的,且至少到 visionOS 27(2026 秋 GA)都会保持。但门控**不是** Apple Developer Enterprise Program 本身——`com.apple.developer.arkit.main-camera-access.allow` 这个 entitlement 单独申请,**$99 普通开发者账号 OR $299 企业计划的 Account Holder 都能提交**,而且 Apple 明确提供 **"Development Only" 访问**,让你在单台注册的 Vision Pro 上原型,无需完整企业分发。**立刻申请 "Development Only"(不额外花钱,社区反馈审批几周~几个月)**,同时把感知栈架构成"摄像头路径"与"纯网格回退"在同一个接口后可互换。另外,RTX Pro 6000 PC 有个新选项:**visionOS 27 的 Foveated Streaming 框架**(基于 NVIDIA CloudXR),是 Apple 首条把 OpenXR PC 内容串流到 Vision Pro 的官方路径。

### A.1 默认状态——第三方 app 拿不到什么

开箱即用,第三方 visionOS app 被**沙箱隔离,拿不到前向 passthrough 摄像头的原始 RGB 像素**(Apple 理由:隐私)。无需任何特殊 entitlement 就能拿到的:
- `SceneReconstruction` provider——带纹理的 `SceneMesh`(只有几何,**无语义标签**,低频)
- `HandTracking` provider——骨骼关节变换(~每手 26 关节)
- `WorldTracking` provider——设备位姿、世界锚点、`RoomAnchor`/`PlaneAnchor`
- `ObjectTracking` provider——追踪最多 N 个用户训练的参考物体(用 Create ML 扫描);visionOS 27 增强了高频 + Metric Space Pose
- `BarcodeDetectionProvider`——其实是企业级(见 A.4)
- 完整 Apple **Vision 框架**——但只能跑在你能拿到的画面上。没摄像头权限时,只能喂静态图 / `CIImage` / 程序生成图

### A.2 AVCaptureSession / AVFoundation 常见误区

AVFoundation 的采集 API **在 visionOS 上存在,但只用于外接/连接设备**(UVC USB 摄像头、医疗成像设备等),**不是**头显自带 passthrough 摄像头阵列。visionOS 27 起,通过 Developer Strap 接 UVC 设备**对所有开发者开放**(visionOS 2 时是企业级)。所以 `AVCaptureSession` 拿不到头显自带摄像头,只能拿到插在 Developer Strap 上的 USB 摄像头。

### A.3 门控路径:`com.apple.developer.arkit.main-camera-access.allow`

这是 Apple 唯一认可的、从头显自带前向摄像头读像素的方式。精确解锁内容:
- **API**:`CameraFrameProvider`(ARKit `DataProvider`),跑在 `ARKitSession` 上。通过 `cameraFrameProvider.cameraFrameUpdates(for: someFormat)` 收帧
- **逐样本访问**:`cameraFrame.sample(for: .left)` 和 `.right` 返回 `CameraFrame.Sample`,含 `CVPixelBuffer` + 相机内外参
- **分辨率/格式**:运行时通过 `CameraVideoFormat.supportedVideoFormats(for: .main, cameraPositions: [.left]/[.right]/[.both])` 枚举。社区实测 visionOS 26 格式:**864×704,YUV 4:2:0(`kCVPixelFormatType_420YpCbCr8BiPlanarFullRange`/"420f"),`stereoCorrected` 校正,最高 ~60 fps**
- **visionOS 2(WWDC24)**:仅左摄像头
- **visionOS 26(WWDC25,2025-06)**:加右摄像头 + 立体处理模式;`CameraFrameProvider.CameraRectification`;摄像头画面在 Immersive Space 和 Shared Space 都可用。**这让端侧立体深度变得可行**
- **visionOS 27(WWDC26,现已 dev beta,2026 秋 GA)**:未移除企业门控。摄像头访问仍是企业级
- **visionOS 26/27 新替代——`CameraRegionProvider` + `CameraRegionView`**:不是拿整幅宽视场画面,而是在 3D 空间里放一个给定尺寸(米)的虚拟"窗口"锚点,只接收该空间区域的、已稳定、可增强对比度/鲜艳度的 `pixelBuffer`。同样需要 entitlement。适合只需感知眼前特定工件/仪表/人的场景——带宽比全帧低

### A.4 该 entitlement **不**给你的东西

重要——vision.engineer(Tony Morales,有该权限)原话:*"What do you get with visionOS's Main Camera Access? An image. That's it. No depth data baked into the frame. No segmented anything."*
- **无 LiDAR 深度**——Vision Pro 的深度来自立体匹配,不是专用深度传感器流
- **无逐帧语义分割**——要自己算(Vision 框架、Core ML)
- **无 iOS 上 ARKit `ARFrame` 那种丰富度**——visionOS 的 CameraFrame 只有裸像素 + 内外参

### A.5 麦克风访问

visionOS 麦克风采集用标准 iOS/iPadOS 音频栈(**无特殊企业 entitlement**),由普通用户授权隐私权限(`NSMicrophoneUsageDescription`)门控。

### A.6 完整企业 API 目录(2026 年中)

Apple 官方文档列出 visionOS 企业 API 现包括:

**第 1 类——增强传感器访问:**
1. `com.apple.developer.arkit.main-camera-access.allow`——主摄像头像素(visionOS 26+ 左右立体)
2. `com.apple.developer.arkit.camera-region.allow`——`CameraRegionProvider`/`CameraRegionView` 空间区域
3. `com.apple.developer.arkit.shared-coordinate-space.allow`——`SharedCoordinateSpaceProvider`(visionOS 26 新;多台头显在同一房间通过你自己的网络传输对齐坐标)
4. `com.apple.developer.screen-capture.include-passthrough`——穿戴者所见复合画面(窗口+passthrough)经 ScreenCaptureKit;需用户每次会话点"开始广播"
5. `com.apple.developer.arkit.barcode-detection.allow`——用 `BarcodeDetectionProvider` 做空间条码/QR 扫描

**第 2 类——平台控制:**
- `com.apple.developer.protected-content`——`contentCaptureProtected` SwiftUI 修饰符(visionOS 26 新)
- `com.apple.developer.arkit.object-tracking-parameter-adjustment.allow`——可调物体追踪
- `com.apple.developer.app-compute-category`——提升性能余量(Increased Performance Headroom)
- `com.apple.developer.window-body-follow`——窗口跟随模式(visionOS 26 新)
- `com.apple.developer.arkit.visual-fidelity.allow`——贴合/视场监测

**visionOS 26 从企业级提升为对所有开发者免费**(无需 entitlement):通过 Developer Strap 接 UVC 外设,以及 Apple Neural Engine 访问(`MLModel.availableComputeDevices` 含 `.neuralEngine`)。

### A.7 企业路径:是什么 & 时间线

#### A.7.1 关键区分:企业 entitlement vs 企业分发

Apple 官方文档原话:*"the Account Holder of your Apple Developer Program and/or Apple Developer Enterprise Program can submit an entitlement request."* 这个 "and/or" 是关键——**摄像头 entitlement 与企业分发计划解耦**。只用 **$99/年 Apple Developer Program** 会员就能申请摄像头访问。

Apple 文档还明确提供两级授权:
1. **"Development Only" 访问**——用开发配置文件在自己的注册测试设备上 build & run。足够在单台 Vision Pro 上原型
2. **分发访问**——要部署到其他设备,要么 (a) 在 **Apple Developer Enterprise Program**($299/年)下作为专有内部 app 分发,要么 (b) 通过 **Apple Business Manager 走 custom app**(可用 $99 计划;设备需被客户组织 MDM 注册/拥有)

**申请表单**(Apple 文档链接,也可直达):`https://developer.apple.com/contact/request/enterprise-apis-visionos/`。提交时按 API 逐一说明业务用例。

#### A.7.2 资格要求(Apple 文档原话)
- *"Be for use in a business setting only"*
- *"Meet specific criteria associated with usage for each API"*
- WWDC25 Session 223 澄清:*"designed for proprietary, in-house apps, developed by your organization for your employees, or for custom apps you build for another business to distribute internally."*

**entitlement 本身没有公布最低公司规模/员工数/营收门槛**。(100+ 员工是**企业分发计划**的要求,不是摄像头 entitlement 的。)

#### A.7.3 Apple Developer Enterprise Program(仅在需要内部分发时)
- **费用**:$299/年(标准计划 $99)。同一实体不能同时持有个人和企业会员
- **要求**:D-U-N-S 号码(法人实体,经 D&B ~5 个工作日)、认可的法人实体(不接受 DBA/分支机构)、**100+ 员工**、仅限内部分发、单独身份验证(含验证电话)
- **时间线**(社区反馈,Apple 无 SLA):端到端通常 **2–4+ 周**;验证电话约 7–13 天进。D-U-N-S 记录与注册信息不匹配造成最长延迟
- **公开分发**:**不允许**。企业分发的 app 不能上消费级 App Store

#### A.7.4 Main Camera entitlement 审批时间线(真正的门槛)

Apple **无官方 SLA**。社区证据:
- vision.engineer(Tony Morales):2024 年 7 月获批 Main Camera Access,用于本地原型
- Griffin Hurt:visionOS 26 发布后不久发布了立体深度 demo
- 多个 Reddit/论坛报告:有时限的开发许可(~6 周),续期通过新的 `VisionEntitlementServices` 框架(visionOS 26+)管理,暴露 `licenseStatus` 和 `expirationDate`
- visionOS 26 带来了**空中自动许可续期推送**和从 Apple Developer 账号直接下载许可文件

按**几周到几个月**规划端到端。提交强而具体的业务理由(面向内部培训/辅助的 AI 角色符合;"我想做消费 app"不符合)。

#### A.7.5 已知拒绝/限制

Apple 按**每个 app** 审核分发授权。已知痛点:
- 模糊的消费产品定位会被拒
- Apple 认为隐私侵入式的摄像头数据处理过不了审核
- 许可文件**有时限**,必须续期;若 Apple 拒绝续期,你的 app 会静默停止收帧
- 需要 `NSMainCameraUsageDescription` Info.plist 键(visionOS 2 时叫 `NSEnterpriseMCAMUsageDescription`——未文档化的改名坑过社区)

#### A.7.6 "申请企业 API" 实际映射到什么

你说的中文直译是"申请企业 API"。在 Apple 实际机制里,这是**通过上面表单的 entitlement 申请**——不一定是 Enterprise Program。对单设备原型、可能小团队最干净的路径:
1. Apple Developer Program($99)→ 2. 提交 entitlement 申请表 → 3. 收到 "Development Only" 许可文件 → 4. 本地 build & 测试。**在真正需要机群分发前,完全跳过 Enterprise Program**。

### A.8 今天就能在本地(单设备、Xcode)测什么

无需任何 entitlement,今天,在 M1 Max Mac + 一台 Vision Pro + $99 开发者账号上,就能 build & run:
- 完整 **ARKit 感知栈**减去摄像头像素:`SceneReconstruction` mesh、`HandTracking`(骨骼,visionOS 2+ 低延迟预测关节)、`WorldTracking`、`RoomTracking`(visionOS 2+)、`ObjectTracking`(用自己训练的 Create ML 参考物体;基线无需企业 entitlement,entitlement 只解锁可调参数)
- 完整 **Vision 框架**管线跑在 `CIImage`/`CGImage`/`CVPixelBuffer` 上:物体检测(`VNDetectRectanglesRequest`、带自己 Core ML 模型的 `VNCoreMLRequest`)、人/脸/手体姿(`VNDetectHumanBodyPoseRequest`、3D 体姿)、人像分割、文字识别、条码扫描(注:空间条码锚点位置是企业级,但 Vision 的非空间条码识别不是)
- visionOS 27 新的 **"Visual Intelligence"** 系统功能(Siri 推理用户正看的东西)**不需要你的 app 有摄像头权限**——Apple 在系统侧处理感知,通过标准 Siri/App Intents 暴露结果。某些用例下可能是可行替代路径
- 所有 **RealityKit** 渲染、**SwiftUI** 空间 UI、**Core ML** 端侧推理(Neural Engine 自 visionOS 26 起免费),以及任何不依赖头显摄像头像素的 RTX Pro 6000 工作

拿到 "Development Only" entitlement(几周~月等待)后,增加:
- Apple 的 **"Accessing the Main Camera" 示例工程**(可从 Apple 文档克隆)——权威参考。加入 `Enterprise.license` 文件、设 `NSMainCameraUsageDescription`、实例化 `CameraFrameProvider`、跑在 `ARKitSession`、迭代 `cameraFrameUpdates(for: someFormat)`、从 `sample(for: .left)` 和(visionOS 26+)`sample(for: .right)` 取 `pixelBuffer`
- 对**实时帧**跑 Vision 框架请求——这是你的虚拟角色真正"看见"的时刻
- 经局域网把帧串流到 RTX Pro 6000——社区项目(如 YOLOv11 + Main Camera Access demo,Reddit r/VisionPro 2025)证明此模式可行

本地开发坑(社区发现,Apple 文档没有):
- **为 visionOS 26/27 显式安装 Metal 工具链**——有时默认不装,导致静默运行时失败
- **`CVPixelBuffer` 在 visionOS 26 被弃用**改用 `CVBuffer`,但 Core ML 还没更新接受 `CVBuffer`——暂时还得用 `CVPixelBuffer`
- **Info.plist 键改名**:用 `NSMainCameraUsageDescription`,不是旧的 `NSEnterpriseMCAMUsageDescription`
- **Core ML 混合精度模型**:Vision Pro 的 Neural Engine 对 FP32 中间张量挑剔("`error: ANE cannot handle intermediate tensor type fp32`")。可能要强制 `.cpuAndGPU` 计算单元,或把全 FP16 量化跑通才能落到 ANE

### A.9 摄像头访问解锁了什么(架构层面)

#### A.9.1 端侧,实时帧
- **Vision 框架跑实时流**:`VNDetectRectanglesRequest`、`VNCoreMLRequest`、`VNDetectHumanBodyPose3DRequest`、`VNDetectFaceLandmarksRequest`、人像分割掩码。在 M5/R1 SoC 上端侧跑,Neural Engine 现已免费
- **立体深度**经左右对(visionOS 26+)。社区参考:Griffin Hurt 的 RAFT-Stereo-on-Vision-Pro 实现达 ~9 fps 视差@512×512、GPU 上 ~135 ms 推理(Neural Engine 被 FP32 权重挡住——可修)。用 `depth = focal_length × baseline / disparity` 逐帧
- **语义 3D 记忆**:融合逐帧深度 + Vision 检测的 2D 框/掩码 + 一直可用的 `SceneMesh` 和 `WorldTracking` 位姿,成连贯的语义点云/体素网格,跨会话持久。这是你的虚拟角色推理的基底
- **面向用户的感知**:用户表情(部分——Vision Pro 的眼动摄像头不暴露,但前向外部摄像头在反射面时能见下脸;更实用的是经用户许可的 `EyeTracking` provider 拿注视、`HandTracking` 拿手势、麦克风拿语音)

#### A.9.2 离侧,串流到 RTX Pro 6000 96GB PC

两个互补模式:
1. **DIY 帧串流(今天可用)**:`CVPixelBuffer` → 编码(JPEG/H.264)→ 经 LAN 发送(NWConnection/gRPC/自定义)→ PC 跑开放词表 VLM(Qwen2.5-VL、InternVL、LLaVA-NeXT)→ 返回 JSON 场景描述/物体标签回头显。864×704 YUV@30fps 带宽不大,千兆 LAN 轻松扛
2. **Apple 官方路径(visionOS 27,2026 秋 GA)——Foveated Streaming**:基于 **NVIDIA CloudXR** 的一方框架,把 OpenXR 内容从 Windows PC/云串流到 Vision Pro,带眼动追踪的注视点视频压缩(只在用户看的方向全质量)。集成 ARKit、SwiftUI、RealityKit。Demo 含 Autodesk VRED、X-Plane 12、iRacing。**对这套硬件是最重要的进展**——重 VLM/渲染放 RTX Pro 6000,低摩擦串流到 Vision Pro,无需移植。注意:Foveated Streaming 面向 OpenXR 渲染;是否能把摄像头帧**回传** PC 做推理,要等 visionOS 27 上市对 API 核实。即便不能直接回传,它也解决了渲染侧一大集成难题

#### A.9.3 有无摄像头访问对比

| 能力 | 无 entitlement | 有 main-camera entitlement |
|---|---|---|
| 场景几何(`SceneMesh`) | 是 | 是 |
| 手部/眼动/世界位姿/房间锚点追踪 | 是 | 是 |
| 追踪用户训练物体 | 是(基线) | 是(可调) |
| Apple Vision 框架跑实时 RGB | 否(仅静态图) | **是** |
| 开放词表 VLM 跑实时流(PC) | 否 | **是** |
| 逐像素语义分割 | 否 | **是** |
| 稠密立体深度图 | 否 | **是**(visionOS 26+ 立体对) |
| 真正带语义标签的 3D 记忆 | 粗(网格+稀疏锚点) | **稠密、帧率级** |
| 识别任意没见过的物体 | 否 | **是**(经 VLM) |

无 entitlement 栈下角色也能反应(知道你手在哪、墙/桌在哪、训练过的物体在哪),但**看不到眼前任意东西**。摄像头访问是"空间感知"与"视觉感知"的分水岭。

### A.10 约束 & 商业权衡

#### A.10.1 分发天花板
- 用**任何**企业 API 的 app **不能上消费级 App Store**。Apple 文档确认:*"Enterprise APIs for visionOS are eligible for business use only..."*
- 现实分发渠道:(a) 企业内部(你公司的员工,$299/年 Enterprise Program,100+ 员工),(b) 经 Apple Business Manager 走 custom app 到客户组织的 MDM 注册设备(可用 $99 计划;B2B)
- Apple **未公布**任何把原始摄像头访问扩展到消费 App Store 的时间表或意图。visionOS 26 和 27 都保留了企业门控。平台趋势是**企业范围内扩 API**(更多 API、更灵活、立体、camera region)而**受众不变**(仅商业)。按"消费级摄像头访问不会按可预测时间表到来"来规划

#### A.10.2 消费品影响

这是项目最重要的商业事实:**若终态是消费级 Vision Pro app,原始摄像头访问目前是死路。** 现实战略选项:
1. **保持 B2B/企业**:把 AI 角色作为内部培训/辅助产品卖给企业(制造、医疗、现场服务、教育)。设备组织拥有、MDM 注册、经 Apple Business Manager 分发。符合 Apple 规则。**这是唯一能用你完整感知栈的路径**
2. **转向系统功能**:靠 visionOS 27 的 **Visual Intelligence**(Apple 系统侧感知,经 App Intents/Siri 把结果给 app,不给摄像头帧)+ 一直可用的 `SceneMesh`/`HandTracking`/`ObjectTracking`
3. **纯网格"感知"路径**:可上 App Store 的消费 app,用场景几何 + 手部追踪 + 训练物体追踪。角色知道房间和用户身体,但对没显式训练过的东西"瞎"。重大产品约束但合法
4. **等/游说 Apple**:经 Feedback Assistant 和 entitlement 申请提增强请求;Apple 偶尔扩权限(UVC 和 Neural Engine 在 visionOS 26 提升出企业级)。不要拿路线图赌这个

#### A.10.3 逐设备配置、审核、吊销
- 用企业 API 的每个 app 被 Apple **逐 app 审核**后才给分发
- **许可文件有到期日**,必须续期;visionOS 26+ 续期空中推送。运行时用 `VisionEntitlementServices.EnterpriseLicenseDetails.shared.licenseStatus` 和 `.isApproved(for: .mainCameraAccess)` 验证状态
- Apple 可吊销。把 app 做成优雅降级:检查许可状态,无效/过期时回退纯网格感知
- Enterprise Program 下逐设备安装需设备 UDID 在配置文件;Custom Apps(Apple Business Manager)需设备被客户组织 MDM 注册

### A.11 工程意义(对"角色看你写代码"的本项目)

- **Foveated Streaming 把帧喂工作站(感知管线)**:visionOS 27 GA 后,Apple 自建 NVIDIA CloudXR 管线解决"PC 渲染串流回头显";同时自定义 LAN 通道把摄像头 `CVPixelBuffer` 送 RTX Pro 6000 跑 VLM。两路互补。
- **外部摄像头看不见用户自己的脸**:Vision Pro 的眼动摄像头不暴露,前向摄像头朝外。用户表情/情绪只能靠**语音(麦克风)+ 注视(`EyeTracking`)+ 手势(`HandTracking`)** 三路融合推断,不能直接拍脸。

### A.12 建议(立刻可执行步骤)

#### A.12.1 立刻申请
1. **今天**:若还没有,以**组织**身份(非个人——对后续 Enterprise/Custom Apps 路径更顺)加入 **$99 Apple Developer Program**。核实 D-U-N-S 信息精确匹配
2. **本周**:到 `https://developer.apple.com/contact/request/enterprise-apis-visionos/` 提交 entitlement 申请,同时申请 `com.apple.developer.arkit.main-camera-access.allow` **和** `com.apple.developer.arkit.camera-region.allow`,框定为**"Development Only"**,用于原型一个面向内部/B2B 空间辅助的 AI 角色。业务问题写具体(培训、现场服务、无障碍)。**不要**框定为消费 app。一次申请多个相关 entitlement 是正常的
3. **并行**:若预判要给 100+ 员工机群分发,现在就启动 **Enterprise Program($299)注册**——验证流程是最长的一环。若团队小或走 B2B 经客户 MDM,跳过 Enterprise,规划 Apple Business Manager custom app
4. **等待期间(几周~月)**:今天就开始建**纯网格回退版**感知栈。无需任何 entitlement,需要时能上 App Store

#### A.12.2 现在就按"摄像头/网格可互换"架构

把感知层设计成单一 Swift 协议(`PerceptionSource`),两个实现:
- `MeshPerceptionSource`——包 `SceneReconstruction`、`WorldTracking`、`HandTracking`、`ObjectTracking`、`EyeTracking`。始终可用。返回 `SceneModel`(mesh + 锚点 + 追踪物体 + 用户状态)
- `CameraPerceptionSource`——包 `CameraFrameProvider`(可选 `CameraRegionProvider`),逐帧跑 Vision 框架请求,可选地把帧串流到 RTX Pro 6000 PC 做 VLM 推理,把结果融合回同一个 `SceneModel`,丰富语义标签和稠密深度

角色逻辑**只消费 `SceneModel`,绝不碰原始摄像头帧**——所以在任一后端上行为一致。启动时用 `VisionEntitlementServices` 选更富的源。这是应对 entitlement 延迟/被拒、许可过期、Apple 吊销的保险,**还能从一个代码库同时出消费安全的 App Store 版(纯网格)和企业版(完整感知)**。

#### A.12.3 审慎规划 PC 集成(两阶段)
- **阶段 1(今天可用)**:自定义 LAN 串流。`CVPixelBuffer` → 编码(JPEG/AVAssetWriter H.264)→ `NWConnection` 到 PC → VLM 推理(RTX Pro 6000 上 Qwen2.5-VL/InternVL)→ JSON 结果回。预算 ~30–60 ms LAN RTT + VLM 推理。先端到端验证再投入下一步
- **阶段 2(visionOS 27 GA,2026 秋)**:评估 **Foveated Streaming** 用于渲染路径(Apple 自建 NVIDIA CloudXR 管线)。它不会消除自定义摄像头帧回传通道,但用一方支持解决更重的"PC 渲染内容串流回头显"问题,带眼动注视点压缩、ARKit/SwiftUI/RealityKit 集成。值得现在对着 visionOS 27 dev beta 原型

#### A.12.4 校验产品定位

花钱上 Enterprise Program 前,明确决定:这是内部工具、卖给其他公司的 B2B 产品,还是消费品?只有前两个兼容 main-camera 访问。若答案是"终将消费化",现在就开始谈:体验有多少能在纯网格/Visual Intelligence 栈上存活,因为那才是 App Store 允许的。

---

## B. 锚点持久化(来自原 04)

### B.0 一句话结论

visionOS **有**锚点持久化机制——`WorldTrackingProvider` 下的 `WorldAnchor`,且是**自动、系统托管**:你在一个 app run 里加的锚点,当设备回到同一真实位置时,会在未来 run 里自动回来。这是"AR 角色回到老地方"在**同一设备**上的受支持答案。

visionOS **没有**的是 iOS 的 `ARWorldMap` 等价物:**没有可保存/导出的地图 blob、没有 `getCurrentWorldMap`、没有 `setWorldMap`、没有"对文件重定位"API**。锚点"被困"在创建设备上(Apple 论坛/社区)。所有持久化都是不透明、端侧、单用户、单设备。

### B.1 ARKit on visionOS — 无 `ARWorldMap`/世界地图持久化

**确认缺失。** Apple 概览文档 "ARKit in visionOS" 枚举了平台所有 ARKit data provider,完整列表:`PlaneDetectionProvider`、`WorldTrackingProvider`、`HandTrackingProvider`、`SceneReconstructionProvider`、`ImageTrackingProvider`、`ObjectTrackingProvider`、`BarcodeDetectionProvider`、`RoomTrackingProvider`、`EnvironmentLightEstimationProvider`、`CameraFrameProvider`、`AccessoryTrackingProvider`、`SharedCoordinateSpaceProvider`、`StereoPropertiesProvider`、`CameraRegionProvider`、`VisualFidelityProvider`。**没有 `WorldMapProvider`、没有 `WorldMap`、没有 `MapSession`。** https://developer.apple.com/documentation/arkit/arkit-in-visionos

iOS 文档 "Saving and loading world data"(文档化 `ARWorldMap`、`ARSession.getCurrentWorldMap`、`setWorldMap` 重定位)是 **iOS/iPadOS 专属**。https://developer.apple.com/documentation/arkit/saving-and-loading-world-data

迁移指南 "Bringing your ARKit app to visionOS" 明确列 iOS 功能中**"visionOS 无等价物"**的,含"用经纬度放置锚点的 Geotracking"。https://developer.apple.com/documentation/visionos/bringing-your-arkit-app-to-visionos

### B.2 RealityKit on visionOS — 无 `AnchorEntity` 持久化 API

`AnchorEntity` 在 visionOS 存在且遵循 `HasAnchoring`,但其参考页**没有 save/load/serialize/persist API**——只有 anchoring targeting。https://developer.apple.com/documentation/realitykit/anchorentity

**【纠错/勘误】对常见社区说法的重要纠正**:"`ARAnchorManager`" **不是 Apple API**。它是 Unity XR AR Foundation 插件(`UnityEngine.XR.ARFoundation`)的一部分。论坛/YouTube 称"ARAnchorManager 让锚点自动持久"描述的是 Unity 对 Apple `WorldAnchor` 的包装——**非 Apple 原生 API**,不应作为 Apple 持久化 API 引用。

RealityKit 在 visionOS 经 `SpatialTrackingSession`(visionOS 2.0+)或直接 ARKit `WorldTrackingProvider` 暴露世界锚点;持久化行为即 B.3 的 `WorldAnchor` 行为。

### B.3 `WorldAnchor`/`WorldTrackingProvider` — visionOS 持久化模型(已确认)

Apple 一手文档直引:

#### B.3.1 "Tracking specific points in world space"(https://developer.apple.com/documentation/visionos/tracking-points-in-world-space)
> "Use world anchors along with an arkit session's `WorldTrackingProvider` to track points of interest in the world over time, as a person moves while wearing the device, **and across device usage sessions**."
> "ARKit keeps track of a unique identifier for each world anchor your app creates and **automatically places those anchors back in the space when the person returns to your app in the same location**."
> "**Persist world anchors across sessions** — The only information ARKit persists about the world anchors in your app is their **UUID** — a `WorldAnchor` instance's `id` property — and pose in a particular space. It's your app's responsibility to persist additional information, such as the meaning of each anchor."
> "...the `anchorUpdates` sequence only provides world anchors for **nearby objects**."

#### B.3.2 "Bringing your ARKit app to visionOS"
> "If you use the world-tracking data provider in visionOS, **ARKit automatically persists the anchors you add to your app's content. You don't need to persist these anchors yourself.**"

#### B.3.3 `WorldAnchor` API(https://developer.apple.com/documentation/arkit/worldanchor)
```
struct WorldAnchor
init(originFromAnchorTransform:)
var id: UUID                  // ARKit 唯一持久化的东西(+ transform)
var originFromAnchorTransform
var isTracked
var description
```
无 `Codable`/`Serializable`/`save`/`load`。文档:"ARKit persists world anchor UUIDs and transforms across multiple runs of your app."

#### B.3.4 `WorldTrackingProvider` API(https://developer.apple.com/documentation/arkit/worldtrackingprovider)
```
final class WorldTrackingProvider
init()
anchorUpdates             // async sequence
allAnchors
addAnchor(_:)             // 注册锚点;系统持久化
removeAnchor(_:)
removeAnchor(forID:)
queryDeviceAnchor(atTimestamp:)
state, isSupported, requiredAuthorizations
```

#### B.3.5 已确认限制(WWDC24 Session 10100 "Create enhanced spatial computing experiences with ARKit")
> "If the system switches to orientation-based tracking, your existing world anchors may be marked as 'not tracked.' **When world tracking recovers, the tracked status of your persisted anchors will be restored.**"

既确认 "persisted" 用于世界锚点,也确认失败模式:弱光下锚点可能临时 `isTracked == false`。

### B.4 `SceneReconstructionProvider`/RoomPlan mesh — 会话级,不持久

`SceneReconstructionProvider`(https://developer.apple.com/documentation/arkit/scenereconstructionprovider)文档为"A source of **live data** about the shape of a person's surroundings"。接口 `anchorUpdates`/`allAnchors`/`state`/`modes`/`init(modes:)`。**参考页无任何持久化/保存/跨会话措辞**——与明确点名跨 run 持久化的 `WorldAnchor` 页对比鲜明。mesh 每次 session 实时重建。`RoomAnchor`(来自 `RoomTrackingProvider`)同样会话级。

**有意义的非对称:世界锚点持久;mesh/平面/房间锚点不持久。**

### B.5 WWDC session 实情

| Session | 持久化/重定位内容 |
|---|---|
| WWDC23 "Meet ARKit for spatial computing"(10082)| 介绍 ARKitSession+provider 架构;摘要无显式持久化讨论 |
| WWDC24 "Create enhanced spatial computing experiences with ARKit"(10100)| 确认世界锚点"persisted";弱光仅朝向回退;引入 RoomTrackingProvider/ObjectTrackingProvider/斜面/DeviceAnchor.trackingState |
| WWDC24 "Introducing enterprise APIs for visionOS"(10139)| 主摄像头/条码/passthrough/NE。**无世界地图持久化** |
| WWDC24 "Explore object tracking for visionOS"(10101)| 物体锚点(锚到已知 3D 物体)— 可能的重定位 workaround |

**【纠错/勘误】找不到**名为 "Discover Spatial Tracking" 或 "Explore more of the visionOS world" 的 Apple session。WWDC25 实际 session(awesome-visionOS 索引)含 "What's new in visionOS 26"、"Explore enhancements to your spatial business app"、"Share visionOS experiences with nearby people"、"Explore spatial accessory input on visionOS"——无主要关于锚点持久化,WWDC26 列表也未引入 `WorldMap` 等价物。**WWDC23/24/25/26 无一宣布可导出/可保存的 visionOS 世界地图。**

### B.6 企业 / main-camera-access entitlement — 不增加持久化

(与 A 节交叉引用,此处仅记结论)"Accessing the main camera" 示例确认 entitlement `com.apple.developer.arkit.main-camera-access.allow`(visionOS 2.0+,需企业许可)启用 `CameraFrameProvider`,交付立体 `CameraFrame` 像素缓冲用于 CV/直播。按社区与 WWDC24 企业 notes 摘要,**帧中不含深度数据**(详见 A.4),且**无关联的世界地图或锚点持久化**。纯传感器数据流;任何持久化需自建(对像素做 SLAM,严格劣于原生世界追踪系统)。

### B.7 GitHub / 开源

**无可信开源 visionOS 项目实现跨会话/跨设备世界地图持久化**(底层 API 不存在)。相关:
- Apple 示例 **"Tracking specific points in world space"** — `WorldAnchor` 跨 run 增/取的权威 demo
- PlanePlopper、GoncharKit、RealityBounds、FindSurface、visionOS-2-Object-Tracking-Demo、SpatialYOLO(用主摄像头跑 YOLO)— 均未实现持久世界地图
- Unity 讨论(anchor-and-worldmap-support-on-visionos、how-to-implement-persistent-anchors-in-visionpro)是主社区帖;确认缺口,且 Unity `ARAnchorManager` 包装的是 Apple 自动持久化的 `WorldAnchor`(纠错见 B.2)

### B.8 相比 iOS 明显缺失

| 能力 | iOS ARKit | visionOS ARKit |
|---|---|---|
| 可保存会话地图(`ARWorldMap`)| 是 | **否** |
| `getCurrentWorldMap`/`setWorldMap` 重定位 | 是 | **否** |
| 导出锚点/地图到文件或服务器 | 是 | **否**—仅端侧 |
| 跨设备异步共享(不同时间)| 是(共享地图文件)| **否** |
| Geotracking/经纬度锚点 | 是 | **否**(迁移文档明确不可用)|
| 同一真实点多用户持久锚点 | 是 | 仅经 SharePlay 共处(`SharedCoordinateSpaceProvider`)|
| 人脸/人体追踪 | 是 | **否** |

### B.9 可信 workaround(按来源权威排序)

1. **`WorldAnchor` 自动持久化(Apple 受支持)**。对"角色回到老地方、同设备、同用户",这是受支持路径。在你的 app 存储里持久化**语义含义**(UUID → "这是茶壶锚点");让 ARKit 持久化 UUID + transform。设备回到锚点附近时自动返回。
2. **`ImageTrackingProvider` + `ReferenceImage`**。每会话靠识别环境中一张打印图像重建世界系,再相对该图像锚点放内容。原生、端侧。
3. **`ObjectTrackingProvider` + `ReferenceObject`**。visionOS 2+;锚到房间中持久存在的已知 3D 真实物体。
4. **`SharedCoordinateSpaceProvider` 共处**。基于 SharePlay、实时、同处多设备共享坐标空间。**非异步/跨会话**——只解决多 headset 同处一室。
5. **iOS/iPadOS 中介 + `ARWorldMap`**—Apple 论坛建议的多设备 workaround:在 iPhone/iPad 建图、共享文件、再手动让 Vision Pro 对齐 iOS 设备导出变换。**社区来源,精度差,非 Apple 认可。**
6. **Apple Maps Location Anchor/MapKit 地理锚点**—**visionOS 不可用**。"经纬度 Geotracking"明确列为 visionOS 无等价物。勿作为 visionOS workaround 引用。

### B.10 与本项目关系:场景图物体恒常性

**节点按 `WorldAnchor` UUID 键**(呼应"架构设计决策 8/9":物体恒常性 = 长期信念 + 实时核验)。

- `WorldAnchor.id` (UUID) 是 ARKit 跨 run 唯一持久化的东西 → 把它当作场景图里物体节点的**主键**。
- 你的 app 自己负责持久化"UUID → 语义"映射(如 UUID → "茶壶"、"椅子"、"显示器"),存 app local storage;ARKit 负责 UUID + transform 的空间持久化。
- 设备回到锚点附近,`anchorUpdates` 自动把该 UUID 推回来,你用 UUID 查表恢复语义节点 → 实现"角色几天后回到同一真实位置、还记得这是茶壶"。
- `SceneMesh`/`RoomAnchor`/`PlaneAnchor` 不持久(会话级,见 B.4)→ 场景图的几何细节每次 session 重建,只有 `WorldAnchor` 锚定的语义节点跨 session 活下来。
- 弱光回退(B.3.5):`isTracked` 可能临时 false,但 tracking 恢复后自动 restore → 场景图节点不能因 `isTracked == false` 就丢弃,要标"暂未核验"等回填。

---

## C. 一手来源索引(两源合并,去重)

### C.1 Apple 官方文档
- ARKit in visionOS(provider 全列表):https://developer.apple.com/documentation/arkit/arkit-in-visionos
- Tracking specific points in world space(**关键持久化文档**):https://developer.apple.com/documentation/visionos/tracking-points-in-world-space
- WorldAnchor:https://developer.apple.com/documentation/arkit/worldanchor
- WorldTrackingProvider:https://developer.apple.com/documentation/arkit/worldtrackingprovider
- Bringing your ARKit app to visionOS(迁移指南;"ARKit automatically persists the anchors"):https://developer.apple.com/documentation/visionos/bringing-your-arkit-app-to-visionos
- Saving and loading world data(仅 iOS 的 ARWorldMap):https://developer.apple.com/documentation/arkit/saving-and-loading-world-data
- SceneReconstructionProvider:https://developer.apple.com/documentation/arkit/scenereconstructionprovider
- ImageTrackingProvider:https://developer.apple.com/documentation/arkit/imagetrackingprovider
- AnchorEntity(RealityKit):https://developer.apple.com/documentation/realitykit/anchorentity
- Accessing the main camera:https://developer.apple.com/documentation/visionos/accessing-the-main-camera
- Building spatial experiences for business apps with enterprise APIs for visionOS:https://developer.apple.com/documentation/visionos/building-spatial-experiences-for-business-apps-with-enterprise-apis/
- CameraVideoFormat.supportedVideoFormats:https://developer.apple.com/documentation/arkit/cameravideoformat/supportedvideoformats(for:camerapositions:)
- CameraFrameProvider:https://developer.apple.com/documentation/arkit/cameraframeprovider
- Entitlements:https://developer.apple.com/documentation/bundleresources/entitlements
- Displaying video from connected devices:https://developer.apple.com/documentation/visionos/displaying-video-from-connected-devices
- Vision framework:https://developer.apple.com/documentation/vision
- visionOS 26 Release Notes:https://developer.apple.com/documentation/visionos-release-notes/visionos-26-release-notes
- Apple Developer Enterprise Program:https://developer.apple.com/programs/enterprise/
- Compare Memberships:https://developer.apple.com/support/compare-memberships/
- D-U-N-S Number requirement:https://developer.apple.com/help/account/membership/D-U-N-S/
- Apple Business / Vision Pro for Enterprise:https://www.apple.com/business/enterprise/apple-vision-pro/
- What's new for enterprise in visionOS 2(Apple Support):https://support.apple.com/en-us/121160

### C.2 WWDC sessions
- WWDC23 Session 10082 — Meet ARKit for spatial computing
- WWDC24 Session 10100 — Create enhanced spatial computing experiences with ARKit(世界锚点"persisted" 一手确认):https://developer.apple.com/videos/play/wwdc2024/10100/
- WWDC24 Session 10101 — Explore object tracking for visionOS:https://developer.apple.com/videos/play/wwdc2024/10101/
- WWDC24 Session 10139 — Introducing enterprise APIs for visionOS:https://developer.apple.com/videos/play/wwdc2024/10139/
- WWDC25 Session 223 — Explore enhancements to your spatial business app:https://developer.apple.com/videos/play/wwdc2025/223/
- WWDC25 Session 317 — What's new in visionOS 26:https://developer.apple.com/videos/play/wwdc2025/317/
- WWDC26 Session 283 — Explore Enhancements to visionOS Object Tracking:https://developer.apple.com/videos/play/wwdc2026/283/

### C.3 社区/开发者博客
- Griffin Hurt — Stereo Matching and Depth Map Creation on the Vision Pro(864×704 stereoCorrected 格式、RAFT-Stereo 性能、`NSMainCameraUsageDescription` 改名):https://griffinhurt.com/blog/2025/avp-stereo/
- Vision Engineer (Tony Morales) — Building an Interior Design Visualizer using the Main Camera API("Development Only" 确认;"an image, that's it"):https://vision.engineer/posts/visionos-main-camera-enterprise-api/
- WWDCNotes — WWDC24-10139:https://wwdcnotes.com/documentation/wwdc24-10139-introducing-enterprise-apis-for-visionos/
- FrameSixty — What's New in visionOS 27: WWDC 2026 Highlights(Foveated Streaming、Spatial Accessories、Visual Intelligence):https://framesixty.com/whats-new-in-visionos-27/
- UploadVR — Apple Will Give Enterprise Access To Vision Pro's Cameras:https://www.uploadvr.com/apple-vision-pro-enterprise-raw-camera-access/
- Stack Overflow — provisioning profile & main-camera-access entitlement:https://stackoverflow.com/questions/78796607/
- Reddit r/VisionPro — Open Source Object Detection with YOLOv11 and Main Camera Access:https://www.reddit.com/r/VisionPro/comments/1jner1e/
- Reddit r/VisionPro — The API in visionOS 26 is still limited:https://www.reddit.com/r/VisionPro/comments/1l8hdtk/
- Blake Crosley — Apple Vision Framework: On-Device CV Most Devs Skip:https://blakecrosley.com/blog/vision-framework-built-in
- GitHub — Waley-Z/visionos-main-camera setup guide:https://github.com/Waley-Z/visionos-main-camera
- Apple Forums — "VisionOS Equivalent to ARWorldMap"(社区 + 开发者分析;**非 Apple 员工**):https://developer.apple.com/forums/thread/831943
- Unity Discussions — anchor-and-worldmap-support-on-visionos(澄清 ARAnchorManager 是 Unity 的):https://discussions.unity.com/t/anchor-and-worldmap-support-on-visionos/344102
- GitHub — tomkrikorian/awesome-visionOS(项目/示例索引):https://github.com/tomkrikorian/awesome-visionOS

### C.4 关键事实区分(一手 vs 社区)

- **Apple 文档确认**:entitlement 名、申请 URL、资格措辞("business setting only")、分发约束、"Development Only" 选项、许可文件要求+到期、`VisionEntitlementServices` API、`CameraFrameProvider` API 面、UVC/Neural Engine 在 visionOS 26 对所有开发者免费、Foveated Streaming 框架(visionOS 27);`WorldAnchor` 跨 session 自动持久化、只持久化 UUID + transform、`ARWorldMap` 缺失、Geotracking visionOS 不可用。
- **社区报告/未文档化**:864×704 stereoCorrected YUV 格式与 ~60 fps 上限、`NSMainCameraUsageDescription` 从 `NSEnterpriseMCAMUsageDescription` 改名、~6 周时限开发许可、FP32-on-ANE Core ML 限制、几周~月的实际审批时间、visionOS CameraFrame 不含 iOS `ARFrame` 那种 depth/segmentation 丰富度;"`ARAnchorManager`" 非而 Apple API(是 Unity 的);iOS+ARWorldMap 中介 workaround(精度差、非 Apple 认可)。

### C.5 无法确认项 / 方法注
- Apple visionOS ARKit 参考中**无 `ARAnchorManager`、iOS 式 `ARAnchor`、`Serializable` 或任何"世界锚点文件" API**。未臆造任何。
- 找不到名为 "Discover Spatial Tracking"/"Explore more of the visionOS world" 的 Apple session;可能是记错的名字。
- WWDC25/visionOS 26 session 列表未宣布可保存世界地图;若存在需在实际 session 转录中核验。未发现。
- "Apple Maps Location Anchor" 地理锚点 workaround 按迁移文档明确 visionOS 不可用;勿用。
- **方法注**:`developer.apple.com` 与 `docs.developer.apple.com` 对直接 WebFetch 被屏蔽,故一手文档文本经 `docs.developer.apple.com/tutorials/data/.../*.md` 镜像(Apple 自家 DocCD JSON 渲染为 markdown)取回,并对同 URL 的 WebSearch 引用交叉核验。WWDC24 Session 10100 文本为逐字 Apple 转录。本报告无任何 API 名为杜撰。
