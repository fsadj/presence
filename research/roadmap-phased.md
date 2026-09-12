# 分阶段研发路线(sim-first 迁移,Apple 原生;2026-08,设计级)

> 把极复杂的研发拆成可独立验证的阶段;**用 RealityKit 自己当 sim 环境**,在虚拟世界跑通交互再迁移到真实 AR;每阶段都 Apple 原生可落地。本文是**设计级路线**(非开工代码规格——开工暂缓,待设计打磨;见 architecture-log 结论 13)。

## 核心策略:RealityKit-as-sim
- 在 Mac 上用 RealityKit 建一个**虚拟房间场景**(虚拟家具 + 虚拟角色 rig)。
- sim 和 AR **共用同一套代码**:rig / 动画 / 运动解析器 / 小脑 / 大脑 / 交互插件 / 场景图记忆。
- **只有两处不同**(= 两个可插拔端口):
  - **PerceptionSource 端口**:sim 里脚本生成假相机/mesh/手眼脸 vs 真实 ARKit(SceneReconstruction mesh + 企业摄像头 + Hand/Eye/Face 追踪)。
  - **Display 端口**:Mac 屏渲染 vs Vision Pro passthrough + AR。
- **迁移 = 换这两个插口**。不需要 Isaac Lab;全 Apple 原生;最大化代码复用。这正是 architecture-log 结论 10(可进化脊柱 + 可换叶子)的工程兑现。

## 端口驱动的迁移
5 个可插拔端口让阶段切换 = 换实现,不动脊柱:

| 端口 | Phase A(sim) | Phase B/C(真实) |
|---|---|---|
| **PerceptionSource** | 脚本生成的虚拟感知(虚拟相机 POV + 虚拟 mesh + 假手眼脸) | 真实 VP:SceneReconstruction mesh + 企业摄像头帧 + Hand/Eye/Face |
| **Display** | Mac 屏 RealityKit | VP passthrough + AR |
| Brain | 工作站 LLM(同) | 同 |
| MotionResolver | RealityKit rig(同) | 同 |
| MemoryService | Graphiti + Milvus(同) | 同 |

## 阶段

### Phase A — 纯 sim(RealityKit @ Mac)
- **做什么**:虚拟房间 + 占位 rig(Mixamo/RPM)+ 大脑(LLM)+ 小脑(OpenPI 推理先跑通,未微调)+ 交互插件 v1(坐 / 看屏 / 靠近)+ 场景图记忆(对模拟感知建图)+ 运动解析器。
- **不需要**:Vision Pro、企业 entitlement、真家具、干净美术资产。
- **感知源**:sim 脚本生成(虚拟相机渲第一人称 POV + 虚拟 mesh + 假手眼脸)。
- **交付**:感知→大脑→小脑→运动解析器→RealityKit 渲染 **闭环跑通**,Mac 屏上能看到虚拟角色在虚拟房间里反应。
- **价值**:**不被 entitlement 审批卡住**——大脑/小脑/交互的绝大部分在 A 阶段就能迭代。
- **出口**:角色能听指令坐虚拟躺椅、看虚拟屏、靠近你的虚拟化身;场景图能记忆虚拟房间。

### Phase B — 真实感知接入
- **做什么**:PerceptionSource 换成真实 VP 数据(mesh + 企业摄像头帧 + 追踪)。场景图从**真实房间**建图(SAM3.1 + VLM 上语义)。大脑/小脑吃真实感知。第一人称 POV 切到真实 mesh。
- **依赖**:企业 entitlement 到位。
- **仍可**:Mac 屏渲染(不必 VP AR)。
- **交付**:角色能感知**你的真实房间**(认得躺椅/床/桌/屏),记忆跨会话(物体恒常性)。
- **出口**:角色对真实房间有语义理解 + 持久记忆。

