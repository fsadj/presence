# Phase A 构建日志:VRM → visionOS RealityKit(实测工程笔记,2026-08-03)

> 这是 Phase A **实际动手**的工程记录(坑 + 修法 + 命令),区别于 `research/` 里偏研究/全景的文档。**代码项目在 `/Users/brazion/apps/vr/test0`**(Xcode visionOS app,独立于本 `research/` 目录)。持续更新。

## 0. 现状快照

| 项 | 状态 |
|---|---|
| 代码项目 | `/Users/brazion/apps/vr/test0`(scheme `test0`,bundle `vr.test0`,visionOS 26.5) |
| 占位角色 | 第三方版权 VRM(**仅 Phase A 本地占位**,正式阶段替换为合规定制形象) |
| 渲染管线 VRM→RealityKit | ✅ 通(模拟器渲染、贴图正确、可旋转、补光) |
| 脸通道(blendshape 驱动) | ✅ 通(jawOpen 可驱动,见 §4) |
| 卡通着色器(cel) | ❌ RealityKit 26.5 不可行(MaterialX 贴图采样墙,4 次尝试);用平面 flat 兜底(见 §5) |
| SMPL-X retarget / 小脑 VLA | ⬜ 未开始(M-1 之后) |

## 1. 可用管线:VRM → RealityKit(实测可行)

```
.vrm (VRM/VRoid)
  → Blender 5.x(装 VRM Addon)导入,导出 .usdc(格式选 .usdc,勾形态键/动画)
  → usdzip --arkitAsset <model>.usdc <model>.usdz   # 内嵌贴图 + RealityKit 合规
  → 拖进 Xcode 工程的 app target(同步文件夹会自动进 bundle)
  → RealityView 里 Entity(named:"<名>") 加载
```

> Blender 的 USD 导出**不写法线、不写 blendshape 绑定、材质 metallic=1**(见 §2),所以 .usdc 出来还要 pxr 后处理(§2),才能在 RealityKit 里正常显示 + 驱动表情。

## 2. 实测坑 + 修法(按踩坑顺序)

### 2.1 Blender USD 导出不写法线 → 分界面 / 雀斑
- 现象:Blender 视口正常(实时算法线),但导出的 .usdc `primvars:normals = 0`;RealityKit 按面法线 → 腿多边形棱角、脸雀斑状深深浅浅。
- 修:**pxr 注入平滑顶点法线**,且**按位置焊接**(否则 UV 接缝处重合顶点法线不一致 = 接缝断层)。脚本骨架(用 `usd-core`,`from pxr import Usd, UsdGeom, Sdf, Vt, Gf`):
  ```python
  for mesh prim:
      pts=...; fvc=...; fvi=...
      posnorm = defaultdict(Vec3)  # 按 round(p,4) 位置聚合,焊接接缝顶点
      for each face: n=normalize(cross(p1-p0,p2-p0)); posnorm[pos(vi)] += n
      norms[i] = normalize(posnorm[pos(pts[i])])
      UsdGeom.PrimvarsAPI(prim).CreatePrimvar("normals", Sdf.ValueTypeNames.Float3Array, UsdGeom.Tokens.vertex).Set(Vt.Vec3fArray(norms))
  ```
- 注:Blender 默认/手勾 Normals 都不写;必须 pxr 后注。usd-core 装:`pip install usd-core -i https://pypi.tuna.tsinghua.edu.cn/simple`(国内直连 PyPI 慢/超时,**用镜像**)。

### 2.2 材质 metallic=1, roughness=1, specular=0 → 粗糙纯金属发怪/发暗
- VRM 的 MToon 转 UsdPreviewSurface 时 metallic 被设 1 → 金属无漫反射、几乎不反光。
- 修:所有 Shader `inputs:metallic = 0`(roughness 给 0.5~0.6)。可文本改(usdc→usda→sed→usdc)或 pxr(注意实例 master 的 Shader `.Set` 会抛 `UsdExpiredPrimAccessError`,要 try/except 跳过)。

