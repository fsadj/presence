# 契约 v1:动作 token 词表 + 场景图 schema(2026-08)

> 大脑的**输出契约**(动作 token)与**输入契约**(场景图)。按用户要求**高度细粒化**;唯一边界:LLM **不**直接吐逐关节电机目标(会让动作崩 + LLM 不擅长)——粗动作(posture/locomotion)走**语义 token + 参数**,其余通道(脸/注视/手/情绪/呼吸/手势)全部**细粒、连续、可参数化**。这是 v1,标 [可调] 的后续再优化。

> ⚠️ **定位重梳(2026-08,对齐 architecture-log 结论 12)**:本文「契约 A 动作 token 词表」现在是**大脑的语义意图输出 / Phase-1 fallback 接口**,**不是**小脑的动作空间。小脑(连续动作 VLA,π0/GR00T 在你 rig 关节空间微调)的**连续动作空间 + 接口**见下方「契约 C」。即:大脑吐意图(契约 A)→ 小脑把意图+实时感知翻译成连续 rig 关节轨迹(契约 C)→ RealityKit。Phase-1(小脑未微调前)可让大脑意图直接走 MotionResolver(clip/IK)兜底。

---

## 契约 A:大脑意图输出(动作 token 词表,intent / Phase-1 fallback)

### 设计原则(细粒化如何落地)
- **粗动作语义化**:posture、locomotion 是语义 token(sit@<uuid>、walk-to <uuid>),Resolver 用片段库+物理兑现。LLM 不碰关节。
- **细通道全参数化**:脸(52 ARKit blendshape)、注视、手/指、情绪、呼吸、头、手势——**连续值、可叠加、带时序**,这是"活感"住的地方,且 LLM/生成模型在这些通道上**胜任**。
- **时序化**:每个动作带 duration/transition/offset,不是瞬时开关。
- **目标用 `ref`(统一引用,见下)**:支持世界点 / 物体-部件 / 语义角色 / 自身相对。
- **多通道并发 + 优先级**:通道间基本独立并发;冲突由**行为导演层**裁决。

### 目标引用 `ref`(贯穿所有通道)
```
ref = { type: "world_point", xyz:[x,y,z] }
     | { type: "object", uuid, part?: "seat"|"backrest"|"armrest_l"|"surface"|... }   // 引用场景图节点+部件
     | { type: "role", name: "user_face"|"user_torso"|"screen"|"self_hand_l"|... }
     | { type: "self_relative", offset:[dx,dy,dz] }
```

