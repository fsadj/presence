# visionOS 空间锚点持久化 — 一手源调研(2026-08)

> 直接关系到"角色几天后回到同一真实位置"和"角色记住房间"。全部对 Apple 一手文档/WWDC 核验;无臆造 API。

## 一句话结论
visionOS **有**锚点持久化机制——`WorldTrackingProvider` 下的 `WorldAnchor`,且是**自动、系统托管**:你在一个 app run 里加的锚点,当设备回到同一真实位置时,会在未来 run 里自动回来。这是"AR 角色回到老地方"在**同一设备**上的受支持答案。

visionOS **没有**的是 iOS 的 `ARWorldMap` 等价物:**没有可保存/导出的地图 blob、没有 `getCurrentWorldMap`、没有 `setWorldMap`、没有"对文件重定位"API**。锚点"被困"在创建设备上(Apple 论坛/社区)。所有持久化都是不透明、端侧、单用户、单设备。

---

## 1. ARKit on visionOS — 无 `ARWorldMap`/世界地图持久化
**确认缺失。** Apple 概览文档 "ARKit in visionOS" 枚举了平台所有 ARKit data provider,完整列表:`PlaneDetectionProvider`、`WorldTrackingProvider`、`HandTrackingProvider`、`SceneReconstructionProvider`、`ImageTrackingProvider`、`ObjectTrackingProvider`、`BarcodeDetectionProvider`、`RoomTrackingProvider`、`EnvironmentLightEstimationProvider`、`CameraFrameProvider`、`AccessoryTrackingProvider`、`SharedCoordinateSpaceProvider`、`StereoPropertiesProvider`、`CameraRegionProvider`、`VisualFidelityProvider`。**没有 `WorldMapProvider`、没有 `WorldMap`、没有 `MapSession`。** https://developer.apple.com/documentation/arkit/arkit-in-visionos

iOS 文档 "Saving and loading world data"(文档化 `ARWorldMap`、`ARSession.getCurrentWorldMap`、`setWorldMap` 重定位)是 **iOS/iPadOS 专属**。https://developer.apple.com/documentation/arkit/saving-and-loading-world-data

迁移指南 "Bringing your ARKit app to visionOS" 明确列 iOS 功能中**"visionOS 无等价物"**的,含"用经纬度放置锚点的 Geotracking"。https://developer.apple.com/documentation/visionos/bringing-your-arkit-app-to-visionos

## 2. RealityKit on visionOS — 无 `AnchorEntity` 持久化 API
`AnchorEntity` 在 visionOS 存在且遵循 `HasAnchoring`,但其参考页**没有 save/load/serialize/persist API**——只有 anchoring targeting。https://developer.apple.com/documentation/realitykit/anchorentity

**对常见社区说法的重要纠正**:"`ARAnchorManager`" **不是 Apple API**。它是 Unity XR AR Foundation 插件(`UnityEngine.XR.ARFoundation`)的一部分。论坛/YouTube 称"ARAnchorManager 让锚点自动持久"描述的是 Unity 对 Apple `WorldAnchor` 的包装——**非 Apple 原生 API**,不应作为 Apple 持久化 API 引用。

RealityKit 在 visionOS 经 `SpatialTrackingSession`(visionOS 2.0+)或直接 ARKit `WorldTrackingProvider` 暴露世界锚点;持久化行为即下文 §3 的 `WorldAnchor` 行为。

## 3. `WorldAnchor`/`WorldTrackingProvider` — visionOS 持久化模型(已确认)
Apple 一手文档直引:

### "Tracking specific points in world space"(https://developer.apple.com/documentation/visionos/tracking-points-in-world-space)
> "Use world anchors along with an arkit session's `WorldTrackingProvider` to track points of interest in the world over time, as a person moves while wearing the device, **and across device usage sessions**."
> "ARKit keeps track of a unique identifier for each world anchor your app creates and **automatically places those anchors back in the space when the person returns to your app in the same location**."
> "**Persist world anchors across sessions** — The only information ARKit persists about the world anchors in your app is their **UUID** — a `WorldAnchor` instance's `id` property — and pose in a particular space. It's your app's responsibility to persist additional information, such as the meaning of each anchor."
> "...the `anchorUpdates` sequence only provides world anchors for **nearby objects**."

### "Bringing your ARKit app to visionOS"
> "If you use the world-tracking data provider in visionOS, **ARKit automatically persists the anchors you add to your app's content. You don't need to persist these anchors yourself.**"

