# 大脑(Agent 架构)选型调研:具身 AR 伴侣的「大脑」(2026-08)

> 本文是 [architecture-log 结论 4/12] 与 [contracts-v1 契约 A/B/C] 的**研究配套**:回答"大脑这一层用哪种 agent 架构"。所有外部事实 2026-08 经 WebSearch/WebFetch 复核;每项标 ✅(已核)/ ⚠️(未证实/存疑)。
>
> **范围**:大脑 = 慢审议层(1–5Hz,工作站)。它的人格/对话/规划/工具使用/记忆**全部**由 LLM(Qwen3/DeepSeek/GLM)承担;VLM=眼(工具)、VLA=小脑(连续动作,契约 C)、行为/反射=更底层(本文不涉及)。
>
> **本文产出**:候选谱 + 一套**推荐大脑架构(复合)** + **契约 C 的开放项收口**(大脑→小脑条件化机制)。

---

## 1. 一句话结论

> **采用 LangGraph 编排 + ReAct/SayCan/ECoT 复合推理环 + Graphiti(时序 KG)+Milvus(向量)双层记忆 + Letta 式「自管记忆」工具层;大脑→小脑用「自然语言意图串」作 token-prefix 条件化(π0/π0.5/GR00T N1.7 的原生接口),不另写 cross-attention 适配器。** 人格不外采商用角色引擎,靠系统提示词 + 持久人格记忆自承载。

---

## 2. 大脑的 Job-to-be-Done(需求清单)

承自 architecture-log 结论 2/3/4 + 结论 11(活感优先)+ 契约 A/B/C:

| # | 能力 | 约束 | 现有栈 |
|---|---|---|---|
| B1 | **人格一致性**:跨会话稳定的"她" | 长期记忆 + 系统提示 | 待建 |
| B2 | **流式对话**:中文、低延迟、barge-in | 首 token <300ms、可打断 | Qwen3-Omni / GLM-4-Voice 候选 |
| B3 | **意图/规划**:主动、多步、前瞻(ForesightService) | 1–5Hz,可中断 | 待建 |
| B4 | **工具调用**:VLM(感知)、场景图检索、ForesightService、可插拔 affordance | 结构化 JSON 输出稳 | Qwen3/DeepSeek/GLM(已定) |
| B5 | **记忆**:情景/语义/时序,跨会话;**物体恒常性** | "记忆=信念,实时感知=核验" | Graphiti+Milvus(已定) |
| B6 | **接地/抗幻觉**:动作前对场景图核验 | staleness>θ 重感 | 契约 B 已定 |
| B7 | **输出**:语义意图 Decision JSON(契约 A)→ 小脑(契约 C)+ MotionResolver 兜底 | 粗动作语义、细通道参数化 | 契约 A 已定 |
| B8 | **连续内心独白**:没说话也在"想"→ 微动作涓流 | 一直跑热(活感优先) | 待建 |
| B9 | **安全/鲁棒降级**:输出可观测、可回退 | confidence 字段 + fallback | 契约 A meta 已定 |

**关键**:大脑是**单角色、单实例、长生命周期**(不是多 agent 协作),但内部有多套**循环模式**(对话/规划/回忆/空想)。这决定了下面"编排框架"的选型。

---

## 3. 候选谱(按类别,带 2026 复核状态)

### 3.1 记忆架构(Memory)

