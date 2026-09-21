# PRESENCE · 在场

**真实房间里的具身 AI 陪伴角色(Apple Vision Pro)**

> 大脑 = 具身推理 LLM,小脑 = 连续动作 VLA(π0/GR00T 范式),眼与耳 = Vision Pro 传感器。
> 她感知你的真实房间与屏幕,跨会话记住一切,坐在你真实的椅子上。

**为什么值得做**:目前没有任何系统同时具备 AR 空间 + 真实房间 grounding + 跨会话持久记忆 + 真实家具 affordance(坐/躺/趴)+ 屏幕感知 + VLA 锚定动作。最接近的两者各缺一半——SOLAMI(CVPR'25,端到端社交 VLA,但环境是 VR 合成)与阿里 MNN-TaoAvatar(Vision Pro 已有 90fps demo,但无真实房间感知)。逐项对比见 `research/competitive-positioning.md`。

**为什么是虚拟身体**:物理人形造不起,但具身智能的"脑-感知-动作"闭环可以先在虚拟身体里跑通。换身体 = 换两个接口,换脑子 = 换模型——新基础模型发布当天就能接进闭环评测;在这里练好的每一版脑子,都可以搬上物理身体。

---

## 架构

```
┌────────────── Vision Pro(显示 + 传感)──────────────┐
│ passthrough 真房间 + 角色渲染 90Hz(纯动画平滑,不过脑)   │
│ SceneReconstruction mesh / WorldAnchor / 手·眼·脸追踪   │
│ 企业摄像头 864×704 立体帧(只像素,无深度无分割 → 自己算) │
└──────────────────────────┬───────────────────────────┘
                           │ 30Hz ingest ── LAN 10GbE ──┐
┌──────────────────────────▼───────────────────────────┐
│ Mac M1 Max(编排 + 渲染 + 本地反射)                     │
│ RealityKit 渲染 / IK / barge-in / 行为基线(永远在跑)    │
└──────────────────────────┬───────────────────────────┘
┌──────────────────────────▼───────────────────────────┐
│ 工作站 RTX Pro 6000 96GB(常驻大脑,7×24)               │
│ 感知 5–10Hz:SAM3.1 / VLM / 屏幕-VLM → 场景图           │
│ 认知 1–5Hz:大脑 LLM(人格/意图/对话)+ 小脑 VLA(关节轨迹)│
│ 持久动态场景图(物体恒常性)+ 前瞻预判服务                │
└───────────────────────────────────────────────────────┘
```

**关键决定**(推导过程在 `research/architecture-log.md`,此处只留结论):

| # | 决定 | 一句话理由 |
|---|---|---|
| 1 | 大脑/小脑双系统 | LLM 泛化"该干嘛",VLA 泛化"怎么动";语义与运动各归其位 |
| 2 | 多时间尺度,认知与渲染解耦 | 90Hz 渲染 / 30Hz ingest / 5–10Hz 感知 / 1–5Hz 认知 / <200ms 反射;200–500ms 的刻意反应读起来自然 |
| 3 | 三层 subsumption | 慢审议(大脑)+ 行为基线(呼吸/眨眼/微动,常驻)+ 快反射(本地);大脑只在"需要决定"时介入 |
| 4 | 场景图即认知 | 角色的"视觉"是视角无关的 3D 场景图(结构化文本),不是 POV 像素;仅"凑近看"才渲 POV |
| 5 | 记忆 = 信念,实时感知 = 执行前核验 | 世界 = 走过之处的累积;记忆会过期(椅子被挪走),动作前用当前 FOV 局部重核验 |
| 6 | sim-first,RealityKit 当仿真环境 | sim 与 AR 共用全部代码,只换 PerceptionSource / Display 两个插头 |
| 7 | VLA 微调最小化 | Phase A 零微调可跑(A/C 通道兜底);LoRA 只训自研 rig 关节空间,数据存标准 SMPL-X,换底座模型只重训头 |

---

## 当前进度(2026-09)

| 模块 | 状态 | 备注 |
|---|---|---|
| VRM → RealityKit 资产管线 | 已跑通 | Blender USD 导出不写法线 → pxr 按位置焊接注入;metallic=1 修复;布料缓存 `*_0_0` 去重;`usdzip --arkitAsset` 内嵌贴图 |
| blendshape 脸通道 | 已跑通 | 根因是 UsdSkel 平行数组错位(92 名 vs 91 目标,index 77 起错位 → 整绑定判废);按规范逆向定位、重写 `skel:blendShapes` 后 52 维表情可代码驱动 |
| cel 卡通着色 | 未实现 | 4 次尝试均卡在 RealityKit 26.5 MaterialX 贴图采样墙(与接不接 UV 无关);当前 flat 兜底,死因分析见构建日志 |
| M0 五端口骨架 / M1 最小闭环 | 进行中 | loop-first:先全 stub 端到端,再把 stub 逐个换真 |
| 大脑(LangGraph)/ VLA 微调 | 设计已定稿,未开工 | 契约 A–E 已细化到可开工 |
| 立项调研 | 已完成 | 13 篇(2026-06~08),`research/`,只读存档 |

> 工程日志(含每个坑的现象→根因→修法→验证)在 `research/phase-a-build-log.md`。

---

## 核心设计(伪代码)

> 下面的伪代码是开工规格的骨架:端口即 Swift protocol,循环频率即调度约束,与 `research/contracts-v1.md` 的数据契约 A–E 对应。

### 1. 五端口——sim 与真机只差两个插头

```swift
protocol PerceptionSource {          // Phase A: 脚本生成假观测; Phase B: 真实 ARKit
    var stream: AsyncStream<Observation> { get }   // 30Hz: mesh + 手眼脸 + 立体帧 + Mac 截屏
}
protocol DisplaySink {               // Phase A: Mac 屏; Phase C: VP passthrough + AR
    func render(_ frame: RenderFrame)              // 90Hz
}
protocol BrainService {              // 工作站, 1–5Hz
    func decide(_ ctx: CognitionContext) async -> Decision    // 契约 A
}
protocol CerebellumService {         // 工作站, 连续动作 VLA
    func act(_ input: PolicyInput) -> JointTrajectory           // 感知 → rig 关节轨迹
}
protocol MemoryService {
    func sceneGraph(region: Region?) -> SceneGraph               // 契约 B
    func verify(_ anchor: AnchorUUID) async -> SceneNode?        // 执行前核验
}
```

### 2. 多时间尺度调度(各层各跑各的,只通过总线说话)

```
90Hz    Mac/VP      MotionResolver.blend(baseline, overrides) → display.render
30Hz    VP → Mac    perception.stream → ingestQueue
5–10Hz  工作站      SAM3.1 / VLM / 情感融合(ingestQueue) → memory.update(delta)
1–5Hz   工作站      brain.decide(ctx) → Decision{intent, utterance, affect}
<200ms  Mac(本地)  on barge-in | gaze-shift | intrusion → reflex.handle   // 不过脑
```

### 3. 三层仲裁(subsumption:反应速度递减,思考量递增)

```python
def motion_tick(dt):
    if reflex.pending():                       # <200ms 层,最高优先级,本地
        return reflex.apply()                  #   barge-in / 注视跟随(纯IK) / 侵入姿态
    if brain.has_fresh_decision():             # 慢审议层意图刚到达
        cerebellum.set_intent(brain.decision.intent)
    return baseline.tick(dt)                   # 行为基线:呼吸/眨眼/换重心,永远在跑
```

### 4. 大脑——LangGraph 状态图(人格/记忆/反思)

```python
brain = StateGraph(PersonaState, checkpointer=persistent)   # 人格在系统提示词,不外采角色引擎
brain += node("perceive", compile_scene_graph)   # 场景图 → 按对话相关性编译为上下文
brain += node("reflect",  ecot_monologue)        # 内心独白流(活感:她一直在想)
brain += node("reason",   react_tools)           # ReAct 主线
brain += node("select",   saycan_score)          # 意图 × 场景图 affordance 打分 → 可行动作
brain += node("speak",    stream_tts)            # 首包 <500ms
# select 输出契约 A:
# Decision{ "intent": "sit_on(anchor:躺椅)", "utterance": "...", "affect": {...} }
```

### 5. 大脑 → 小脑:自然语言意图串当 token-prefix

```python
def intent_to_vla_prompt(d: Decision) -> str:
    # π0/GR00T 原生支持 prompt 条件化 —— NL 意图串直接当前缀,免适配器、免重训 backbone
    return f"<intent>{d.intent.nl()}</intent><persona>{PERSONA_TAG}</persona>"
# 分工:小脑只扛身体大动作;脸(Audio2Face-3D)/注视(IK)/手势/呼吸走 MotionResolver 细通道
```

### 6. 物体恒常性——记忆是信念,执行前核验

```python
async def act_near(anchor: AnchorUUID):
    belief = memory.scene_graph().get(anchor)        # 记忆(可能过期)
    fresh  = await memory.verify(anchor)             # 当前 FOV 局部重核验
    if fresh and close(belief.pose, fresh.pose):
        return cerebellum.execute(belief)
    return replan(fresh or re_perceive(anchor))      # 椅子被挪走 → 优雅失败/重感知,不硬坐
```

### 7. 贴家具 affordance-fit(Phase C 核心自研,无现成端到端解)

```python
def affordance_fit(furniture_mesh, rig) -> Pose:
    p0 = retrieve_or_generate(furniture_mesh)   # 动作库检索 / HOSIG·CHOIS 生成粗姿态
    p1 = contact_optimize(p0, furniture_mesh)   # PhySIC:接触 + 穿透优化
    p2 = physics_settle(p1)                     # RealityKit 物理沉降(不漂浮)
    return ik_fine(p2)                          # WWDC26 IKcomponent 精修手/脚/视线
```

---

## 路线图

| 阶段 | 时间 | 出口判据 |
|---|---|---|
| **A · sim 闭环** | 2026.10–11 | M0 端口骨架 → M1 stub 闭环 → M2 rig 关节空间定型 → M3 各子系统接真 → M4:虚拟房间里听-看-想-动全闭环 |
| **B · 真实感知** | 2026.12–2027.01 | 企业摄像头 entitlement → SAM3.1+VLM 语义场景图;屏幕感知通道;对话 <500ms |
| **C · AR 落位** | 2027.02–03 | VP passthrough 渲染;跨会话物体恒常;贴真家具坐/躺 v1;首次 VLA LoRA |
| **可演示** | 2027.04 | 她坐在真实的椅子上、看着你的屏幕、记得上周说过的话 |
| **D · 活感打磨** | 2027.05+ | 行为导演层 / 连续微反应 / 前瞻预判;开源发布 |

---

## 仓库地图

```
README.md              你在这里(项目脸面 + 设计规格骨架)
research/              立项调研存档(13 篇,2026-06~08,只读;结论已吸收进本文件)
  ├─ overview.md           执行摘要
  ├─ architecture-log.md   架构决定与开放问题(权威)
  ├─ contracts-v1.md       数据契约 A–E
  ├─ roadmap-phased.md     分阶段路线(sim-first)
  ├─ phase-a-build-log.md  Phase A 实测构建日志(坑与修法)
  └─ …                     感知/记忆/动画/VLA/生态分文件
apps/test0/           (规划并入)visionOS 工程:VRM 管线 + blendshape 驱动
```

**硬件**:RTX Pro 6000 96GB 工作站(常驻大脑)+ M1 Max 32GB(编排/渲染)+ Vision Pro,10GbE 局域网,全本地、全隐私。

**联系**:https://github.com/fsadj