### Decision(大脑每拍吐一个,JSON)
```jsonc
{
  "ts": 1700000000.123,            // 时间戳
  "meta": {
    "priority": "reflex" | "normal" | "background" | "idle",
    "interrupt": "replace" | "queue" | "blend",   // 新决策如何处理在途动作
    "duration_ms": 1200, "transition_ms": 200,
    "confidence": 0.0..1.0, "reason": "用户皱眉+报错"   // 可观测/调试用
  },

  // —— 细通道(全参数化)——
  "gaze": {                        // 注视(IK)
    "target": ref, "mode": "lock"|"track"|"glance"|"saccade"|"avoid",
    "duration_ms": 800, "saccade_speed": "fast"|"normal",
    "pupil_dilation": 0.0..1.0, "squint": -1..1, "blink_rate": 0.0..1.0
  },
  "head": {
    "orient": ref|null, "tilt": -1..1,
    "nod":  { "count": 2, "speed": 0.6, "amplitude": 0.4 } | null,
    "shake":{ "count": 1, "speed": 0.5 } | null
  },
  "face": {                        // ARKit 52 blendshape 驱动(Audio2Face-3D 之外的表情层)
    "preset": "会心一笑"|"关切"|"惊讶"|"困惑"|null,
    "custom": { "browInnerUp": 0.3, "mouthSmileL": 0.5, "eyeSquintL": 0.2, /* …52 个 */ } | null,
    "intensity": 0.0..1.0, "asymmetry": -1..1, "transition_ms": 150,
    "micro": [ { "au":"cheekPuff", "at_ms":200, "dur_ms":300 } ],   // 微表情时序
    "breath_modulation": { "rate":1.1, "depth":1.2 }
  },
  "emotion": {                     // 连续内部情感态(驱动脸+身基线,跨拍持久)
    "valence": -1..1, "arousal": 0..1, "dominance": -1..1, "drift_rate": 0.1
  },
  "speech": {
    "text": "第42行少了个冒号", "prosody": { "pitch":1.0, "rate":0.95, "energy":0.6,
                                    "emotion":"关切", "emphasis":["42行","冒号"] },
    "voice_id":"default", "bargeable": true, "ssml": null, "start_offset_ms": 0
  },
  "gesture": [                     // 手臂/手势(可多组并发;C 生成式)
    { "arm":"r"|"l"|"both", "kind":"generate"|"clip",
      "desc":"摊手解释、掌心朝上、轻柔",          // kind=generate 时给 MotionLCM 的描述
      "clip_id":"wave"|"point"|null,              // kind=clip 时
      "target": ref|null, "intensity":0.0..1.0,
      "sync_to_word":"42行", "blend_in_ms":120, "blend_out_ms":180 }
  ],
  "hands": [                       // 手指(抓/指/捏,细)
    { "hand":"r", "pose":"point"|"grip"|"pinch"|"open"|"custom",
      "target": ref|null, "reach":0.0..1.0, "custom_finger_joints":{...}|null }
  ],
  "body": {                        // 身体微动/呼吸/重心(细)
    "breath_rate":0.9, "breath_depth":1.0,
    "weight_shift": ref|null, "fidget":"手指轻敲"|null
  },

  // —— 粗动作(语义 token + 参数;Resolver 兑现)——
  "posture": {                     // 坐/躺/趴/站(A 片段库 + affordance 拟合)
    "intent": "sit@<uuid>"|"lie@<uuid>"|"lean@<uuid>"|"stand"|"crouch"|null,
    "contact_regions": ["seat","backrest","feet"],   // 指定接触区(贴家具拟合用)
    "lean_angle": -30..30, "settle": true            // 物理沉降
  } | null,
  "locomotion": {                  // 走动(A 片段 + navmesh)
    "target": { "type":"object","uuid":"<uuid>","where":"nearby_front" } | navmesh_point,
    "speed":"stroll"|"walk"|"hurry", "arrival_radius_m":0.3,
    "facing": ref|null, "path_constraint": null
  } | null
}
```

### 优先级/冲突模型(归行为导演层)
- **通道并发**:gaze+gesture+speech+face 可同时发。
- **同通道**:新命令按 `meta.interrupt`(replace/queue/blend)处理;`transition_ms` 控混。
- **跨通道冲突**:导演层裁决(例:gesture 目标 vs posture 接触区冲突 → 压制会穿模的手势)。
- **idle/空**:某通道 `null` = 该通道交回**行为基线**连续层(不冻住)。
- **reflex 优先**:priority=reflex 的决策可打断一切(用户插话→停 TTS+转向)。

### 我替你定的子选择(细粒取向)
- 粒度:粗动作语义、细通道参数化(边界如上)。✅
- 通道集合:gaze/head/face/emotion/speech/gesture/hands/body + posture/locomotion + meta。[可调加通道]
- **手势**:默认 `kind:"generate"`(自由文本→MotionLCM 即兴),重要/高频手势可 `kind:"clip"`。✅ 走 C。
- **目标坐标系**:三种全支持(world_point / object+part / role / self_relative);大脑优先用 role 与 object+part(语义),需要精确时用 world_point。
- **posture 家具引用**:用场景图节点 `<uuid>`(把动作词表和场景图绑在一起)。✅

---

## 契约 B:场景图 schema(感知/记忆 → Brain)