### 2.3 布料缓存重复几何(*_0_0)→ z-fighting / 面数翻倍
- 每个部件有两份 `*_baked`(Blender 布料缓存残留: `<part>/<part>_baked` + `<part>/<part>_0_0/<part>_baked`)。
- 修:pxr 删所有 `endswith("_0_0")` 的 Xform。顺带消除"幽灵脸"(两份 head_baked 表情不同步),对驱动表情有利。

### 2.4 USD 里的灯被 RealityKit 丢弃 → 暗
- 文件里的 SphereLight/DomeLight 在 RealityKit 导入时不变成 RealityKit 灯;体积窗口默认只有系统 IBL,常偏暗。
- 修:RealityView 里手动加 `DirectionalLightComponent` 实体(主光+补光)。

### 2.5 贴图外挂 + 打包
- Blender 导出贴图到 sibling `textures/`,usdc 引用 `@./textures/x@`。必须打进包,否则 RealityKit 显示灰/无贴图。
- `usdzip --arkitAsset <usdc> <usdz>`:**内嵌贴图(保 `textures/` 前缀)+ RealityKit 合规**,但会 flatten。
- ⚠️ `usdzip --asset`(不扁平)**会把贴图路径改成 `0/` 前缀,破坏引用 → 贴图丢失**。要保贴图用 arkitAsset。

### 2.6 .usdc ↔ .usda / round-trip
- `usdcat a.usdc > a.usda`(转文本);`usdcat a.usda --out b.usdc`(转回二进制)。无损,blendshape/skeleton 都在。
- ⚠️ **Blender 的 USD 导入→导出往返会丢 shape key + skeleton + 法线**。所以修复一律在源 .usdc 上 pxr 后处理,**不要往返 Blender**。

### 2.7 法线/双面都修不好的"腿断层" → 占位接受
- 低模(全身才 ~6k 面)+ 单面渲染 + UV/贴片接缝叠加;法线(两版)、双面、去重都试过仍在。免费 VRM 固有限制,Phase A 占位接受。

## 3. RealityKit 加载/驱动 API 笔记(visionOS 26.5,实测)

- 加载:`try await Entity(named:"名")`(主 bundle;同步文件夹里的资源自动进 bundle)。
- 找子实体:`entity.findEntity(named:"名")`(**不是** `findModelEntity`——不存在)。
- **不要** `import RealityFoundation`(是 RealityKit 实现细节,编译报错);`import RealityKit` 已暴露全部类型。
- blendshape 组件:`BlendShapeWeightsComponent`(在 `RealityFoundation` 定义,但经 RealityKit 暴露)。读写:`comp.weightSet["<表情名>"]`(BlendShapeWeightsData,`.weights` 可写)+ `comp.weightSet.set(data)`;`weightSet.contains("名")` / `.blendShapeNames`。
- 方向灯:`DirectionalLightComponent(color:intensity:)` + `entity.components.set(...)`;朝向用 `transform.rotation`。
- 日志:`Logger(subsystem:"<id>", category:".").info(...)` → `xcrun simctl spawn <udid> log show --last 90s --predicate 'subsystem=="<id>"' --info`。(`print()` 被 stdout 缓冲,`simctl launch --console-logs` 抓不到——用 Logger。)

## 4. 脸通道 — ✅ 已通(RealityKit blendshape 驱动)

- **目标**:代码驱动 ARKit blendshape(如 `jawOpen`)→ M-4(Audio2Face lip 同步)前置。
- **现象(初始)**:RealityKit 加载后不给任何实体建 `BlendShapeWeightsComponent`(实测 walk:只有 "Armature" 实体 model=true,blendshape 全 false)。
- **真实根因(比"缺绑定"更细)**:Blender 导出的绑定**其实在**,但**被长度不匹配破坏**——
  - `rel skel:blendShapeTargets`(91 个 BlendShape 目标 prim)**是存在的**(注意规范名是 `blendShapeTargets`,不是 `blendShapes`)。
  - 但 `uniform token[] skel:blendShapes` 名字数组有 **92** 个(布料缓存去重残留的伪 `_2` 混在 index 77)。
  - UsdSkel 要求两者**平行数组**(name[i]↔target[i]);77 之后错位 → RealityKit 整个 blendshape 绑定判废 → 不建组件。
  - 注:UsdSkel **没有** `skel:blendShapeIndices` 属性(索引按名字隐式查 SkelAnimation,不像我先前以为的要单独 attr)。
