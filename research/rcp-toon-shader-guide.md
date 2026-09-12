# RCP cel/toon 材质操作指引(2026-08-04)

> 攻克 Phase A 的二次元明暗色块渲染。**前提结论**:运行时手写 MaterialX(ShaderGraphMaterial)已判死——nodegraph 被 RealityKit fall back 成法线可视化(见 [phase-a-build-log.md](phase-a-build-log.md) §5)。**正道 = Reality Composer Pro 的 Shader Graph**(节点正确绑定、有 Debug Views、cel/toon 是成熟用法)。RCP 已装:`/Applications/Xcode.app/Contents/Applications/Reality Composer Pro.app`。

## 0. 原理(一句话)

**cel = 贴图底色 × 量化(N·L)**。N 是几何法线,L 是固定主光向;N·L 被切成 2~3 档硬色块(亮/中/暗),乘到贴图底色上 → 二次元明暗。用 **Unlit Surface 直出**(绕开 PBR 的连续光照),关键是 RCP 的 `Normal` + `Image` 节点在 Unlit 下应能正确取值(手写时取不到,RCP 高层绑定应该能——阶段 1 专门验证这点)。

## 1. 准备:打开 RCP + 导入模型

1. 打开 RCP(Xcode →菜单 Open Developer Tool → Reality Composer Pro,或直接双击上面那个 .app)。
2. New Project(空场景)。
3. **File → Import**(或直接拖)→ 选 test0 工程里的占位模型 `<占位模型>.usdz`(第三方版权物,仅本地测试,不入库不分发)。
4. 视口里出现占位模型,默认 PBR 显示——确认能看到模型 + 贴图正常(和 app 里 flat 之前的 PBR 一样)。若太暗,RCP 里加个灯或调 IBL。
5. 左侧层级展开,找到 6 个材质(cloth / other / Hair / suit / Body / head,对应 `vrmPartOrder`)。每个材质导入后已经是 **PBR Surface + 已连好的 BaseColor Image 节点**——这张 Image 我们等下直接复用,不用重指定贴图。

## 2. 阶段 1:在 head 上做最简 cel(先不接贴图,验证明暗分块)

> 目的:**先证明 RCP 的 Normal + If Greater 量化在 Unlit 下能出明暗色块**。这一步通了,cel 就成了一半;通不过就走 §5 fallback。先用纯色底色,排除贴图干扰(和 app 里的 band test 同思路,但在 RCP 正确环境里)。

1. 双击 **head 材质**打开 Shader Graph 编辑器。你会看到一个 `PBR Surface`(或 Preview Surface)输出节点,以及已连的 `Image`(BaseColor)。
2. 把底色 Image **先断开/记一下**(阶段 2 再接回),临时用一个 **Color3 (Float)** 常量当底色,值设皮肤色,比如 `0.9, 0.75, 0.7`。
3. 加节点(顶部 `+` 或右键 Add,搜名字):
   - **`Normal`**(Geometry 分类)→ 输出 N(vector3)。Inspector 里把 **space 设为 World**。
   - **`Vector3 (Float)`** 常量 → 光向 L,值 `0.3, 0.8, 0.5`(前上方主光,可调)。
   - **点积节点**(搜 `Dot` / `Dot Product`)→ 两个输入接 N 和 L,输出 `ndotl`(float)。
   - **`Float`** 常量 ×3:阈值 `0.0`、亮档 `1.0`、暗档 `0.4`。
   - **`If Greater`** → `value1`=ndotl, `value2`=0.0, `True Result`=1.0, `False Result`=0.4 → 输出 `shade`(float)。(这就是两段 cel:朝光面 1.0,背光面 0.4,硬切。)
   - **`Multiply`** → 底色 Color3 × shade → 输出 `final`(color3)。