### 设计原则(细粒化如何落地)
- **家具部件级**:椅子拆 seat/backrest/armrest/leg——坐姿要分"坐坐面、靠椅背"。
- **富字段**:几何(bbox+mesh+SDF)、语义(类别+CLIP+开放词表别名)、affordance(按部件)、时序(首/末见、置信度、观测次数、新鲜度)、快照。
- **动态实体**(用户、角色自身、移动物体)与静态家具同图。
- **WorldAnchor 键 + 多索引**(类别/CLIP/affordance)。
- **时序/置信度模型**支撑"记忆=信念、执行前核验"。

### 节点类型
```jsonc
// 1) 家具/物体(静态为主)
{
  "uuid": "...", "anchor_uuid": "WorldAnchor_UUID_xxx",   // 跨会话重定位钥匙
  "category": "armchair", "open_vocab_aliases": ["躺椅","单人沙发"],
  "clip_feature": [ ... ],                                 // 开放词表检索
  "transform": mat4, "bbox": AABB,
  "mesh_ref": "meshes/m001.usd",                           // 碰撞/渲染
  "sdf_ref":  "sdf/s001.mha",                              // [可调] 预算 SDF,给 affordance 拟合接触优化用
  "parts": [ PartNode ],                                   // 部件级(下)
  "affordance": { "object": ["sit-able","lean-able"],
                  "scores": { "sit":0.9,"lie":0.2,"lean":0.7,"grasp":0.1 } }, // 连续得分,启发式起步、可学
  "material": "fabric", "movable": false, "mass_kg": 12,
  "confidence": 0.92, "first_seen": t0, "last_seen": t1,
  "observation_count": 37, "staleness_score": 0.1,         // 越高越可能过期 → 触发核验
  "snapshots": [ frame_ref ]                               // 3D-Mem 式记忆快照
}
// 1b) 部件 PartNode(家具细粒)
{ "id":"seat"|"backrest"|"armrest_l"|"armrest_r"|"leg"|"surface"|"headrest",
  "transform": mat4, "bbox": AABB, "mesh_ref":..., "sdf_ref":...,
  "affordance": ["sit-support"|"lie-support"|"lean-support"|"back-support"|"arm-support"],
  "contact_offset_m": -0.02 }   // animation-affordance 的 ARKit mesh 膨胀修正,每部件美术调

// 2) 区域/房间
{ "uuid":"room_living", "anchor_uuid":..., "boundary":..., "label":"客厅", "contains":[uuid,...] }

// 3) 用户(动态)
{ "uuid":"user", "transform": head_pose_mat4,
  "gaze": { "target": ref, "ray":[dx,dy,dz], "confidence":... },
  "hand_poses":[l,r], "emotion":{valence,arousal}, "activity":"coding"|"idle"|...,
  "screen_attention": ref|null, "last_updated": ts }

// 4) 角色自身(动态)
{ "uuid":"self", "transform":..., "current_posture":"sit@<uuid>",
  "current_action":"speaking"|"idle"|"walking", "body_state":{...} }

// 5) 移动物体(动态)
{ ...同物体, "movable":true, "velocity":[...], "last_seen":ts, "trajectory":[...] }
```

### 边(关系)
```jsonc
{ "from":uuid, "to":uuid,
  "type": "on"|"behind"|"in_front"|"left_of"|"right_of"|"inside"|"reachable_from"|
          "supports"|"blocks"|"adjacent"|"belongs_to"|"same_as",
  "params": { "distance_m":1.2, "angle_deg":30 },
  "confidence":0.8, "valid":[t_start,t_end], "dynamic":false }
```

### 时序/置信度模型(支撑"记忆=信念、执行前核验")
- 每节点 `staleness_score = f(now - last_seen, observation_count, movable)`。
- **执行前核验触发**:大脑命令某动作引用某节点时,若 `staleness_score > θ` 或 `confidence < φ` → 触发对该区域的**实时重感知**再执行(architecture-log 结论9)。
- 时序边 `valid` 支撑"昨天杯子在哪"类查询。

### 索引
- `anchor_uuid`(跨会话重定位)、`category`(过滤)、`clip_feature`(Milvus 向量)、`affordance`(查"可坐的")。