- **修法(pxr)**:`UsdSkel.BindingAPI.CreateBlendShapesAttr()` 重写 `skel:blendShapes` = 91 个目标 basename(去掉伪 `_2`,与 targets 严格平行)+ `CreateAnimationSourceRel()` 指到 SkelAnimation。`usdchecker` 通过;`usdzip --arkitAsset` flatten **保留**绑定(实测验:91 名/91 目标/animationSource/91 BlendShape prim/法线都在,贴图也正常——`0/` 前缀在包内能解析,不算坏)。
- **RealityKit 怎么暴露(踩到的两个点)**:
  1. 组件挂在 **"Armature"** 实体(SkelRoot 级),**不是** head_baked——找它要按"谁有 `BlendShapeWeightsComponent`"递归找,**别按名字**找 head_baked。
  2. `weightSet` 是**两层**:一个 `default` 组(id=""),其 `weightNames` 含全部 91 名;驱动要走 `comp.weightSet.default`,用 `weightNames.firstIndex(of:name)` 定位 `data.weights[idx]`。**直接 `weightSet["jawOpen"]` 顶层取 = nil**(初版驱动失效就是这原因)。
- **验证**:jawOpen=0.8 → 嘴张开(下颌落、口腔变暗)✅;重置 0 闭合 ✅(截图确认)。
- **产物**:`<占位模型>_bound.usdc`(绑定后)→ 打包成 `apps/vr/test0/<占位模型>.usdz`;ContentView 的 `driveJaw` 已按两层结构改对;旧 usdz 备份在 `/tmp/<占位模型>.usdz.bak`。
- 关联研究:[animation-affordance.md](animation-affordance.md) §4(Audio2Face-3D 吐 ARKit52)、[brain-agent-architecture.md](brain-agent-architecture.md)(大脑→小脑 NL 意图)。脸通道是 M-4(lip 同步)前置。

## 5. 卡通着色器(cel 明暗色块)— ❌ RealityKit 26.5 不可行(贴图采样墙);平面 flat 兜底

- 目标:RealityKit 无 MToon → VRM 导入只剩平面 PBR("不够二次元");想要 cel 明暗色块。
- **结论(4 次尝试:2×运行时 + 2×USD,全卡同一墙):cel 在 visionOS 26.5 RealityKit 做不出来。** 死穴是 **MaterialX 的贴图采样节点采不到贴图**。
- **当前可用替代:平面 flat**(`UnlitMaterial` + typealias 复用各材质 baseColor)——贴图对、不崩、偏 2D,但无明暗色块。app 现在用这个。

### 5.1 RealityKit 自定义着色器的硬约束(实测)
1. **`CustomMaterial`+Metal 在 visionOS `@available(unavailable)`——硬墙**。唯一自定义着色器路 = `ShaderGraphMaterial`(MaterialX)。
2. **MaterialX-in-USD 的着色器确实能在 RealityKit 跑**(实证:把 head 材质换成 `surface_unlit` 后,头从 PBR 变成 shader 输出的纯色——shader 被执行,非忽略)。所以"色块数学/自定义着色"本身通。
3. **但贴图采样不工作(cel 死穴)**:
   - `ND_RealityKitTexture2D_color3` 有 `texcoord` 输入、`defaultgeomprop="UV0"`;但 VRM 网格 UV 是 `primvars:st`(≠ UV0)。
   - 实测:不接 texcoord → 灰;**接 `ND_geompropvalue_vector2(geomprop="st")`→texcoord,实跑仍灰**(用户肉眼确认;贴图已正确内嵌、路径对)。即 RealityKit 的 MaterialX 贴图节点**与接不接 UV 无关地采不到贴图**。
   - cel = 贴图 × 色块;贴图进不来 → 塌成纯色(4 次:乱码 / 纯蓝 / 灰,皆此)。
4. 运行时 `ShaderGraphMaterial(materialXLabel:data:)` 额外限制:贴图 `file` 非发布参数(不能 `setParameter`)→ 只能每 part 硬编(脆弱)。