### Phase C — AR 渲染出
- **做什么**:Display 换成 VP passthrough + AR。角色渲染进真实空间。**affordance-fit 贴真家具**(HOSIG/CHOIS 粗姿态 → PhySIC 接触+穿透优化 → RealityKit 物理沉降 → IKcomponent 精修)。第一人称 POV 从角色眼位置渲真实 mesh。
- **交付**:角色坐在**你的真躺椅**、躺你的床、趴你的桌、看你的屏——**北极星落地**(呼应 project-vision-scenarios)。
- **出口**:愿景场景跑通。

### Phase D — 活感打磨 / 产品化
- **做什么**:alive-first 各层(behavior-director、baseline 状态机、reflex <200ms、连续微反应、ForesightService);鲁棒性/接地(抗幻觉、安全);干净 IP 资产(外包替换解包);企业/B2B 分发(Apple Business Manager);学术/资金对接。
- **出口**:可对外谈合作 / 报项目 / 演示。

## 从零起步:Phase A 工程里程碑 + 依赖路径

> 五条 track 的**里程碑级细化**(每 track 的步骤/依赖/核验到的 API 现状/风险)见 [`roadmap-tracks.md`](roadmap-tracks.md);本节是策略 + 关键路径。

### 核心原则:loop-first,全 stub 起步
不要先把任何子系统做到位。先搭**最薄的端到端闭环**(所有环节 stub),证明管线通,再逐个把 stub 换成真。这把"集成"——复杂项目头号杀手——提前到最便宜的阶段消解;且让每个子系统一接上就能在真上下文里测。

### 关键路径(依赖链)
```
M0 开发环境 + 5 端口骨架 ── unblocks 全部
      └─ M1 最小闭环(stub)── 证明管线
            └─ M2 ★Rig 定义★(gating:卡住整个 motion + 小脑动作空间 + 契约 C 具体化)
                  └─ M3 各子系统接真(并行):perception / brain / motion(A-C) / interaction
                        └─ M4 Phase A 出口
[独立长 sub-track] 小脑微调 = M2(rig)+ 自采数据管线 → 晚些插入;Phase A 先用 A/C 兜底
```

### Phase A 从零里程碑

**M0 · 开发环境 + 端口骨架**(复杂度:低)
- Xcode visionOS app 项目(SwiftUI + RealityKit)。
- **Phase A sim = visionOS Simulator @ Mac**:跑**同一个 visionOS app**,模拟器给模拟场景 + 模拟手眼追踪(无真摄像头)——最大化代码复用(sim↔真机同 app,只换端口)。备选:Mac 原生 RealityKit 场景。⚠️ 待核:visionOS 26 模拟器的场景/传感器模拟能力边界。
- 工作站:vLLM 起 LLM;OpenPI repo clone;websocket 服务器骨架。
- app↔工作站 websocket 链路(最小 ping/echo);后续上 10GbE。
- **5 端口定义成 Swift protocol**(PerceptionSource / Display / Brain / MotionResolver / MemoryService)——sim/真机可换全靠这层。

**M1 · 最小闭环(stub)**(复杂度:低)
- PerceptionSource stub:返回固定假观测。
- Brain stub:返回固定动作(如"挥手")。
- MotionResolver stub:动作 → 播一个 clip 或定 pose。
- Display:RealityKit 渲染角色。
- **产出**:角色渲染 + 按"大脑"决定播一个 canned 动作。**管线打通——此后每个子系统都有真上下文可测。**

**M2 · ★Rig 定义★(gating)**(复杂度:中高)
- 定虚拟人形 rig(Blender;占位用 Mixamo/RPM,但**锁定关节集**)→ 导出 USDZ 进 RealityKit。
- **定动作空间向量**(契约 C 的具体化):根 6-DoF + 身体关节(关节名/DOF/范围)+ 52 ARKit blendshape + 注视 + 手。⚠️ 关节数/旋转表示待 rig 定型后钉死(参考 SMPL-X/HumanML3D ~22–24 关节)。
- retarget:AnimationLibrary clip 映射到本 rig;IK 目标定义。
- **这是整个 motion track + 小脑 + 契约 C 的前置门槛,必须早做。** 它不定,小脑没法训、动作空间没法签契约。