4. 把 surface 输出节点换成 **`Unlit Surface (RealityKit)`**(删掉 PBR/Preview 节点,Add 搜 Unlit)。把 `final` 接到 Unlit 的 **Color**(或 emission/发光颜色)输入,Opacity 接 1.0。
5. **验证(关键检查点)**:看 RCP 视口里 head:
   - ✅ 出现**明暗色块**(朝光面亮、背光面暗、有明显硬边分块)→ **Normal+量化在 Unlit 下 work,cel 通了**。→ 进阶段 2。
   - ❌ 整片均匀单色、转视角无明暗变化 → Normal 在 Unlit 下没取到。→ 走 §5 fallback(PBR emission)。
   - 💡 可用 RCP 右上角 **Debug Views** 切到 Normal,确认法线是否正确(彩色 smooth,不是纯色)。

> 三段色块(更细腻)可选:把单个 If Greater 换成两层嵌套:
> `shade = IfGreater(ndotl, 0.5, 1.0, IfGreater(ndotl, 0.0, 0.65, 0.35))` → 亮/中/暗三档。

## 3. 阶段 2:接贴图

1. 把阶段 1 临时用的 Color3 底色,换回材质原本那个 **`Image`** 节点(导入时已连的 head BaseColor)。`Multiply` 的底色输入接这个 Image 输出。
2. **确认 UV**:Image 默认用 UV0;VRM 网格 UV 是 `primvars:st`。若贴图错位/重复,给 Image 的 UV 输入接一个 **`Texture Coordinates`** 节点(选 st / UV0 对应的集),直到贴图正。
3. 验证:head 应为「贴图底色 × 明暗色块」= cel 效果(有贴图细节 + 硬边明暗)。这就是目标效果。

## 4. 阶段 3:复制到其余 5 个 part + 导出

1. 对 cloth / other / Hair / suit / Body 重复阶段 2:在 RCP 里可**复制 head 的 toon nodegraph**,粘贴到其它材质,只把 Image 换成各自的 BaseColor(导入时各材质已有自己的 Image,直接用)。
2. 6 个 part 都换成 toon 后,整体预览确认。
3. **File → Export → USD**:格式选 `.usdc` 或 `.usdz`,勾选 **embed textures / flatten off**(贴图要打进包)。导出到 `<占位模型>_toon.usdz`(或覆盖)。
4. 告诉我导出完成,我负责集成回 test0 工程(Entity(named:) 加载,替换当前 flat)。

## 5. Fallback:Unlit 拿不到 Normal 时

若阶段 1 验证失败(Unlit 下 Normal 没值),改用 **`PBR Surface (RealityKit)`**:
- PBR 一定绑定法线(它要算光照),所以 `Normal` 节点能取到。
- 把 §2 的 `final`(cel 颜色)接到 PBR 的 **emissive / emission_color**(自发光项,直出、不被光照二次乘)。
- PBR 其它输入压平:diffuse/base_color = 黑或 0,roughness=1,metallic=0,specular=0,让 emission 项主导输出。
- 这样绕过 PBR 的连续光照,但借它的 normal 绑定算 cel 的 N·L。本质:用 PBR 的壳,出 Unlit 的 cel。

## 6. 集成回工程(我做)

导出的 .usdz 材质已是 toon,工程侧只需:
- 把 `<占位模型>_toon.usdz` 放进 test0 app target(同步文件夹自动进 bundle)。
- ContentView 里 `Entity(named: "<模型实体名>")` 直接加载(材质已内嵌 toon),**移除运行时 `applyFlatToon`/`applyToonShader`**(材质自带 cel,不需运行时换)。
- 脸通道(BlendShapeWeightsComponent)不受影响(只改材质,不动几何/绑定)。
- 编译 + 模拟器(干净重启)→ 用户肉眼 + MCP 双重验证。

## 7. 可选增强(后续)

- **Rim/描边**:加 `Dot(View Direction, Normal)` → Fresnel → 暗化边缘,二次元轮廓感。
- **阴影色偏**:暗档不是底色×0.4,而是偏冷色(× 偏蓝),更像动画上色。
- ** specular 高光点**:脸上加小圆点高光(纹理或计算)。

## 关联
- 判死手写的全过程:[phase-a-build-log.md](phase-a-build-log.md) §5(band test → 青色法线可视化 → nodegraph fall back)。
- 节点参考:[rcpguide.com](https://rcpguide.com/)(RCP 全节点 + Apple 描述)。
- 架构:cel 是 Phase A 占位渲染;正式模型/产品阶段再精修或换渲染路径。
