# Phase A 开工前准备清单(2026-08)

> 勾选式,一个个解决。标记:🔴=长提前期/立刻并行 · 🟡=卡 day-1 · 🟠=决策 · 🟢=顺手 · ⚪=Phase A 不做。
> 状态:`- [ ]` 未做 → `- [x]` 完成。配套 [roadmap-phased.md](roadmap-phased.md) / [roadmap-tracks.md](roadmap-tracks.md) / [contracts-v1.md](contracts-v1.md)。

## 🔴 账号 / 申请(有提前期,立刻并行)
- [ ] **Apple Developer Program($99)** — developer.apple.com/programs/ 。entitlement + 上架都要。
- [ ] **企业摄像头 entitlement 申请**(已启动)— `com.apple.developer.arkit.main-camera-access` 申请表。**卡 Phase B,几周~月**;并行推进,别干等。
- [ ] **HuggingFace 账号 + token** — huggingface.co 。下大模型权重(π0/Qwen3/SAM3.1)要登录 + 同意 license。

## 🔴 大模型 / 权重预下载(几十~百 GB,提前下)
> 工作站磁盘先确认够(预留 **≥300 GB**)。各按 repo README 拉。
- [ ] **π0 / π0.5 权重** — `github.com/Physical-Intelligence/openpi`(openpi repo 指引拉 HF 权重)。小脑 base(Phase A 设计适配/wire;微调延后)。
- [ ] **大脑 LLM** — HF `Qwen/Qwen3-32B`(或 D3 benchmark 后定的)。⚠️ 显存见 D6。
- [ ] **SAM3.1** — HF Meta(2026-03 发布,✅ 核验)。Phase B 用,Phase A 可缓。
- [ ] **Audio2Face-3D** — NVIDIA 开源(2025-09)+ 社区 Apple Silicon 移植。脸通道(Phase A 如做脸)。
- [ ] **BGE-M3** — HF embedding 模型(检索)。
- [ ] (规划,Phase A 不下)**AMASS / BEAT / HumanML3D** — 运动数据,M-6 小脑数据管线用。先记存储位。

## 🔴 占位 rig(M-1 前置,卡整个运动 track)
- [ ] **sourcing 占位人形 rig** — Mixamo(mixamo.com,免费)或 Ready Player Me(readyplayer.me),**SMPL-X/HumanML3D 兼容**。
- [ ] **导出 USDZ**(给 RealityKit)。
- [ ] **锁关节集**(SMPL-X ~22–24 关节 + 根 6-DoF + ARKit 52 blendshape 槽)→ 契约 C 动作空间 + M-1 产出。
- [ ] 占位家具 USDZ(椅/床/桌/屏)+ 几个 clip(坐/挥手/idle)。

## 🟡 Mac 开发环境(卡 day-1)
- [ ] **Xcode 26** — Mac App Store / developer.apple.com。
- [ ] **visionOS 26 SDK** — Xcode 内装。
- [ ] **Reality Composer Pro** — 随 Xcode(USDZ 资产)。

## 🟡 工作站环境(RTX Pro 6000 96GB)
- [ ] **OS 钉死:Ubuntu 22.04**(OpenPI 仅测这版)或走 Docker(openpi `docs/docker.md`)。⚠️ 先定。
- [ ] **NVIDIA 驱动 + CUDA**(Blackwell 需较新驱动)。
- [ ] **uv**(OpenPI 依赖)— astral.sh/uv 安装。
- [ ] **OpenPI** — `git clone github.com/Physical-Intelligence/openpi` + `uv sync`(✅ `serve_policy.py` ws8000 核验)。
- [ ] **vLLM** — `pip install vllm`(✅ v0.26 核验,OpenAI 兼容)。
- [ ] **LangGraph ≥1.0.10** — `pip install langgraph`(⚠️ CVE-2026-27022:补丁 + 加密 checkpointer + 不外暴露)。
- [ ] **Graphiti ==0.29.3** — `pip install graphiti==0.29.3`(⚠️ 锁版本,别用 0.30 rc)。
- [ ] **Neo4j 5.26** — `docker pull neo4j:5.26` 或 neo4j.com。
- [ ] **Milvus 2.5 / Milvus Lite** — dev 用 Lite(`pip install pymilvus`);生产 docker。
- [ ] (可选)**Apple Foundation Models framework** — 端侧 LLM fallback(visionOS 26)。

## 🟡 网络
- [ ] **LAN**:Mac↔工作站 千兆起(10GbE 理想);线 + 交换机。
- [ ] **配固定 IP + 防火墙放行** websocket(8000)+ HTTP(vLLM 端口)。

## 🟠 开工前拍板的决策(附推荐)
- [ ] **D1 rig 骨骼** → 推荐 **SMPL-X 表示 + Mixamo/RPM render rig + 标准 retarget**。
- [ ] **D2 VLA base** → 推荐 **π0/π0.5**(微调成熟);GR00t N1.7 备(LoRA 坑)。
- [ ] **D3 大脑对话路线**(结论 待决策1)→ Phase A **默认 text+TTS**(Qwen3 + 外置 TTS);**早 benchmark** S2S(Qwen3-Omni / GLM-4-Voice)。
- [ ] **D4 action space 细节** → 旋转表示(推荐 **6D**)+ chunk K(推荐 **~32**,M-1 钉)。
- [ ] **D5 Phase A 屏幕感知** → 推荐 sim **虚拟屏**(真 Mac 屏 ScreenCaptureKit 留 Phase B)。
- [ ] **D6 96GB 显存同驻策略** → vLLM(LLM)+ OpenPI(小脑)+ SAM/VLM 各设 `--gpu-memory-utilization` 预留(防互 OOM);深推理 70B 按需起。

## 🟢 工程(顺手)
- [ ] **git 仓库** — 代码项目单开 repo(当前 `research/` 非 git)。定 monorepo(app + workstation)或分。
- [ ] **Xcode workspace**(visionOS app target + macOS 回退)+ 工作站 Python 项目骨架。
- [ ] **DEVENV.md** — SDK 版本 / Simulator 传感器边界(无真建图/手眼→脚本兜底)/ 资产导入流程 / 工作站版本锁。

## ⚪ Phase A 不做(别误做)
Vision Pro 硬件(sim 跑)· entitlement(已并行申请)· 真家具扫描 · 小脑微调 · 干净美术资产(外包留 Phase D)· affordance-fit · behavior-director 调优。

---

## 最小并行启动组(今天就动)
1. 🔴 entitlement 推进(长提前期)
2. 🔴 大模型预下载(磁盘 + 带宽)
3. 🔴 占位 rig sourcing + 锁关节集(M-1 前置)
4. 🟡 工作站 OS 钉死 + 环境装好

这四件并行 ≈ day-1 真能写代码的前提。其余可边做边补。