**M3 · 各子系统接真(可并行)** — 全部 @Phase A(sim 感知)
- **perception**:sim 感知生成器(虚拟相机渲第一人称 POV + 假 mesh + 假手眼脸)→ 场景图搭建器消费 → 建虚拟房间场景图(契约 B schema)。
- **brain**:真 LLM(Qwen3 via vLLM)吃"场景图文本 + 用户语音"→ 输出 Decision JSON(意图);Graphiti + Milvus 记忆搭起。
- **motion**:运动解析器路由意图 → A/C 通道(脸 Audio2Face-3D、注视 IK、"坐"的 clip);**小脑未微调 → 先 A/C 兜底**。
- **interaction**:v1 插件(坐虚拟椅、看虚拟屏)——靠 sim 场景图的接近事件触发。

**M4 · Phase A 出口**
- Mac sim 里:角色能听你说话(语音→大脑)、环顾(感知)、坐虚拟躺椅(affordance + clip)、记住虚拟房间(场景图)。**闭环对模拟感知跑通**——大脑/交互/motion 的 ~80% 逻辑已被演练,且**没碰 VP / entitlement**。

### 并行性(spine + rig 就绪后)
M2 之后,brain / perception / motion(A-C) / interaction 四条可独立推进(都挂 stub 脊柱上测)。**小脑微调**是独立长 sub-track(依赖 M2 rig + 自采数据管线),Phase A 可缺省(A/C 兜底),微调好了再插入替换——不影响主线。

### Week 1 就能动手(零阻塞)
- **M0**:建 Xcode visionOS 项目 + 模拟器场景 + 工作站 websocket ping。
- **M2 启动**: sourcing 占位 rig(Mixamo/RPM),开始锁关节集。
- 这两件不依赖任何审批/硬件,今天就能开始,且解锁后续一切。

## Apple 原生组件映射(贯穿各阶段)
| 角色 | 技术 |
|---|---|
| sim + 渲染 | **RealityKit**(Mac @A;VP @C) |
| 感知 | **ARKit**(SceneReconstruction / WorldTracking+WorldAnchor / Hand/Eye/Face / CameraFrameProvider) |
| 端侧 fallback | Core ML / **MLX**(小 VLM/ASR) |
| 大脑 / 小脑 / 重模型 | 工作站 RTX Pro 6000,经 websocket / OpenPI 策略服务器 |
| 记忆 | Graphiti + Milvus(工作站) |
| 通信 | LAN **10GbE** |

## 现在就能做(不被卡)
**Phase A 全部**:占位 rig + Mac RealityKit 虚拟房间 + 工作站 LLM/OpenPI + 最简交互插件 + 模拟感知。**entitlement 批之前**就能迭代大脑/小脑/交互/记忆的绝大多数。

## 风险 / 门
- **Phase B 被 entitlement 审批卡**(已申请,$99,几周~月)→ A 阶段先跑,不被卡。
- **Phase C 的 affordance-fit(贴真家具)** 是最高 R&D 风险(无端到端解)→ 见 [animation-affordance.md](animation-affordance.md)。
- **sim→真实迁移的 gap**:虚拟感知干净、真实感知噪声大(mesh 略膨胀/遮挡/光照)→ PerceptionSource port 要做真实数据的去噪/对齐/重核验(呼应结论 9"执行前核验")。

## 关联
架构 / 可进化端口〔[architecture-log.md](architecture-log.md) 结论 10〕;交互插件〔结论 13①,待打磨〕;第一人称 POV〔结论 13③〕;大脑精细化〔[brain-agent-architecture.md](brain-agent-architecture.md)〕;动画/affordance〔[animation-affordance.md](animation-affordance.md)〕;契约〔[contracts-v1.md](contracts-v1.md)〕;总览〔[overview.md](overview.md)〕。