### 大脑怎么消费(相关性编译器)
- 整图太胀不喂大脑。**编译器**按当前决策需要 + 角色/用户位置,挑**相关子图**(附近物体、被引用目标、近期变化)→ 序列化成**紧凑结构化文本**喂 LLM。〔architecture-log〕

### 我替你定的子选择(细粒取向)
- **家具部件级** ✅(坐/靠要分清坐面与椅背)。
- **affordance**:物体级标签 + **连续得分**(启发式起步,后续可学)。✅
- **几何**:bbox(必有)+ mesh_ref(碰撞/渲染)+ **SDF_ref(预算,给动画接触优化)**。[可调:SDF 是否预算]
- **关系**:空间+语义+时序三类,空间关系部分临时算(不全显式存)。
- **动态实体**:用户/自身/移动物体同图,带 last_updated/velocity/trajectory。
- **时序/置信度**:staleness_score + 核验阈值 θ/φ。[θ/φ 可调]

---

## 两契约的交叉点(必须一致)
1. **动作 `posture.sit@<uuid>` / `locomotion target.uuid` → 引用场景图节点 uuid**(动作词表消费场景图)。
2. **场景图 `parts[].sdf_ref` + `contact_offset` → 喂给动作那步的 affordance 接触拟合**(场景图喂动画,animation-affordance)。
3. **`ref.type="object", part:"seat"` → 场景图 PartNode**(注视/手势指向家具部件)。
三者 uuid/部件命名必须**统一**。

## 后续可调(已标 [可调]):SDF 是否预算、θ/φ 核验阈值、手势 generate vs clip 比例、是否加更多通道(如眼睑/瞳孔独立通道)、affordance 是否从启发式升为学得。

---

## 契约 C:小脑连续动作空间 + 接口(2026-08,对齐结论 12;部分待 rig 定型/大脑精细化)

> 小脑 = 连续动作 VLA(π0/π0.5/GR00T N1.7,在你 rig 关节空间微调)。这是**主动作通路**;契约 A 的离散 token 降为 fallback。**大脑→小脑条件化机制已定**(C.3:NL 意图串 token-prefix,见 brain-agent-architecture);下到可开工颗粒度仅剩 **rig 关节定义定型**(决定 C.1 维度)——故 C.1 标【待定】处待 rig 绑定后钉死。

### C.1 动作空间(连续 rig 关节向量)
一个 action chunk = 短时序(len K 步,~16–64,@10–30Hz)的 rig 状态向量,逐帧:
- **根**:6-DoF(世界/hip 根平移+旋转)
- **身体骨架**:**采用标准人形骨骼 SMPL-X/HumanML3D(~22–24 关节,非自定义)**的旋转(6D/轴角)— 与 AMASS/BEAT/HumanML3D 运动数据原生对齐,微调数据免造
- **脸**:52 ARKit blendshape 权重(复用契约 A 的 face 通道定义,但作连续值)
- **注视**:gaze 方向(2D/yaw-pitch,或 3D 向量)
- **手**(可选):手指关节 DOF
- 【已定】采用标准 SMPL-X/HumanML3D 骨架(关节列表已知);【待定】仅剩旋转表示(6D vs 轴角)与 chunk 长度 K。

### C.2 小脑 → RealityKit(输出契约)
- 工作站跑 VLA 策略服务器(OpenPI 式 websocket),流式吐 action chunk。
- Mac 侧解码:关节轨迹 → IK 目标 / blendshape / 根变换,@90Hz 插值平滑喂 RealityKit rig。
- 与契约 A 的 MotionResolver 共享同一 rig 写入口(见 C.4)。