### 5.2 尝试记录(全失败)
- 运行时 cel ×2:MaterialX cel 图 + 硬编 part 顺序 → 崩(乱码 / 纯蓝)。
- USD-MaterialX de-risk ×1:head 挂 `surface_unlit`+`RealityKitTexture2D`,贴图内嵌+路径对 → 灰(证实 shader 跑、贴图不采样)。
- USD-MaterialX + UV ×1:`geompropvalue_vector2(st)`→texcoord → 仍灰。

### 5.3 ⚠️ 验证大坑:模拟器帧缓冲冻结
visionOS 模拟器**频繁重启后帧缓冲会冻结**——`simctl io screenshot` 返回**上一帧陈旧图**(跨完全不同的 shader 都返回同一张)。这**多次骗过 agent 和我的截图/MCP 核验**(把灰判成有贴图、把崩判成成功)。**可靠核验:`simctl shutdown <UDID>` + 重启 + 再截图**;或直接以**用户肉眼**为准(最可靠)。教训:别只信截图。

### 5.4 还剩什么没试(可能性低,搁置)
- 贴图作为材质**发布参数**(published material input)绑定,而非 graph 内 `file`(机制不同,未验证)。
- Reality Composer Pro 编译的 `.reality` 着色器图(贴图烘焙进去;需 GUI)。
- 等 Apple 放开 visionOS Metal 自定义材质 / 修 MaterialX 贴图采样。
- 平面 flat 已够 Phase A 占位;cel 是 RealityKit 已知限制,正式模型/产品阶段再攻或换渲染路径。

### 5.5 产物
- `ContentView.swift`:`applyFlatToon`(当前用,可靠平面);`applyToonShader`/`toonMaterialXML`(cel 尝试,留备查,已停用)。
- pxr 脚本 `/tmp/mtlx_test.py`:USD-MaterialX 作者参考(head + UV + texture + surface_unlit)。
- 节点定义:`/Applications/Xcode.app/Contents/SystemFrameworks/ShaderGraph.framework/Versions/A/Resources/MaterialX_NodeDefs`。

## 6. 关键命令速查

```bash
# 装 pxr(国内镜像!)
python3.13 -m venv /tmp/usdm && /tmp/usdm/bin/pip install usd-core -i https://pypi.tuna.tsinghua.edu.cn/simple

# 打包(保贴图)
usdzip --arkitAsset '/path/model.usdc' '/path/model.usdz'

# 看 USD 内部(不 dump 值用 usdtree;usdcat 会把 offset 数组撑爆)
usdtree model.usdz | grep -i blendshape
usdcat model.usdz | grep -c 'primvars:normals'   # 法线在不在

# 编译(Mac,不需真机)
xcodebuild -project test0.xcodeproj -scheme test0 -sdk xrsimulator -configuration Debug -derivedDataPath /tmp/test0dd CODE_SIGNING_ALLOWED=NO build

# 跑 + 截图
UDID=$(xcrun simctl list devices available | grep -i 'vision pro' | head -1 | grep -oE '[0-9A-F-]{36}')
xcrun simctl boot "$UDID"; xcrun simctl bootstatus "$UDID" -b
xcrun simctl install "$UDID" /tmp/test0dd/Build/Products/Debug-xrsimulator/test0.app
xcrun simctl launch "$UDID" vr.test0
xcrun simctl io "$UDID" screenshot /tmp/shot.png
# 查日志
xcrun simctl spawn "$UDID" log show --last 90s --predicate 'subsystem=="vr.test0"' --info
```

## 7. 关联
- 研究深度:[animation-affordance.md](animation-affordance.md) §7(RealityKit 原生能力)、[roadmap-tracks.md](roadmap-tracks.md) M-1(rig gating)/M-4(脸)、[open-vla.md](open-vla.md)+[data-and-iteration.md](data-and-iteration.md)(SMPL-X/AMASS 对齐)。
- 架构决策:[architecture-log.md](architecture-log.md) 结论 5(动画分通道)/12(大脑+小脑 VLA)。
- 准备清单:[phase-a-preflight.md](phase-a-preflight.md)。