### `WorldAnchor` API(https://developer.apple.com/documentation/arkit/worldanchor)
```
struct WorldAnchor
init(originFromAnchorTransform:)
var id: UUID                  // ARKit 唯一持久化的东西(+ transform)
var originFromAnchorTransform
var isTracked
var description
```
无 `Codable`/`Serializable`/`save`/`load`。文档:"ARKit persists world anchor UUIDs and transforms across multiple runs of your app."

### `WorldTrackingProvider` API(https://developer.apple.com/documentation/arkit/worldtrackingprovider)
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

### 已确认限制(WWDC24 Session 10100 "Create enhanced spatial computing experiences with ARKit")
> "If the system switches to orientation-based tracking, your existing world anchors may be marked as 'not tracked.' **When world tracking recovers, the tracked status of your persisted anchors will be restored.**"
既确认 "persisted" 用于世界锚点,也确认失败模式:弱光下锚点可能临时 `isTracked == false`。

## 4. `SceneReconstructionProvider`/RoomPlan mesh — 会话级,不持久
`SceneReconstructionProvider`(https://developer.apple.com/documentation/arkit/scenereconstructionprovider)文档为"A source of **live data** about the shape of a person's surroundings"。接口 `anchorUpdates`/`allAnchors`/`state`/`modes`/`init(modes:)`。**参考页无任何持久化/保存/跨会话措辞**——与明确点名跨 run 持久化的 `WorldAnchor` 页对比鲜明。mesh 每次 session 实时重建。`RoomAnchor`(来自 `RoomTrackingProvider`)同样会话级。
**有意义的非对称:世界锚点持久;mesh/平面/房间锚点不持久。**

## 5. WWDC session 实情
| Session | 持久化/重定位内容 |
|---|---|
| WWDC23 "Meet ARKit for spatial computing"(10082)| 介绍 ARKitSession+provider 架构;摘要无显式持久化讨论 |
| WWDC24 "Create enhanced spatial computing experiences with ARKit"(10100)| 确认世界锚点"persisted";弱光仅朝向回退;引入 RoomTrackingProvider/ObjectTrackingProvider/斜面/DeviceAnchor.trackingState |
| WWDC24 "Introducing enterprise APIs for visionOS"(10139)| 主摄像头/条码/passthrough/NE。**无世界地图持久化** |
| WWDC24 "Explore object tracking for visionOS"(10101)| 物体锚点(锚到已知 3D 物体)— 可能的重定位 workaround |

**找不到**名为 "Discover Spatial Tracking" 或 "Explore more of the visionOS world" 的 Apple session。WWDC25 实际 session(awesome-visionOS 索引)含 "What's new in visionOS 26"、"Explore enhancements to your spatial business app"、"Share visionOS experiences with nearby people"、"Explore spatial accessory input on visionOS"——无主要关于锚点持久化,WWDC26 列表也未引入 `WorldMap` 等价物。**WWDC23/24/25/26 无一宣布可导出/可保存的 visionOS 世界地图。**

## 6. 企业 / main-camera-access entitlement — 不增加持久化
"Accessing the main camera" 示例确认 entitlement `com.apple.developer.arkit.main-camera-access.allow`(visionOS 2.0+,需企业许可)启用 `CameraFrameProvider`,交付立体 `CameraFrame` 像素缓冲用于 CV/直播。按社区与 WWDC24 企业 notes 摘要,**帧中不含深度数据**,且**无关联的世界地图或锚点持久化**。纯传感器数据流;任何持久化需自建(对像素做 SLAM,严格劣于原生世界追踪系统)。

## 7. GitHub / 开源
**无可信开源 visionOS 项目实现跨会话/跨设备世界地图持久化**(底层 API 不存在)。相关:
- Apple 示例 **"Tracking specific points in world space"** — `WorldAnchor` 跨 run 增/取的权威 demo。
- PlanePlopper、GoncharKit、RealityBounds、FindSurface、visionOS-2-Object-Tracking-Demo、SpatialYOLO(用主摄像头跑 YOLO)— 均未实现持久世界地图。
- Unity 讨论(anchor-and-worldmap-support-on-visionos、how-to-implement-persistent-anchors-in-visionpro)是主社区帖;确认缺口,且 Unity `ARAnchorManager` 包装的是 Apple 自动持久化的 `WorldAnchor`。

## 8. 相比 iOS 明显缺失
| 能力 | iOS ARKit | visionOS ARKit |
|---|---|---|
| 可保存会话地图(`ARWorldMap`)| 是 | **否** |
| `getCurrentWorldMap`/`setWorldMap` 重定位 | 是 | **否** |
| 导出锚点/地图到文件或服务器 | 是 | **否**—仅端侧 |
| 跨设备异步共享(不同时间)| 是(共享地图文件)| **否** |
| Geotracking/经纬度锚点 | 是 | **否**(迁移文档明确不可用)|
| 同一真实点多用户持久锚点 | 是 | 仅经 SharePlay 共处(`SharedCoordinateSpaceProvider`)|
| 人脸/人体追踪 | 是 | **否** |

## 9. 可信 workaround(按来源权威排序)
1. **`WorldAnchor` 自动持久化(Apple 受支持)**。对"角色回到老地方、同设备、同用户",这是受支持路径。在你的 app 存储里持久化**语义含义**(UUID → "这是茶壶锚点");让 ARKit 持久化 UUID + transform。设备回到锚点附近时自动返回。
2. **`ImageTrackingProvider` + `ReferenceImage`**。每会话靠识别环境中一张打印图像重建世界系,再相对该图像锚点放内容。原生、端侧。
3. **`ObjectTrackingProvider` + `ReferenceObject`**。visionOS 2+;锚到房间中持久存在的已知 3D 真实物体。
4. **`SharedCoordinateSpaceProvider` 共处**。基于 SharePlay、实时、同处多设备共享坐标空间。非异步/跨会话。
5. **iOS/iPadOS 中介 + `ARWorldMap`**—Apple 论坛建议的多设备 workaround:在 iPhone/iPad 建图、共享文件、再手动让 Vision Pro 对齐 iOS 设备导出变换。**社区来源,精度差,非 Apple 认可。**
6. **Apple Maps Location Anchor/MapKit 地理锚点**—**visionOS 不可用**。"经纬度 Geotracking"明确列为 visionOS 无等价物。勿作为 visionOS workaround 引用。

## 来源索引
- https://developer.apple.com/documentation/arkit/arkit-in-visionos — ARKit-visionOS provider 全列表
- https://developer.apple.com/documentation/visionos/tracking-points-in-world-space — **关键持久化文档**
- https://developer.apple.com/documentation/arkit/worldanchor ; https://developer.apple.com/documentation/arkit/worldtrackingprovider
- https://developer.apple.com/documentation/visionos/bringing-your-arkit-app-to-visionos — 迁移指南;"ARKit automatically persists the anchors"
- https://developer.apple.com/documentation/arkit/saving-and-loading-world-data — 仅 iOS 的 ARWorldMap
- https://developer.apple.com/documentation/arkit/scenereconstructionprovider ; https://developer.apple.com/documentation/arkit/imagetrackingprovider
- https://developer.apple.com/documentation/realitykit/anchorentity ; https://developer.apple.com/documentation/visionos/accessing-the-main-camera
- WWDC: 2023/10082, 2024/10100, 2024/10101, 2024/10139
- https://developer.apple.com/forums/thread/831943 — "VisionOS Equivalent to ARWorldMap"(社区 + 开发者分析;**非 Apple 员工**)
- https://discussions.unity.com/t/anchor-and-worldmap-support-on-visionos/344102 — Unity 社区(澄清 ARAnchorManager 是 Unity 的)
- https://github.com/tomkrikorian/awesome-visionOS — 项目/示例索引

## 无法确认项
- Apple visionOS ARKit 参考中**无 `ARAnchorManager`、iOS 式 `ARAnchor`、`Serializable` 或任何"世界锚点文件" API**。未臆造任何。
- 找不到名为 "Discover Spatial Tracking"/"Explore more of the visionOS world" 的 Apple session;可能是记错的名字。
- WWDC25/visionOS 26 session 列表未宣布可保存世界地图;若存在需在实际 session 转录中核验。未发现。
- "Apple Maps Location Anchor" 地理锚点 workaround 按迁移文档明确 visionOS 不可用;勿用。

**方法注**:`developer.apple.com` 与 `docs.developer.apple.com` 对直接 WebFetch 被屏蔽,故一手文档文本经 `docs.developer.apple.com/tutorials/data/.../*.md` 镜像(Apple 自家 DocCD JSON 渲染为 markdown)取回,并对同 URL 的 WebSearch 引用交叉核验。WWDC24 Session 10100 文本为逐字 Apple 转录。本报告无任何 API 名为杜撰。