### C.3 大脑 → 小脑(条件化接口)— 已定(2026-08,见 brain-agent-architecture)
大脑把**意图**作为条件传给小脑(非逐关节指令)。候选意图字段(契约 A Decision JSON 的子集):`speech.text + prosody`、`emotion`(valence/arousal)、`posture.intent`(sit/lie/lean/stand)、`gesture.desc`、`gaze.target`。
- **条件化机制 = 自然语言意图串 token-prefix**:大脑吐契约 A Decision JSON → 经轻量**意图编译器**(intent compiler)序列化成结构化 NL 串 → 作为 π0/π0.5/GR00T N1.7 的 `prompt` 输入。这是它们的**原生接口**——无需自写 cross-attention 适配器、无需重训 backbone。
- **小脑作用域(重要)**:VLA 只扛**身/大动作通路**(posture / locomotion / 大手势 / 身基线)。**细通道**(脸 blendshape / 注视 / 手指 / 呼吸)**不经小脑**,仍走契约 A + MotionResolver(Audio2Face-3D、IK 等)。即小脑 ≠ 全身,只管大动作;脸/眼/手是确定性映射,各走各的通道。

### C.4 与契约 A/B 的关系
- 契约 A(离散意图 token)= 大脑语义层 + Phase-1 fallback(意图→MotionResolver→clip/IK,不经小脑)。
- 契约 B(场景图)= 小脑与大脑共同的感知输入(SDF/affordance 喂 affordance-fit;part ref 喂接触目标)。
- MotionResolver = 统一的 rig 写入口:小脑轨迹 / fallback clip / IK / 生成式 都经它仲裁后写 rig(仲裁=运动解析器 port,见 animation-affordance)。

---

## 契约 D:感知 + 感知总线(PerceptionSource → Observation + 感知管线;2026-08)

> 把"感知数据单元"与"感知处理管线数据流"定成正式 schema,让 PerceptionSource 端口(sim stub / 真实 ARKit)可换、下游(场景图/大脑/小脑)统一消费。Phase A 用 sim stub 产 Observation;Phase B 换真实 ARKit provider(同 schema)。

### D.1 Observation(感知总线统一数据单元,PerceptionSource 输出)
异步吐(`AsyncStream<Observation>`),一拍 = 一帧观测:
```
Observation {
  ts:            Float64            // 秒,单调
  device_pose:   Transform          // 头显/相机世界位姿(RealityKit Transform)
  pov_frame:     FrameRef | null     // 角色第一人称 POV 渲染帧(喂小脑 VLA + VLM);见 D.3
  mesh_anchors:  [MeshAnchor]        // 房间几何;Phase A=sim 合成,Phase B=SceneReconstruction
  hand_poses:    [HandPose]          // 用户左右手关节(ARKit HandAnchor)
  eye_gaze:      GazeRay | null      // 用户注视(ARKit EyeTracking):{origin, dir, target_ref, conf}
  face_blendshapes:[Float32×52]|null // 用户脸(ARKit Face)——⚠️VP 外摄像头看不见用户自己脸 → 多为 null,情绪走语音/注视/手势
  user_state:    UserState | null    // {presence, proxemics_m, speaking, motion_level}
  confidence:    Float32             // 整拍置信
  source:        Enum{sim, real}     // 端口来源标记
}
MeshAnchor { uuid, transform, geometry_ref,语义占位 null(Phase B 由管线填) }
HandPose  { chirality, joint_transforms[×N], conf }
```
**Phase A**:PerceptionSource stub 返回一份固定假 Observation(脚本生成 sim POV + 假 mesh + 假手眼脸)。
**Phase B**:换 ARKit provider 实装(SceneReconstruction→mesh_anchors;CameraFrameProvider→pov 候选/或 POV 渲真实 mesh;Hand/Eye/Face→对应字段)。⚠️ CameraFrameProvider 仅像素+内外参,无烘焙深度。