| 候选 | 状态 | 它是什么 | 与本项目 Graphiti+Milvus 的关系 |
|---|---|---|---|
| **Generative Agents**(Stanford Park 2023) | ✅ 论文 [arxiv 2304.03442](https://arxiv.org/abs/2304.03442),~6900 引用;学术基线,无产品 | 记忆流(memory stream)+ 反思(reflection)+ 规划三件套;按 recency×importance×relevance 检索 | **思想借鉴**:反思与"重要性评分"可叠加在 Graphiti 之上。本项目场景图的 `staleness_score` 即其 recency 工程化版本。不替代 Graphiti。 |
| **Letta / MemGPT** | ✅ v0.16.8(~2026 中),Apache 2.0,~17x release;[letta-ai/letta](https://github.com/letta-ai/letta)、[letta.com](https://www.letta.com/)。2026 新动向:**Memory Models**(JUN 2026,memory-native RL 训练)、**Context Constitution**(APR 2026) | OS 式分层(core/archival memory),LLM **用工具调用自管上下文**(自我决定何时 recall/forget/archieve);单一 perpetual thread | **取其"自管记忆"范式,不引入其 runtime**:把 Letta 的"记忆即工具"模式作为大脑的一组 tool(perceive/recall/reflect/forget)实现,后端仍写 Graphiti+Milvus。这样大脑可控、可观测,不被锁死在 Letta 的 agent loop 里 |
| **Mem0** | ✅ 2026-04 新算法(single-pass ADD-only,multi-level user/session/agent),[mem0ai/mem0](https://github.com/mem0ai/mem0)、[mem0.ai](https://mem0.ai/)。商业+开源 | 可插拔记忆 API 层:抽取事实 → 三级(user/session/agent)存储;新算法一次 LLM 调用完成 | **功能与 Graphiti 重叠**:Mem0 主推"事实抽取 + 向量检索";Graphiti 已有"时序 + 关系"。**不引入 Mem0**——避免两套记忆真相源。可借鉴其"agent 生成的事实=一等公民"思路 |
| **A-MEM**(agiresearch) | ✅ NeurIPS 2025,~900 引用;[arxiv 2502.12110](https://arxiv.org/abs/2502.12110)、[agiresearch/a-mem](https://github.com/agiresearch/a-mem) | Zettelkasten 式:笔记构造 + 链接生成 + 记忆自演化(动态再组织) | **思路对齐 Graphiti 的演化**:A-MEM 的"链接自生成 / 记忆演化"等价于 Graphiti 的边自动抽取;Graphiti 多了**时序双轴**(valid_time + transaction_time),对"昨天杯子在哪"类查询更强。不另引入 |
| **Zep / Graphiti**(已选) | ✅ ~28.9k⭐(2026-07),Apache 2.0,Neo4j 后端;[getzep/graphiti](https://github.com/getzep/graphiti)、[arxiv 2501.13956](https://arxiv.org/abs/2501.13956)(323 引用) | 时序知识图谱:实体/关系/事实/episode + **valid 时间窗**——知道"现在何为真、过去何为真、何时改变" | **核心持久层,保留**。直接吃契约 B 场景图 + 跨会话情景。是"记忆=信念"的信念容器 |
| **Milvus + BGE-M3**(已选) | ✅ Milvus 持续维护;BGE-M3 多语(含中文)向量 | 向量检索:CLIP/语义/open-vocab 别名检索 | **保留**:场景图 `clip_feature` 索引 + 历史情景向量召回。Graphiti 管"结构化真值",Milvus 管"相似度召回",互补不冲突 |

**记忆层结论**:本项目**已有正确的栈**(Graphiti+Milvus),不替换;**只缺一层"记忆自管理工具"**——把 Letta/A-MEM 的"反思/演化/自遗忘"作为大脑可调用的工具补上。详见 §4。

---

### 3.2 编排 / Agent 框架(Orchestration)

| 候选 | 状态 | 它是什么 | 是否契合"单角色大脑" |
|---|---|---|---|
| **LangGraph** | ✅ v1.0 稳(2024 末/2025),[LangChain 生态](https://github.com/langchain-ai/langgraph);2026 被广泛认作"有状态图编排"首选。⚠️ 注意:[2026-03 CSA 研究简报](https://labs.cloudsecurityalliance.org/research/csa-research-note-langchain-langgraph-vulnerabilities-202603/) 报告 checkpoint 持久层有两条独立漏洞——上线须打补丁 + 加密后端 | StateGraph:节点=函数、边=条件路由;**checkpointer** 在每节点后序列化全图状态,支持 pause/resume/HITL/崩溃恢复 | **强契合**。大脑的"对话/规划/回忆/空想"循环天然是有状态图;持久状态支撑跨 tick 的"内部情感态/活跃意图"。**推荐** |
| **AutoGen**(Microsoft) | ⚠️ 0.4 是**最后大版本**;官方[迁移指南](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)已把用户导向新的 **Microsoft Agent Framework**;社区 fork **AG2** 延续。2026 起新项目不建议入坑 | 多 agent 对话式协作(conversational multi-agent) | **不契合**:本项目是单角色;AutoGen 焦点是多 agent 群聊。且生态在迁移、风险高。**排除** |
| **CrewAI** | ✅ v1.15.9(2026-07-29),Apache 2.0;[crewAIInc/crewAI](https://github.com/crewAIInc/crewAI)、[docs.crewai.com](https://docs.crewai.com/)。Flows 管有状态工作流 + Studio 可视化 | 角色化(role-based)多 agent:Agent(role/backstory/goal)+ Task + Crew + Flow | **过杀**:CrewAI 的甜区是"研究员/作家/评论员"这种多专长协作。本项目只有一个"她"。**排除**(但 Flow 的状态机思想与 LangGraph 同源) |
| **Microsoft Agent Framework** | ✅ 微软 2026 主推方向(承接 AutoGen);[learn.microsoft.com/agent-framework](https://learn.microsoft.com/en-us/agent-framework/) | 企业级 agent 编排,与 Azure/语义内核集成 | ⚠️ 偏 Azure 生态绑定,本项目工作站在本地、跨设备(Mac+VP);**不选** |

**编排层结论**:**LangGraph**。把大脑建模成一个 StateGraph(详见 §4)。AutoGen 在退场,CrewAI 解错题。

---

### 3.3 推理环(Reasoning Loop)

| 候选 | 状态 | 思想 | 本项目怎么用 |
|---|---|---|---|
| **ReAct**(reason+act) | ✅ 经典(Yao 2022);2026 仍是工具型 agent 的默认骨架 | 思考→行动→观察循环;LLM 边想边调工具 | **工具调用主线**:大脑调 VLM/场景图/ForesightService 走 ReAct。LangGraph 节点天然实现 |
| **Reflexion**(Shinn 2023) | ✅ 经典;自我反思 + 语言强化 | 失败→自然语言反思→存入记忆→下次更聪明 | **接 B6 抗幻觉**:动作失败时(核验不一致/工具报错),写一条 Reflexion 笔记进 Graphiti。不作为主环,作为"教训"通道 |
| **SayCan**(Google Ahn 2022) | ✅ [say-can.github.io](https://say-can.github.io/)、[arxiv 2204.01691](https://arxiv.org/abs/2204.01691),~3500 引用 | **LLM 的语言概率 × affordance 值** = 选下一个动作;让 LLM 的"想做"被"能做"束缚 | **直接落地"记忆=信念,实时=核验"**:大脑选 posture/locomotion 时,候选 token 的 LLM 概率乘以场景图该节点的 `affordance.scores`(契约 B)。这正是 B6 接地的数学化 |
| **ECoT**(Zawalski 2024) | ✅ [embodied-cot.github.io](https://embodied-cot.github.io/)、[arxiv 2407.08693](https://arxiv.org/abs/2407.08693),364 引用 | VLA 在动作前生成多步推理链(task → plan → subtask → motion → boundary) | **借给大脑的"内心独白"**:大脑在吐 Decision 前先走 ECoT 风格的 plan/subtask/motion 内推(满足 B8 活感),只是输出语言而不是 VLA 内嵌(那是小脑的事) |
| **Plan-and-Execute / Tree-of-Thoughts** | ✅ 成熟 | 树搜索式规划 | ⚠️ 过重;1–5Hz 大脑用不上全树搜索。前瞻由 ForesightService 承担,大脑消费其结果即可 |

**推理层结论**:**ReAct(主线)+ SayCan(接地)+ ECoT 风格内心独白(活感)+ Reflexion(教训)** 四件套,都嵌进 LangGraph 节点里。

---

### 3.4 角色 / 对话引擎(Character)

| 候选 | 状态 | 它是什么 | 本项目如何用 |
|---|---|---|---|
| **Inworld AI** | ✅ [inworld.ai](https://inworld.ai/);"#1 Realtime Voice AI",Realtime TTS-2 首 chunk **<130ms**,$5–10/M 字符。2026 仍领跑 TTS arena | 云端角色 SaaS:人格设计 + voice clone + 流式 TTS + 行为 steering | **只取 TTS 层**(中文流式 + 低延迟 + 可打断);大脑的人格不交给它。⚠️ 需评估其中文质量与延迟是否优于 Qwen3-Omni/GLM-4-Voice 的内置声学 |
| **Convai** | ✅ [convai.com](https://www.convai.com/);2026 推 Convai Sim(no-code 仿真);被评"技术上最 впечатляющий" XR 角色平台 | 云端对话 SaaS:云处理 voice/text → 角色回复;Action-aware(UE5/Unity 集成) | ⚠️ 强 XR/游戏集成,但云依赖与"工作站本地大脑"冲突。**不选**(其架构思想可参考) |
| **character.ai** | ✅ 2026 仍运营,Plus 订阅;但[社区评价情感深度/记忆不如竞品](https://www.reddit.com/r/CharacterAIrunaways/);封闭 | 消费级角色聊天产品 | **不适用**:封闭、不可私有部署、不控人格细节。**排除** |
| **Simular AI** | ✅ [simular.ai](https://www.simular.ai/);⚠️ **关键澄清:它是"computer-use agent"公司(Agent S,开源自主操作电脑),不是角色/对话引擎**。与本项目无关 | 计算机自动化 agent | **排除**(列出来是为纠正"它是角色引擎"的可能误解) |
| **Hume AI EVI** | ✅ [hume.ai](https://www.hume.ai/empathic-voice-interface);EVI 2 voice-to-voice 基础模型;Series B $50M;数字伴侣是官方 use case | 共情声学:实时检测 + 生成带情感的语音 | ⚠️ 西文为主,中文情感声学待验。**可参考其"情感驱动语音"思想**,但本地化首选 Qwen3-Omni |
| **Qwen3-Omni**(候选大脑) | ✅ [QwenLM/Qwen3-Omni](https://github.com/QwenLM/Qwen3-Omni);[arxiv 2509.17765](https://arxiv.org/html/2509.17765v1);vLLM-Omni 2026-07 优化 | 端到端多模态(text+audio+image+video)→ 流式 text+speech | **若 B2 优先**:大脑直接换 Omni(对齐 log 待决策 1 的选项 2)。代价是丢一些深推理 |
| **GLM-4-Voice**(候选大脑) | ⚠️ 未在本轮检索中作为头部返回(需后续单独核);已知为中文 S2S 强候选 | 中文实时语音对话 | 待 benchmark vs Qwen3-Omni |

**角色/对话层结论**:**人格不外包**(用系统提示词 + Letta 式持久人格记忆自承载,理由:角色细节、可演化、可私有)。**对话与声学**靠 Qwen3-Omni 或 GLM-4-Voice 当大脑本身(若优先级是"说话像真人"),TTS 兜底用 Inworld。Convai/character.ai/Hume/Simular 不进入主线。

---

### 3.5 2026 开源"具身伴侣大脑"栈

| 候选 | 状态 | 备注 |
|---|---|---|
| **NVIDIA ACE + Audio2Face-3D** | ✅ Audio2Face-3D 已开源(Apache,[arxiv 2508.16401](https://arxiv.org/abs/2508.16401));ACE 作为 NIM 微服务套件 | **已在动画层采用**(log 结论 5);不是大脑,是脸。大脑与之解耦:大脑出 `speech.text+prosody`,Audio2Face 兑现脸 |
| **没有现成的开源"完整数字伴侣大脑"** | ⚠️ | 2026 没有一个开源项目把"人格+对话+具身规划+时序记忆+VLA 调度"打成一站式大脑;都得自己拼。**这就是本文要交付的复合架构** |

---

## 4. 推荐大脑架构(复合)

### 4.1 总览(单角色、单实例、LangGraph StateGraph)

```
            ┌──────────────────  LangGraph StateGraph(单一持久图) ──────────────────┐
            │                                                                              │
   感知     │  ┌──────────┐   ┌─────────────┐   ┌──────────┐   ┌────────────┐  │  Decision(契约 A)
  总线  ───▶│  │ Perceive │──▶│  Interpret  │──▶│  Reason  │──▶│  EmitAct   │──┼──▶ JSON
 (场景图    │  │  (工具)   │   │ +Recall 工具│   │ (ECoT 内 │   │ +SayCan 接 │  │     │
  快照/态   │  │ VLM/FVL/  │   │ Graphiti +  │   │  心独白 + │   │  地(乘 aff │  │     ▼
  势)       │  │ screen    │   │ Milvus 检索 │   │  ReAct 工│   │  ordance)  │  │   MotionResolver
            │  └──────────┘   └─────────────┘   │  具调度) │   └────────────┘  │     + 小脑(契约 C)
            │                  Reflexion 通道 ──▶│          │                   │
            │  (失败/不一致写教训)                └──────────┘                   │
            └──────────────────────────── checkpointer(每节点) ─────────────────┘
                                                │
                                  持久态:emotion / active_intent / persona_state
                                  (跨 tick,支撑 log 结论 3 "决策= f(记忆,内部态,意图)")
```

### 4.2 各层选型与"现在采纳 vs 推迟"

| 层 | 立即采纳 | 推迟 / 试验性 |
|---|---|---|
| **编排** | **LangGraph**(StateGraph + checkpointer,加密后端) | — |
| **记忆持久层** | **Graphiti**(时序 KG,结构化真值)+ **Milvus/BGE-M3**(向量召回)——已选 | — |
| **记忆自管工具层** | **自建一组 tool**:`perceive / recall / reflect / forget / update_belief`,后端写 Graphiti+Milvus。借鉴 **Letta**(范式)+ **A-MEM**(演化/链接)+ **Generative Agents**(importance 评分 + 反思) | Letta-as-runtime 整体引入(不必要,锁死循环) |
| **推理环** | **ReAct**(主线工具调度)+ **SayCan**(选 posture/locomotion 时乘 `affordance.scores`)+ **ECoT 风格内心独白**(背景 tick,出活感)+ **Reflexion**(失败写教训进 Graphiti) | Tree-of-Thoughts 全搜索 |
| **人格** | 系统提示词 + 持久人格记忆(Graphiti 中存"她是谁/她的偏好/与用户的关系史")。**不外采角色 SaaS** | — |
| **对话/声学** | 主线走 **Qwen3**(text)+ 外置 TTS;若 B2 优先则大脑换 **Qwen3-Omni / GLM-4-Voice** | Inworld 作 TTS 兜底(若其中文/延迟优于内置) |
| **VLM(眼)** | Qwen3-VL / UniVR-34B 当工具,被 ReAct 调用 | — |
| **ForesightService** | 作为工具被大脑 Reason 节点消费(预测用户下一步 → 注入到活跃意图) | — |
| **可观测/安全** | 每决策 `meta.confidence + reason`;`reflex` 优先级可打断 | — |

### 4.3 大脑怎么消费 Graphiti+Milvus(对齐契约 B)

1. **相关性编译器**(契约 B 已定):每 tick 按当前决策需要 + 角色/用户位置,从 Graphiti 抽**相关子图** → 紧凑结构化文本喂 LLM(不喂整图)。
2. **Recall 工具**:LLM 在 Reason 节点可主动 `recall(query)` → 命中 Milvus 向量(Graphiti 的 `clip_feature`)→ 取回 episode/事实。
3. **Reflect 工具**:周期/触发式把近期 episode 总结成高层事实(Generative Agents 的 reflection)写入 Graphiti。
4. **Forget/Re-evaluate 工具**:对 `staleness_score > θ` 的信念,标记"待核验";**不直接删**(记忆=信念),由 §6 的"执行前核验"流程处理。
5. **时序查询**:Graphiti 的 valid_time 窗直接答"上次躺椅在哪""昨天聊到第几行代码"。

---

## 5. 大脑→小脑 条件化接口(契约 C 开放项收口)

> 这是契约 C 的【待打磨】项:"条件化机制:文本 prompt 前缀 / cross-attention / token-prefix?" —— **本文给出明确推荐**。

### 5.1 现状复核:当前主流 VLA 实际支持什么

| VLA | 语言条件化机制(2026 复核) | 证据 |
|---|---|---|
| **π0**(Physical Intelligence) | **语言 token 作 prefix** 进共享 backbone(PaliGemma VLM),**action expert 的 action token 通过 prefix-attention-mask 对语言/视觉/状态 token 做 cross-attention**;flow-matching 头去噪出连续动作。[arxiv 2410.24164](https://arxiv.org/html/2410.24164v1) | ✅ HuggingFace blog、Steven Gong、pi.website 均一致描述 |
| **π0.5** | 同 π0;另**co-train 出文本输出**(plan/correction),让模型对新指令更泛化。[arxiv 2504.16054](https://arxiv.org/pdf/2504.16054) | ✅ |
| **GR00T N1.7**(NVIDIA) | **language-conditioned**:单策略以(视觉 + **语言指令**)为输入、跨本体出动作;Apache 2.0。[HF nvidia/GR00T-N1.7-3B](https://huggingface.co/nvidia/GR00T-N1.7-3B)、[Isaac-GR00T](https://github.com/Nvidia/Isaac-GR00T) | ✅ |
| **OpenPI 推理服务** | policy server 的输入就是 `prompt`(自然语言指令)+ 图像 + 本体状态;**LoRA 微调**支持你 rig 关节空间。[Physical-Intelligence/openpi](https://github.com/Physical-Intelligence/openpi) | ✅ |

**关键事实**:三大 VLA(π0/π0.5/GR00T N1.7)**原生支持的接口就是「自然语言指令字符串」**。所谓"cross-attention / token-prefix"是它们**内部实现细节**(语言如何与动作 token 融合),不是用户该重写的接口。自己加 cross-attention 适配器 = 重新训练 backbone = 不划算。

### 5.2 推荐:**语言 token-prefix 条件化(大脑侧 = 序列化意图串)**

**机制**:
1. 大脑在 EmitAct 节点吐出**契约 A 的 Decision JSON**。
2. 一个**意图编译器**(大脑侧轻量组件)把 Decision 的子集**序列化成一条结构化自然语言串**,作为 VLA 的 `prompt` 输入:

   ```
   [persona: 温柔的陪伴者,情绪=关切(0.6)]
   [posture: 坐在躺椅 <uuid=armchair_01> 上,略前倾]
   [speech: "第 42 行少了个冒号"]
   [gesture: 右手指向屏幕,轻柔]
   [gaze: 锁定 screen]
   当前情境:用户皱眉、屏幕上有红色报错。
   ```

3. 这条串被 VLA 当 token-prefix 消费;VLA 在你 rig 关节空间微调过(LoRA),把意图翻译成连续 rig 轨迹 → OpenPI websocket → RealityKit(对齐契约 C.2)。

**为什么是这个**:
- ✅ **零额外训练**:用现有 π0/π0.5/GR00T N1.7 的原生接口。
- ✅ **可演化**:意图串 schema 在你手里,加字段不改 VLA。
- ✅ **可观测/可调试**:串可 log、可 diff、可回放。
- ✅ **对齐活感**:连续内心独白(ECoT)产生的"她想干嘛"自然落到串里。

**为什么不选另两个**:
- ❌ **自写 cross-attention 适配器**:要重训 VLA backbone;且当前 VLA 内部已有 cross-attention(prefix-mask),再叠一层是重复造轮。
- ❌ **embedding-level token-prefix**(把意图编成向量直接拼):脱离语言、不可解释、对齐难。

### 5.3 重要边界:VLA 只扛"主动作通路",细通道仍走契约 A

VLA(π0/GR00T 等)是为**机器人操作/移动**设计的——它管"身体粗动作 + 大手势 + 脸的 affect 基线"。**契约 A 的细通道**(ARKit52 blendshape 精确表情、注视 IK、手指精确 pose、呼吸/微动)**不进 VLA**,继续走 MotionResolver:

| 通道 | 走哪 |
|---|---|
| posture / locomotion / 大手势 / 身体基线 affect | **VLA**(语言 token-prefix 条件化) |
| face blendshape(脸/唇)| **Audio2Face-3D**(由 `speech.text+prosody` 驱动,log 结论 5 已定)+ 大脑 `face` 通道叠加 |
| gaze(注视)| 程序化 IK(`gaze.target`) |
| 手指精 pose | 契约 A `hands` 通道 |
| 呼吸/重心/微动 | 契约 A `body` 通道 + 行为基线 |

→ **MotionResolver 是统一仲裁入口**(契约 C.4 已定):VLA 轨迹 / fallback clip / IK / 生成式 都经它仲裁后写 rig。大脑→小脑只是其中一条主通路。

### 5.4 待打磨项(诚实)

- 【待 rig 定型】意图串里 `posture.sit@<uuid>` 的精确 schema、是否带 SDF 接触目标。
- 【待微调】LoRA 数据计划(连续动作,见 data-and-iteration)。
- 【待 benchmark】同一条意图串在不同 VLA(π0.5 vs GR00T N1.7)上的中文/陪伴感表现差异。

---

## 6. 鲁棒性 / 接地 / 抗幻觉与安全("反诈")

承自 log 结论 9("记忆=信念,实时感知=核验")与契约 B 时序模型:

| 机制 | 怎么实现 | 落点 |
|---|---|---|
| **执行前核验**(主防线) | 大脑 Decision 引用某场景图节点时,若 `staleness_score > θ` 或 `confidence < φ` → **先调 perceive 工具重感该区域**再执行;不一致则优雅失败/重感 | 契约 B 已定 θ/φ;大脑工具层实现 |
| **SayCan 接地**(选动作时) | 候选 posture/locomotion 的 LLM 概率 × `affordance.scores`;低 affordance 动作被压 | §3.3 / §4 |
| **结构化输出** | Decision JSON schema 严格 + confidence + reason;解析失败 → fallback 到 MotionResolver + 默认 idle | 契约 A meta |
| **Reflexion 教训** | 失败/不一致 → 写进 Graphiti 一条 reflexion 笔记;下次类似情境被 recall | §4 |
| **可打断** | `priority=reflex` 的 Decision 可打断一切(用户插话→停 TTS+转向);barge-in 由本地 <200ms 层处理 | 契约 A 优先级模型 |
| **降级链** | 70B 深推理跪了 → 轻量 LLM 兜底 → MotionResolver + idle;VLA 跪了 → MotionResolver clip/IK 兜底(契约 A 的 Phase-1 fallback 一直在) | log 结论 5/12 |
| **安全** | 人格系统提示词里写硬约束(不泄私、不教危险行为、儿童安全);工具层做权限白名单;输出做敏感词过滤 | 大脑 prompt 工程 |

**关键**:接地不是"加一个 grounding 模块",而是**三处协同**——选动作时(SayCan 乘 affordance)+ 执行前(staleness 核验)+ 失败后(Reflexion 教训)。

---

## 7. 待决策项(下次从这里挑)

1. **大脑优先级**(log 待决策 1,仍开放):(1) LLM-core+VLM 工具(默认,最聪明/人格最稳)vs (2) **Qwen3-Omni / GLM-4-Voice 当大脑**(说话最像真人/最低延迟)vs (3) VLM 包揽(最弱)。本文推荐架构对三个选项都成立(都是 LangGraph+同一记忆栈);区别在 Reason 节点跑哪个模型 + 是否省一层 TTS。
2. **VLA 选 π0.5 还是 GR00T N1.7** 作为小脑起点?(都 Apache 2.0、都语言条件化、都 LoRA 可微调你 rig)。建议:**先用 OpenPI(π0.5)** 起步(生态最成熟、policy server 现成),GR00T N1.7 作备选(更强 reasoning VLA,跨本体数据多)。
3. **意图串 schema** 5.2 的精确字段与 token 预算(待 rig 定型后细化)。
4. **中文 TTS/语音**:Qwen3-Omni 内置声学 vs Inworld($)/GLM-4-Voice 的中文延迟与情感盲测。
5. **Letta 范式 vs 自建工具层**:是否值得把 Letta runtime 整体拿来用(节省自管记忆逻辑)vs 只借其模式(本文推荐,可控性更高)。
6. **记忆"反思"触发**:周期 vs 事件驱动 vs 混合(推荐混合:低频周期 + 失败/重大事件触发)。

---

## 8. 来源(关键 URL,2026-08 复核)

**记忆**
- Generative Agents:[arxiv 2304.03442](https://arxiv.org/abs/2304.03442)
- Letta/MemGPT:[letta.com](https://www.letta.com/)、[github letta-ai/letta](https://github.com/letta-ai/letta)(v0.16.8)
- Mem0:[mem0.ai](https://mem0.ai/)、[github mem0ai/mem0](https://github.com/mem0ai/mem0)
- A-MEM:[arxiv 2502.12110](https://arxiv.org/abs/2502.12110)、[github agiresearch/a-mem](https://github.com/agiresearch/a-mem)
- Zep/Graphiti:[getzep.com](https://www.getzep.com/)、[github getzep/graphiti](https://github.com/getzep/graphiti)、[arxiv 2501.13956](https://arxiv.org/abs/2501.13956)

**编排**
- LangGraph:[github langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)、CSA 漏洞简报 [labs.cloudsecurityalliance.org](https://labs.cloudsecurityalliance.org/research/csa-research-note-langchain-langgraph-vulnerabilities-202603/)
- AutoGen:[github microsoft/autogen](https://github.com/microsoft/autogen)、迁移指南 [learn.microsoft.com/agent-framework](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)
- CrewAI:[crewai.com](https://crewai.com/)、[docs.crewai.com](https://docs.crewai.com/)(v1.15.9)

**推理**
- SayCan:[say-can.github.io](https://say-can.github.io/)、[arxiv 2204.01691](https://arxiv.org/abs/2204.01691)
- ECoT:[embodied-cot.github.io](https://embodied-cot.github.io/)、[arxiv 2407.08693](https://arxiv.org/abs/2407.08693)

**角色/对话**
- Inworld:[inworld.ai](https://inworld.ai/)
- Convai:[convai.com](https://www.convai.com/)
- Hume EVI:[hume.ai/empathic-voice-interface](https://www.hume.ai/empathic-voice-interface)
- Qwen3-Omni:[github QwenLM/Qwen3-Omni](https://github.com/QwenLM/Qwen3-Omni)、[arxiv 2509.17765](https://arxiv.org/html/2509.17765v1)、vLLM-Omni [vllm.ai/blog/2026-07-01-qwen3-omni-optimization](https://vllm.ai/blog/2026-07-01-qwen3-omni-optimization)
- NVIDIA ACE/Audio2Face-3D:[developer.nvidia.com/blog/nvidia-open-sources-audio2face-animation-model](https://developer.nvidia.com/blog/nvidia-open-sources-audio2face-animation-model/)、[arxiv 2508.16401](https://arxiv.org/abs/2508.16401)

**VLA(小脑条件化,§5)**
- π0:[arxiv 2410.24164](https://arxiv.org/html/2410.24164v1)、HF blog [huggingface.co/blog/pi0](https://huggingface.co/blog/pi0)
- π0.5:[pi.website/blog/pi05](https://www.pi.website/blog/pi05)、[arxiv 2504.16054](https://arxiv.org/pdf/2504.16054)
- OpenPI:[github Physical-Intelligence/openpi](https://github.com/Physical-Intelligence/openpi)、[pi.website/blog/openpi](https://www.pi.website/blog/openpi)
- GR00T N1.7:[HF nvidia/GR00T-N1.7-3B](https://huggingface.co/nvidia/GR00T-N1.7-3B)、[github Nvidia/Isaac-GR00T](https://github.com/Nvidia/Isaac-GR00T)、[developer.nvidia.com/isaac/gr00t](https://developer.nvidia.com/isaac/gr00t)
- VLA 综述:[arxiv 2505.04769](https://arxiv.org/html/2505.04769v1)

**关联项目文件**
- [architecture-log.md](architecture-log.md) 结论 4(LLM=大脑,VLM=眼)/ 结论 12(大脑-小脑双系统)/ 结论 9(记忆=信念,实时=核验)
- [contracts-v1.md](contracts-v1.md) 契约 A(意图 JSON)/ B(场景图)/ C(小脑连续动作 + 待打磨的条件化接口)