### D.2 感知处理管线数据流(Phase B,工作站;Phase A 跳过/桩)
Observation → SAM3.1 → VLM 标注 → 2D→3D lift → 场景图更新(契约 B)。中间格式:
```
Detection { mask_id, bbox, mask_ref, clip_feature }          // SAM3.1 出(实例分割,Object Multiplexing)
Labeling  { mask_id, open_vocab_labels:[str], conf }          // VLM(Qwen3-VL 级)开放词表标注
Lift3D    { mask_id, anchor_uuid, bbox_3d, face_labels[] }    // 反投影 mask→mesh per-face 标签→聚合 3D 实例(HOV-SG 式;⚠️ HOV-SG 非维护库,须移植/重写)
→ 融合成 契约 B SceneNode 增量(upsert:uuid/anchor/clip/category/affordance/confidence/first_seen/last_seen)
```
**物体恒常性**:WorldAnchor uuid 跨会话融合;staleness/confidence 喂"执行前核验"(契约 B 字段 + 见 E.5)。

### D.3 POV 渲染 feed(第一人称视觉通道,喂小脑 VLA + VLM)
- 角色眼位置虚拟相机 → 离屏渲染 → `FrameRef{width,height,encoding(JPEG/WebP/raw),binary_ref,ts,pose}` 经 wire(E.1)送工作站。
- ⚠️ **RealityKit 无第一方离屏 render-to-texture API**(roadmap-tracks 发现 2):Mac 用 `ARView.snapshot()` 轮询 / 自写 Metal 离屏 pass 兜底;VP 更难,可能降分辨率/事件触发。
- **双视觉通道**:POV 渲染(实时视觉/核验,喂小脑+VLM)+ 场景图(记忆/信念,喂大脑)——呼应架构结论 9/13③。

---

## 契约 E:传输 + 运动仲裁 + 插件 + 记忆 + 多时间尺度衔接(2026-08)

### E.1 wire / 传输格式(app ↔ 工作站,LAN 10GbE)
**统一消息信封**:
```
Envelope { msg_type: Enum, seq: UInt32, ts: Float64, encode: Enum{json,msgpack,binary}, payload_ref }
```
**通道**(各定 encode + 速率):
| 通道 | 方向 | 内容 | encode | 速率 |
|---|---|---|---|---|
| perception | app→ws | Observation(帧+mesh+手眼脸) | binary(帧 JPEG/WebP + mesh 增量 + 控制字段 msgpack) | ≤30Hz(帧可降采样) |
| control/llm | app→ws(HTTP) | 大脑请求(场景图子图+用户态+对话) | json(OpenAI 兼容,/v1/chat/completions) | 1–5Hz |
| action | ws→app | 小脑 action chunk(契约 C) | OpenPI 原生格式 ✅(receding horizon,~50 步) | OpenPI 节奏 |
| decision | ws→app | 大脑 Decision JSON(契约 A) | json | 1–5Hz |
| audio | 双向 | TTS 流 / ASR partial | binary(opus/pcm) | 流式 |
| graph_updates | ws→app | 场景图增量(upsert/delete) | msgpack | on-change / 5–10Hz |
**实现**:Mac 侧 `URLSessionWebSocketTask`(ws)+ `URLSession`(HTTP);工作站 OpenPI `serve_policy.py`(ws8000,✅)+ vLLM(HTTP)。⚠️ ws 须自实现重连/心跳/背压。

### E.2 MotionResolver 多源仲裁 I/O(运动解析器 port)
**输入**(多 channel 并发):
```
ChannelInput {
  source:    Enum{brain_decision, cerebellum_chunk, reflex, plugin, baseline}
  channel:   Enum{gaze, face, body, posture, locomotion, hands, breath}
  payload:   Any  // decision 子集 / action chunk / IK target / clip ref / blendshape 集
  priority:  Int  // 对齐契约 A meta.priority
  interrupt: Enum{none, replace, queue, blend}  // 同 channel 中断规则(契约 A)
  blend_weight: Float32
  ts:        Float64
}
```
**仲裁规则**:① 同 channel 按 priority+interrupt(replace/queue/blend);② 跨 channel 冲突压制(如 hand 抓握时压 idle fidget、locomotion 时压坐姿);③ reflex 旁路(高优先直插)。**导演层**(behavior-director)在 MotionResolver 之上做"此刻哪个微反应该显/该压/怎么混"的裁决(IA-4,自建)。
**输出**(统一 rig 写入,@90Hz):
```
RigWrite { joint_targets:[Float×N], blendshape_targets:[Float32×52], root_transform: Transform, ik_targets:[IKAnchor], ts }
```
**缓冲**:action chunk @≤30Hz → 环形缓冲 + @90Hz 插值(receding horizon);reflex <200ms 直插。

### E.3 InteractionPlugin schema(交互插件,架构结论 13①)
```
InteractionPlugin {
  id:        str
  trigger:   { type: Enum{proximity, gaze, event, timeout}, target: object_class|anchor_uuid|role, range_or_cond }
  precond:   { affordance_min: Float, state_req: [str] }   // 复用契约 B affordance.scores
  actions:   [ { name, binding: Enum{cerebellum_intent, clip_plus_affordance_fit, ik, procedural}, params } ]
  exit:      { cond }
  priority:  Int
}
```
**注册**:`PluginRegistry`(push 模式,接近/事件触发激活)。**发现**:大脑查场景图 affordance + 接近事件 → 得当前可用 `actions` → SayCan 乘 affordance 选(契约 A posture.intent)。**binding 双路径**(强依赖契约 C 收口):`cerebellum_intent`(小脑微调后)或 `clip_plus_affordance_fit`(fallback,A 路线)。

### E.4 记忆 schema(Graphiti 时序 KG + Milvus 向量)
Graphiti Pydantic Entity/Edge(✅ 0.29.3 核验)映射契约 B + 扩展:
```
SceneEntity(Entity):   uuid, anchor_uuid, category, clip_feature(→Milvus), affordance, transform_summary   // = 契约 B SceneNode
PartEntity(Entity):    parent_uuid, part_type(seat/backrest/...), sdf_ref, contact_offset
EpisodicEvent(Entity): ts, type{perception|interaction|dialogue|user_action}, summary, participants[uuid], location_uuid
PersonaState(Entity):  mood, energy, relationship, current_posture  // 跨 tick 持久(大脑 LangGraph checkpointer + Graphiti)
UserModel(Entity):     preferences, habits, name, relationship_history
Edges: on/behind/inside/reachable(空间) | same_as(物体恒常) | supports/blocks | temporal(Graphiti 原生时序:事件有效期)
```
**检索**:Milvus/BGE-M3(clip_feature + category + affordance 向量,dense+sparse+multi-vector);Graphiti 时序查询("昨天杯子在哪")。**相关性编译器**(BM-4):抽相关子图 → 紧凑文本喂大脑。

### E.5 多时间尺度衔接(缓冲/插值/核验规格)
| 环 | 频率 | 衔接策略 |
|---|---|---|
| 渲染 | 90Hz | ← MotionResolver @90Hz(插值自 ≤30Hz chunk;RealityKit) |
| 感知 ingest | ~30Hz | → 处理 5–10Hz(降采样 + 事件触发)→ 场景图更新 |
| 认知 | 1–5Hz | Decision → MotionResolver;LangGraph tick |
| 反射 | <200ms | 旁路大脑,本地直插 MotionResolver(高优先) |
| 记忆 | 跨会话 | staleness>θ/confidence<φ 触发**执行前核验**(局部重感再执行,契约 B) |
**缓冲原语**:环形缓冲(action,receding horizon)、最近有效(感知)、staleness 核验(记忆)。**可调**:[可调] θ/φ 阈值、降采样率、chunk K。

---

## 契约总览(A–E)
- **A** 大脑意图输出(Decision JSON)· **B** 场景图(记忆/信念)· **C** 小脑动作空间+接口
- **D** 感知 Observation + 感知管线(实时视觉/核验)· **E** 传输/仲裁/插件/记忆/时序衔接
- **Phase A 需要的**:A/B/C + D.1(Observation stub)+ E.1(wire)+ E.2(MotionResolver)+ E.3(plugin)+ E.4(记忆)+ E.5(衔接)。D.2(感知管线)Phase A 桩、Phase B 实装。
