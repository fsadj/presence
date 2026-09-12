# 中国具身 AI 学术地图

> Vision Pro 虚拟角色项目(原生 Swift,具身认知 + AR 渲染)国内对接全景。中文,一手源核验。覆盖清华大学、北京大学、上交/中科院/中科大、上海AI Lab/智源 BAAI。

## 来源映射

本文档合并自以下原文件:

| 原文件号 | 路径 | 覆盖范围 | 原核验日期 |
|---|---|---|---|
| **02** | `research/_archive/02-china-tsinghua-embodied-ai.md` | 清华大学总览(AIR/CS/自动化系/IIIS/EIR/Apple 缺口) | 2026-08 |
| **03** | `research/_archive/03-china-pku-embodied-ai.md` | 北京大学(CVDA/EPIC/SLAM/BIGAI/ACIR) | 2026-07/08 |
| **07** | `research/_archive/07-china-academic-sjtu-cas-ustc.md` | 上交 + 中科院(CASIA/ICT/SIA)+ 中科大 | 2026-07 |
| **09** | `research/_archive/09-china-shanghai-ailab-baai.md` | 上海 AI Lab(SHLAB)+ 北京智源 BAAI | 2026-07 |
| **10** | `research/_archive/10-china-tsinghua-deepdive.md` | 清华深度补强(赵昊组/GUAVA/贾珈/胡事民/EIR 结构) | 2026-07 |

**状态标注约定**：✅ = 联网核验(≥2 源含官网)；⚠️ = 待核；`[unconfirmed]` = 未直接确认。星级 ★ 来自原 10 文件的相关度评级,保留。

**工具限制注**（来自原 09/07）：本会话 WebFetch 对 `arxiv.org` / `baai.ac.cn` 被屏蔽,部分 arXiv ID/标题取自搜索元数据或聚合站,建议普通浏览器复核。两个标记可靠的 arXiv ID（InternVLA-M1 `2510.13778`、EchoMimic `2407.08136`）经多独立源佐证。

---

## 全局纠错 / 勘误汇总（接触前必读）

以下纠错来自原文件的核验,均已在正文中体现,此处集中列出以防踩坑:

1. **孙富春 Sun Fuchun 在计算机系(CS),不是自动化系**——他做过自动化博后,但讲席在 CS / 智能技术与系统国家重点实验室。(原 02)
2. **张亚勤 Zhang Yaqin 的院系是 AIR(智能产业研究院)**,不是 "AGE" 或 I-AIGI。I-AIGI(人工智能国际治理研究院)是薛澜领衔的独立政策机构,张亚勤只是其学术委员。(原 02)
3. **王海峰 Wang Haifeng 不是清华 AI/机器人 PI**——著名的王海峰是百度 CTO(哈工大 CS 校友),与清华无关。从清华地图删除。(原 02)
4. **邬霞不在 PKU**——她在北京理工大学计算机学院(脑机接口与类脑智能研究中心主任),前北师大人工智能学院。(原 03)
5. **紫东太初(Zidong Taichu)由 CASIA 王金桥 Wang Jinqiao 领衔**——非王亮或黄凯奇(二人皆为 CASIA 相邻领域资深人物)。(原 07)
6. **中科大张兰 Zhang Lan 做数据隐私/数据交易——非具身AI或数字人**。USTC 真正的具身/多模态 PI 是查正军、张燕咏、张世武。(原 07)
7. **Jinguo Liu 常被称"具身AI与机器人研究所所长",但他现在沈阳理工大学**,此前在 SIA——勿列为现任 SIA 教研。(原 07)
8. **Hi3DGen 是字节主导论文**（repo `github.com/bytedance/Hi3DGen`,arXiv `2503.22236`）,赵昊合著——非清华主导。(原 10)
9. **EchoMimic canonical repo 是 `antgroup/echomimic`**——主所有者是蚂蚁集团,SHLAB 相关者合著。按蚂蚁集团主导合作处理,非纯 SHLAB 产品。(原 09)
10. **"InternSpatial" 非独立模型名**——空间 grounding 工作以 InternVLA-M1 发表。(原 09)
11. **IIIS 官方实验室名为 VAR**(视觉与机器人实验室);某聚合帖出现的 "EVAR Lab" `[unconfirmed]`。(原 02)
12. **AnyTeleop(arXiv 2307.04577)不安全归因 MVIG**——基于搜索结果不能确认是卢策吾的,已排除。(原 07)
13. **"SJTU-HVR" 作为实验室名未坐实**——已列已核验 SJTU 人形/VR 实体替代。(原 07)

---

# 第一部分 · 清华大学（合并原 02 + 10）

> 02(清华总览)与 10(深度补强)约 60–70% 重叠,已合并去重。清华是中国 avatar/数字人/具身AI 实力最集中的一站。

## 概要与战略判断

- **张亚勤 Zhang Yaqin** ✅——AIR(智能产业研究院)创始院长,2025–2026 仍活跃(博鳌 2026、2025 图书奖)。AIR 三支柱:多模态 LLM、自动驾驶、生物智能(AI4Science)——**非**数字人/XR。名下无公开 "Life/3IVF/X-Discover" 具身项目(该说法不准)。
- **对 Vision Pro 虚拟角色项目最相关:赵昊组(AIR)**——talking-heads、3DGS avatar、neural rendering、具身 agent 高产;**在招实习/博后**。
- **最佳"纯数字人"实验室:刘烨斌(自动化系)**——DNA-Rendering 数据集、国家杰青、~1.73 万+ 引。
- **清华 2025-11-30 正式成立校级具身智能与机器人研究院(EIR)**——最强制度信号。
- **Apple Vision Pro / visionOS 学术研究在清华几乎空白**——仅一所设计学院 workshop。**真实空白,潜在合作角度**。
- **AIR 明确面向产业**,开放招实习/博后/工程师。

## 1. AIR(智能产业研究院)

AIR 位于清华科技园 C 座。**面向产业设计**("大学与企业创新双引擎")。招聘 `airhr@air.tsinghua.edu.cn`、办公 `airoffice@air.tsinghua.edu.cn`。站:https://air.tsinghua.edu.cn/

### 1.1 张亚勤 Zhang Yaqin ✅
- 清华智能科学讲席教授；**AIR 创始院长**；中国工程院外籍院士。前百度总裁(2014–2019)、前微软(16 年)。[AIR 新闻](https://air.tsinghua.edu.cn/info/1007/2421.htm)；[ZGC 论坛简介](https://www.zgcforum.com.cn/zh2026/guest/t23805/962610)
- **AIR 三支柱**(其书《智能涌现》):多模态大模型、自动驾驶、生物智能。
- 具身AI 非其个人焦点,但 AIR 托 DISCOVER Lab / 赵昊 的具身工作。公开预言"10 年后机器人可能比人多"。力推 **Real2Sim2Real(RSR)** 为"原子与比特的桥梁"。[AIR RSR 帖](https://air.tsinghua.edu.cn/info/1007/2354.htm)
- **不在** I-AIGI(薛澜领衔);张仅为其学术委员。
- **名下未发现公开 Life/3IVF/X-Discover 具身项目**。

### 1.2 赵昊 Zhao Hao 组 ★★★★★（最高相关度）✅
- 助理研究员/助理教授,博导,智源学者。清华 EE 本/博；前 Intel Labs China；PKU 博后。**2026-02 任极智嘉 Geek+ 首席科学家**。
- 联系 **zhaohao@air.tsinghua.edu.cn** ——**招博后与科研实习(≥6 月),从实习池选博士**。
- **工作**:talking-head 合成、3DGS avatar、neural rendering、3D/4D 生成、animatronic 表情脸、具身 VLA、自动驾驶场景生成。明言其渲染工作"直接影响数字人产业"。
- **DISCOVERSE** 高保真仿真器(物理引擎 + 并行渲染,多尺度室内外机器人任务)——赵昊主导公开揭幕,与周谷越 DISCOVER Lab 合作。
- **签名论文(2024–2026)**:
  - **SyncTalk**(talking-head + GS,CVPR 2024);**SyncTalk++**(T-PAMI 2025)——同步 talking head + Gaussian 渲染,**近直接契合 LLM 驱动 Vision Pro avatar**
  - **Morpheus**(神经驱动 animatronic 表情脸 + 情绪控制,RSS 2025)
  - **Ultraman**(超快单图 3D 人体纹理,MVA 2026)
  - **NeAR: Coupled Neural Asset–Renderer Stack**(CVPR 2026)
  - **PAM**(姿态-外观-动作引擎 sim-to-real HOI 视频,CVPR 2026)
  - **Light-X**(生成式 4D 视频渲染含相机 + 光照控制,ICLR 2026);**Relit-LiVE**(重打光视频,SIGGRAPH 2026);**UniVidX**(SIGGRAPH 2026)
  - **Automated Synthesis of Facial Mechanisms for Conversational Animatronic Robots**(RSS 2026 Best Paper Finalist)
  - **Scalable Training of 3DGS**(ICML 2026);SA-GS(CVM 2025);Rip-NeRF(SIGGRAPH 2024);SlimmeRF(3DV 2024 Best Paper)
  - **Hi3DGen**(高保真 3D 几何 via Normal Bridging,ICCV 2025,arXiv **2503.22236**)——⚠️ **字节主导 repo,赵昊合著**,非清华主导
  - 具身/VLA:**Dexora**(ICRA 2026 Best Paper Finalist)、UniDex(CVPR 2026)、RoboChemist(CoRL 2025)
- **Vision Pro 相关度 ★★★★★**——覆盖全栈:实时 GS 渲染、talking head、单图 avatar 重建、重打光、LLM 驱动具身。
- **产业开放度 ★★★★★**——公开实习/博后；与梅赛德斯-奔驰(仿真/渲染)、比亚迪合作;个人任 Geek+ 首席科学家。
- 源:[发表页](https://sites.google.com/view/fromandto)；[Geek+ 任命](https://finance.sina.cn/stock/jdts/2026-02-25/detail-inhnzfcs2325107.d.html)

### 1.3 DISCOVER Lab(协同视觉与机器人实验室)✅
- 主任 **周谷越 Zhou Guyue**——副研究员/副教授；兼**清华-戴尔乐具身协同机器人联合研究院**院长；**求之科技(QiuZhi Tech)**首席科学家(AIR 孵化 2023,A 轮 >1 亿美元)。前大疆。MIT TR35 亚太 2021。HKUST 博士 2014。
- **焦点**:机器人、Real2Sim2Real(RSR)、具身操控。建 **AIRBOT** 标准化双臂移动操作硬件；DISCOVERSE 仿真平台。
- **命名注**:DISCOVER Lab = 周谷越;赵昊跑自己相邻组(上),合作 DISCOVERSE。两者不同。
- **产业开放度 ★★★★★**——戴尔乐(厦门戴尔乐新能源)资助;孵化求之科技;戴尔乐联合研究院是新型产业资助实验室模板。
- 其他 AIR PI:**周浩 Hao Zhou**(副研究员)—LLM 与 AI4Science(分子生成/药物设计),前字节,非数字人;**马维英**(前字节 AI Lab 负责人,前 MSRA)—AIR 科学家。
- 源:[AIR DISCOVER 页](https://air.tsinghua.edu.cn/info/1046/1199.htm)；[求之科技融资](https://www.sohu.com/a/1044130792_120109837)；[163 报道](https://www.163.com/dy/article/L0Q7O22605568W0A.html)；[DISCOVERSE CSDN 访谈](https://devpress.csdn.net/v1/article/detail/137705500)
- 另:AIR 无锡创新中心主任 **陈亦伦 Chen Yilun**——**ApolloFM** 室外具身智能/自动驾驶基础模型。AIR 与**地瓜机器人(D-Robot)**、**毫末智行(HAOMO.AI)**深度签约;与北师大/华东师大共建具身智能未来教育联盟。

## 2. 计算机系(CS)

### 2.1 唐杰 Tang Jie — GLM / KEG / 华智冰 ✅
- 长聘教授,**系副主任**,WeBank 讲席教授,ACM/AAAI/IEEE Fellow。**KEG(知识工程实验室)主任**;清华-中国工程院知识智能联合-KG 联合实验室主任。[CS 页](https://www.cs.tsinghua.edu.cn/info/1111/3486.htm)；[KEG 站](https://keg.cs.tsinghua.edu.cn/jietang/)
- 设计 **GLM** 家族(GLM-130B → GLM-4.5 2025-07 → **GLM-5.2 2026-06 开源**,现为顶流 AI 编码模型)。**智谱 AI(Zhipu)创始人/首席科学家**(拟 IPO)。
- **华智冰(Hua Zhibing)**——中国首个原创虚拟学生,2021"入读"计算机系(唐杰组);脸/声/模型生成;基于悟道 2.0(1.75T 参数),BAAI + 智谱 + 小冰。[清华新闻](https://www.tsinghua.edu.cn/info/1175/84993.htm)
- 兼 **BAAI 学术副院长**,**悟道 WuDao 奠基 lead**。
- **相关度 ★★★★☆**——GLM 是可信国产 LLM 骨干(角色认知大脑);其组自己不做实时 GS 渲染。产业开放度极高:实际运营清华↔智谱管线。

### 2.2 贾珈 Jia Jia — 情感计算 + 数字人 ✅
- 长聘教授,国家杰青,国家级青年人才。**华智冰共创者**。
- **焦点**:数字人生成、情感计算、人机语音交互。~7183 Google Scholar 引。音乐驱动 avatar 舞蹈生成(ACM MM 2018 奖)。
- **相关度 ★★★★☆**——情感层 + 语音正是交互 avatar 的"具身认知"半边;与图形组配渲染半边天然。
- 站:hcsi.cs.tsinghua.edu.cn/jiajia

### 2.3 胡事民 Shi-Min Hu — 图形 & Jittor ✅
- 教授,**中国科学院院士(2023 当选)**,国家杰青。北京市可视媒体智能处理与内容安全工程研究中心主任；**清华-腾讯互联网创新技术联合实验室**主任;虚拟现实技术与系统国家重点实验室(北航)主任。创 **Jittor** 深度学习框架。
- 组站 cg.cs.tsinghua.edu.cn(图形与几何计算组)。
- **工作**:计算机图形、可视媒体、neural rendering、NeRF/3DGS。发 **Jittor Dynamic Human Rendering(JDHR)** 库与 3DGS CVMJ Spotlight 综述(覆盖数字人)。CAD/CG 2023 报告"基于神经表示的虚拟人技术"。
- **相关度 ★★★★☆**——渲染基建(Jittor)+ GS-for-humans 强;LLM 驱动认知弱。
- **产业开放度 ★★★★★**——运营长期清华-腾讯联合实验室。

### 2.4 刘永进 Yong-Jin Liu ✅
- 长聘教授,国家杰青(2017)。人机交互与媒体集成研究所所长(2018–2025)。
- **焦点**:计算几何、计算机图形、CV、多模态媒体。偏理论/几何,非 neural-rendering 中心。
- **相关度 ★★☆☆☆**——几何基础;非直接 avatar 渲染。

### 2.5 刘华平 Liu Huaping — 感知/SLAM(国家重点实验室)
- 智能技术与系统国家重点实验室;多模态主动环境感知、深度 RL、多机器人主动 EKF-SLAM。[中国日报 2019](http://tech.chinadaily.com.cn/a/201907/12/WS5d2833c5a310a6dd41e85a5b.html)
- 注:清华 SLAM **分散**,不在某个品牌 PI 下;师生发视觉 SLAM + 学习式 SLAM。*全国SLAM技术论坛*曾在清华联办(2019,CSIG 三维视觉专委会)。[论坛报道](https://zhuanlan.zhihu.com/p/76048002)。专门 SLAM PI 在 SJTU/HIT/ShanghaiTech 比清华更集中。

### 2.6 媒体所(人机交互与媒体集成研究所)— 图形/数字人
- 承载**华智冰**渲染侧与图形/VR 教研。**穆太江 Mu Taijiang**(助理研究员)在 CCF ADL155《可视媒体生成基础与前沿》教数字人生成(图/视频/3D 模型生成)。[CCF ADL155](https://ccf.org.cn/ADL155)；[研究所页](https://www.cs.tsinghua.edu.cn/jgsz/yjssys/jsjkxyksxrjjhymtjcyjs.htm)
- 一位 CS 长聘副教授 / 长江学者(姓名未定 **[unconfirmed]**——可能在胡事民组或徐枫谱系)做图形 + 3DGS。[CCIG 2025](https://ccig.csig.org.cn/2025/6861/list.html)

### 2.7 孙富春 Sun Fuchun 组 — 智能机器人研究中心 ✅
- ⚠️ **纠错:孙富春在计算机系(CS),不是自动化系**。清华 CS 博士(1997);本硕海军航空工程学院(自动化)。[CS 教师页](https://www.cs.tsinghua.edu.cn/info/1121/3555.htm)
- 头衔:IEEE / CAAI / CAA Fellow;国家杰青;智能技术与系统国家重点实验室副主任;清华大学人工智能研究院**智能机器人研究中心主任**;新**具身智能与机器人研究院(EIR)副院长**(见 §4)。[世界机器人大会简介](https://www.worldrobotconference.com/47/725.html)
- 当前工作:"知行体"具身智能体框架;触感灵巧手、电子皮肤、人形机器人、空间遥操作、智能假肢、3C 精密装配。主张人形机器人是具身智能"最重要平台",且**触觉纠偏比视觉纠偏更重要**。[人民日报 2025](https://paper.people.com.cn/mszk/pc/content/202504/14/content_30069817.html);[上海观察者](https://www.shobserver.com/wx/detail.do?id=986633)
- 邮箱 **fcsun@tsinghua.edu.cn**。
- 注:其官方 CS 页论文停在 ~2010 且写 "IEEE Senior Member"(陈旧);较新公开来源确认 IEEE Fellow 与活跃领导职务。

## 3. 自动化系

### 3.1 刘烨斌 Liu Yebin ★★★★★（核心数字人 PI）✅
- 长聘教授,国家杰青 / 国自然卓越项目。北邮本科,清华自动化博士(2009)。~1.73 万 Google Scholar 引。[自动化系页](https://www.au.tsinghua.edu.cn/info/1080/3160.htm)；[教研页(备用)](https://www.au.tsinghua.edu.cn/info/1166/3336.htm)；[AMiner](https://www.aminer.cn/profile/yebin-liu/542c347dafbfae2b4e1f01f5)；[Scholar](https://scholar.google.com/citations?user=ogXIdlYAAAAJ)
- **焦点**:动态三维重建、3D/4D 内容**生成**、数字人重建/生成/交互、**全息通信**、neural rendering、**Gaussian Splatting**。TPAMI/SIGGRAPH/CVPR/ICCV 100+ 论。
- **学生实习过 Meta Reality Labs 与 USC**——直接对点 Vision Pro 管线。
- **签名论文/数据集**(与学生郑泽荣等):
  - **Gaussian Head Avatar: Ultra High-fidelity Head Avatar via Dynamic Gaussians** — [arXiv:2312.03029](https://arxiv.org/abs/2312.03029)
  - **Animatable Gaussians...** — CVPR 2024
  - **GPS-Gaussian** / **HumanNorm**(CVPR 2024)——角色新视角合成与法向条件人体生成
  - 合著综述 *AI for VR: 虚拟现实中的人工智能*(SCIS, 2026),覆盖 3DGS + neural rendering + GANs for VR。[PDF](http://scis.scichina.com/cn/2026/SSI-2025-0508.pdf)
  - **DNA-Rendering**——大规模高保真神经演员库(ICCV 2023,arXiv **2307.10173**)。https://dna-rendering.github.io/
  - GUAVA(arXiv 2505.03351)——见 §6 SIGS 王好谦组(跨组合作)
- **产业开放度——关键**:其工作据报道被**华为、字节跳动、OPPO、海信、苹果(Apple)**采用/应用([中山讲座](https://cse.sysu.edu.cn/event/72)；[百度百科](https://baike.baidu.com/item/刘烨斌/8687285))。**这是清华数字人/AR 渲染方向与 Apple 最强的一处关联。** `[unconfirmed:与 Apple 的关系性质——可能是 IP 授权、咨询或产品使用,而非正式联合实验室]`
- **本项目相关度:极高。** 与写实虚拟角色直接重叠:Gaussian 头/身 Avatar 建模、动态外观、单视频角色采集、AR/VR 全息/体素串流。

### 3.2 戴琼海 Dai Qionghai — 母实验室,院士 ✅
- 中国工程院院士,自动化系副主任(前)。成像与智能技术实验室首席科学家。
- **焦点**:计算摄影、立体视频、介观活体荧光显微、光计算。基础成像;刘烨斌数字人工作在其伞下。

### 3.3 LEAP Lab(LEarning And Perception)— 黄高 Gao Huang ✅
- 长聘副教授,自动化系工业智能与系统研究所。Cornell 博后;清华自动化博士 2015。**DenseNet 一作**(6.2 万+ 引用);Google Scholar ~10.5 万。**中国自动化学会具身智能专委会副主任委员**。[自动化系页](https://www.au.tsinghua.edu.cn/info/1075/3183.htm)；[个人站](http://www.gaohuang.net)
- **工作**:深度学习基础模型、多模态学习、RL、**具身基础模型与世界模型**;**灵巧/鲁棒操控**(如 *Towards Affordance-Aware Robotic Dexterous Grasping...*,AAAI;综述 *The Developments and Challenges Toward Dexterous and Embodied Robotic Manipulation*,IEEE RAM);自主颈动脉超声机器人(中国首台,与解放军总医院 & BAAI)。获 **NeurIPS 2025 Best Paper runner-up**、CVPR'17 Best Paper、MIT TR35 亚太、达摩院青橙奖。
- **产业开放度**:**明确"长期开展与华为、字节、阿里、腾讯等企业的校企科研合作"**;主持国家优青/中电科联合重点/国家重点研发青年项目。强实习/合作信号。
- **本项目相关度**:中。具身认知 + 操控,非渲染/数字人。适合角色需要**认知/动作骨干**。

## 4. IIIS 交叉信息研究院(姚期智)— 具身视觉

IIIS 院长为图灵奖得主 **姚期智 Andrew Yao**。

### 4.1 VAR Lab(视觉与机器人实验室)— 高阳 Yang Gao ✅
- 助理教授,IIIS。清华 CS 本科 → **UC Berkeley 博士(导师 Trevor Darrell)**。[IIIS VAR 页](https://iiis.tsinghua.edu.cn/kxyj/ktzjs/sjyjqrsys_VAR_.htm)；[IIIS 简介](https://iiis.tsinghua.edu.cn/rydw/qzjs/gaoyang.htm)
- **工作**:通用具身智能体——**人形控制、灵巧操控、VLA 模型、触觉**。代表作框架 **ViLa** 与 **CoPa**(通用操控 LLM)。[知乎访谈](https://zhuanlan.zhihu.com/p/691623973)
- **产业**:**千寻智能(Qianxun Intelligence)**联合创始人——直接创业载体。[Bagevent 简介](https://www.bagevent.com/event/9144186)
- ⚠️ 某聚合帖出现 "EVAR Lab" 名 `[unconfirmed]`——IIIS 官方名为 VAR。

### 4.2 其他 IIIS 组
- **李 Yi(Li Yi)**——三维视觉计算与机器智能实验室;3D 视觉 + 人形机器人学习。[IIIS 组](https://iiis.tsinghua.edu.cn/kxyj/ktzjs.htm)
- **赵行 Zhao Xing**——MARS Lab,多模态 ML。

## 5. 新旗舰:具身智能与机器人研究院(EIR)— 跨院系

清华 **2025-11-30 揭牌**(02 记 2025-12-01 官方宣告)校级研究院,整合跨院系具身智能/机器人。站:**https://eir.tsinghua.edu.cn/**

- **院长**:**张涛 Zhang Tao**(自动化系系主任;信息科学技术学院副院长)。研究:机器人、智能控制、导航、飞控。[新浪报道](https://finance.sina.com.cn/wstock/hyyj/2025-12-01/doc-infzikpx3324972.shtml)
- **副院长**:**孙富春**(计算机系,见 §2.7)与**何潇 He Xiao**(自动化系,控制与决策研究所所长)。[清华公告](https://www.tsinghua.edu.cn/info/1182/122950.htm)；[知乎总结](https://zhuanlan.zhihu.com/p/1979837014863008018)
- **结构**:挂靠科研院;自动化系、机械系、电子系、计算机系共建。**五中心**:模型与交互、感知与控制、软硬件与本体、数据与算力、战略与标准。任务覆盖多模态感知与理解、鲁棒控制/决策、机器人"小脑/大脑"。战略="强健本体 + 智慧大脑"。
- **意义**:清华校级对具身AI 的承诺——最强制度信号。**这是清华任何具身智能合作的机构入口。**

## 6. 清华深圳国际研究生院(SIGS)— GUAVA ★★★★★

- 论文:**GUAVA: Generalizable Upper Body 3D Gaussian Avatar**(ICCV 2025,arXiv **2505.03351**)。
- 作者:Dongbin Zhang、Yunfei Liu、Lijian Lin、Ye Zhu、Yang Li、Minghan Qin、Yu Li、**王好谦 Haoqian Wang**(资深作者在 SIGS)。
- **做什么**:单图→可动上身 3DGS avatar **~0.1 秒**,实时动画/渲染(>50 FPS @512×512)。引 Expressive Human Model(EHM)修 SMPLX 表情弱点。
- **相关度 ★★★★★**——亚秒单图 avatar 重建 + 实时驱动,可以说是清华最 Vision-Pro-ready 单篇论文。
- 源:[arXiv 2505.03351](https://arxiv.org/abs/2505.03351)；[腾讯新闻](https://news.qq.com/rain/a/20250910A014MF00)

## 7. Apple Vision Pro / 空间计算 / AR @ 清华——弱（缺口分析）

- **未找到专门的 Apple × 清华联合实验室**(Vision Pro/空间计算)。唯一可定位的 Apple→清华 正式动作是 **2024-10 环保/气候教育捐赠**(非技术)。[CLS 报道](https://www.cls.cn/detail/2168399)
- **最强 Apple 技术关联**:刘烨斌数字人/GS 工作据报道**被 Apple 采用**(§3.1)——若目标是 Apple 相关 avatar 渲染,这是最佳切入点。
- **清华美术学院信息设计系**:2023-12,Vision Pro 进美院,在清华基础工业训练中心做"空间计算 + 设计创新"学生 workshop,由外部公司**创森智云**运营——*非*深学术研究。[Doublerise](https://doublerise.com/index.php/index/shows?catid=66&id=53)
- **清华大学未来实验室**(thfl.tsinghua.edu.cn):跨学科 compute/media/art/HCI 实验室——可能 XR 伙伴,但无公开 Vision Pro/visionOS 项目。
- AIR 或计算机系**无 visionOS app 研究、Apple Developer Program 实验室合作、空间计算系统研究证据**。空间计算/空间 AI 评论在清华存在(如陈巍技术文),但非命名实验室。[知乎](https://zhuanlan.zhihu.com/p/635083266)
- **结论:真实空白**——强 avatar 实验室无一有公开 Vision Pro/visionOS 项目。把合作框定为"首个清华学术 Vision Pro 空间 avatar 管线"很可能引起共鸣。

## 8. 清华对"Vision Pro 虚拟角色项目"相关度总结

| 需求 | 清华最佳匹配 | 相关度 | 原因 |
|---|---|---|---|
| **AR 可用 avatar 渲染 / neural / Gaussian** | **赵昊组(AIR)** / **刘烨斌(自动化系)** | ★★★★★ | 赵昊:SyncTalk++/Ultraman/NeAR/Light-X 全栈;刘烨斌:Gaussian Head/Body Avatar、DNA-Rendering、单视频采集、全息串流;Apple 据报为采用方 |
| **亚秒单图 avatar** | **GUAVA / 王好谦(SIGS)** | ★★★★★ | 单图→可动上身 3DGS avatar ~0.1s,实时 >50FPS |
| **LLM/agent 认知大脑** | **唐杰 / KEG → 智谱 GLM(CS)** | ★★★★☆ | GLM-5.2 开源;华智冰数字人血统 |
| **情感/语音认知半边** | **贾珈(CS)** | ★★★★☆ | 情感计算 + 数字人 + 华智冰共创者 |
| **具身操控/动作闭环** | **高阳 / VAR(IIIS)**——ViLa/CoPa VLA;或 **黄高 / LEAP(自动化系)** | 中–高 | VLA + 灵巧操控;高阳还运营千寻智能 |
| **Real2Sim2Real 仿真平台** | **AIR DISCOVER Lab(周谷越/赵昊)**——DISCOVERSE + AIRBOT | ★★★☆☆ | 成熟 RSR 栈;开放产业联盟模式 |
| **渲染基建** | **胡事民组(CS)**——Jittor/JDHR | ★★★★☆ | Jittor + GS-for-humans;清华-腾讯通道 |
| **触觉 / 人形平台** | **孙富春(CS / EIR VP)** | — | 知行体框架、触觉灵巧手、电子皮肤 |
| **统一前门(跨院系)** | **新 EIR 具身智能与机器人研究院(院长张涛)** | — | 跨自动化 + 计算机 + 机械 + 电子的伞型机构,2025-11 成立 |

**产业合作开放度高且明确**:LEAP(华为/字节/阿里/腾讯)、刘烨斌组(华为/字节/OPPO/海信/**Apple**)、AIR(地瓜机器人/毫末智行/奔驰/比亚迪/求之科技孵化)、KEG(智谱)、VAR(千寻智能)、赵昊(Geek+)。多个命名孵化创业公司印证活跃的横向课题/实习文化。

### 清华未确认项
- 王海峰**不是**清华 AI/机器人 PI(与百度 CTO 混淆)。
- 刘烨斌组与 Apple 的确切商业关系未公开说明(可能 IP/咨询/产品使用,非公开联合实验室)。
- 一位做 3DGS 的 CS 长江学者(姓名未定)与 "EVAR Lab" 标签仍未确认。
- **SUNDAE**(3DGS 压缩,杨润毅等,清华)见于报道;具体清华子实验室归属未确认。
- 多篇赵昊论文具体 arXiv ID(SyncTalk/Morpheus/Ultraman/NeAR 等)未逐个核 venue 之外——引用前经其[发表页](https://sites.google.com/view/fromandto)核对。

---

# 第二部分 · 北京大学（原 03）

> PKU 有中国最深厚的具身AI 阵容之一。三个承重人物:**王亦洲**(视觉认知/因果世界模型)、**王鹤**(VLA/Galbot CTO)、**查洪彬**(SLAM/AR)。朱松纯持多项 PKU 院长职务与 BIGAI。

## 1. 王亦洲 Wang Yizhou ✅
- 教授;北京大学计算机学院**视频与视觉技术研究所**;兼**前沿计算研究中心(CFCS)**——领衔**计算机视觉与数字艺术实验室(CVDA)**,2007 成立。邮箱 **yizhou.wang@pku.edu.cn**。页:https://cs.pku.edu.cn/info/1089/1774.htm · https://cfcs.pku.edu.cn/people/faculty/yizhouwang/
- 教育:清华 BS 1996 → UCLA CS 博士 2005(导师朱松纯)→ Xerox PARC → 2007-12 入职 PKU。2023 起任**跨媒体通用人工智能全国重点实验室**学术委员。
- **研究领域**:视觉/AI、统计建模、认知计算、**因果学习**、医学影像、人体动作/视频、主动目标追踪、多智能体 RL;2024–2026 明确转向**"主动因果世界模型"**用于具身认知(报告《从感知-行动表征到主动因果世界模型》)。
- **代表作(与具身/Vision Pro 最相关),带 arXiv ID**:
  - **UnrealZoo: Enriching Photo-realistic Virtual Worlds for Embodied AI** — ICCV 2025(Highlight)。**arXiv:2412.20977**。https://arxiv.org/abs/2412.20977 · https://unrealzoo.site/ —— 100+ 拟真 Unreal 世界用于具身智能体训练。*直接相关:高保真虚拟环境 + 可控角色。*
  - **DyWA: Dynamics-adaptive World Action Model...** — ICCV 2025。**arXiv:2503.16806**
  - **Embodied Representation Alignment with Mirror Neurons** — ICCV 2025。*对齐观察动作与执行动作表征——对 avatar 的"看与做"概念核心。*
  - *Simulating Human-like Daily Activities with Desire-driven Autonomy* — ICLR 2025
  - *InteractAnything: Zero-shot HOI Synthesis via LLM Feedback...* — CVPR 2025
  - *GeneMAN: Generalizable Single-Image 3D Human Reconstruction* — NeurIPS 2025
  - *OpenDance: Multimodal Controllable 3D Dance Generation* — CVPR 2026
  - *The Tong Test*(AGI 具身基准,Engineering 2024);*(C,U,V) AGI 数学框架*(Engineering 2026)
  - *Detect-SLAM*(WACV 2018);人体动作谱系:Human Motion Generation Survey(TPAMI 2024)、VMarker-Pro(TPAMI 2025)
- **学生/合作者**:朱文涛(人体动作/镜像神经元)、马晓玄(3D 人体姿态)、钟方威(现北师大,见 §5)、慈海、叶航(NeRF/人体)、刘明舟(因果)、孙鑫伟(因果/医学)、王楚然、彭玉佳、张津路(HOI/舞蹈)等。CFCS 组在招**博雅博士后**(计算机视觉、认知计算、**具身AI/AGI/机器人**)。https://hub.baai.cn/view/52507
- **本项目相关度**:认知侧极高——视觉认知、因果/世界模型、人体动作与 HOI 生成、拟真具身仿真(UnrealZoo)。CVDA 是 AR 角色"具身认知大脑"的天然 PKU 归属。

## 2. 朱松纯 Song-Chun Zhu ✅
- **讲席教授**,智能学院(2021-12 起院长);人工智能研究院院长(2020 起);**北京通用人工智能研究院(BIGAI)**创始院长(2020 起);清华基础科学讲席教授。邮箱 **s.c.zhu@pku.edu.cn**。页:https://sist.pku.edu.cn/info/1022/2211.htm · https://www.bigai.ai/song-chun-zhu/
- 教育:中科大 BS 1991 → Harvard MS/PhD 1996 → Brown 博后 → Stanford → Ohio State → UCLA stat+CS full prof 2006–2020;创 UCLA **VCLA**。
- 其 PKU 邮箱与院长职务活跃;学生(彭玉佳、范丽锋、刘腾宇)出现在 PKU/BIGAI 论文。合著 ICLR 2025 主旨 "Frontiers of AGI"。BIGAI 具身智能体**"通通"(TongTong)**是 PKU/BIGAI 联合产出。
- **研究**:AGI 基础(小数据/大任务范式,"为机器立心")、认知视觉常识、认知推理、自主机器人。与王亦洲大量合著(UCLA 师生)。
- **本项目相关度**:提供认知/AGI 框架与通通式自主智能体——若角色需自主目标驱动行为有用,渲染方面弱。

## 3. 王鹤 Wang He & EPIC Lab ✅（VLA 重镇）
- Tenure-track **助理教授 & 博导**,计算机学院/CFCS;博雅青年学者;国家级海外人才。Lab:**EPIC Lab(具身感知与交互实验室)**,2021-09 成立,"中国首个以'具身'命名的实验室"。产业:**银河通用机器人(GALBOT)创始人 & CTO**;中关村学院(ZGC Academy)研究导师。页:https://hughw19.github.io/ · https://cfcs.pku.edu.cn/research/research_labs/22b682f716b048d3b51559212191abf5.htm 。教育:清华 BS → Stanford 博士(导师 Leonidas Guibas)。荣誉:MIT TR35 China;财富中国 40-under-40(2025);2025 世界互联网大会领先科技奖;ICCV 2023 Best Paper Finalist。
- **工作**:具身基础模型 & VLA、灵巧/人形操控、sim-to-real、具身导航、3D 感知。**2026-02-09 习近平视察 Galbot G1 并见王鹤**;Galbot G1 登 2026 央视春晚。ICRA 2026 产业主旨:"Towards the AlphaGo and ChatGPT Moments of Embodied AI."
- **代表模型/论文**:
  - **GraspVLA**(CoRL 2025)——"首个在十亿级合成 VLA 数据上预训练的端到端具身抓取基础模型"。**arXiv:2505.03233**
  - **DexVLG**(ICCV 2025 Highlight)——**arXiv:2507.02747**
  - **Uni-NaVid**(RSS 2025)& **NaVid/NaVid-4D**(ICRA 2025)——视频 VLA 导航。arXiv 2412.06224
  - **SoFar**(NeurIPS 2025 Spotlight)——arXiv 2502.13143
  - **LDA-1B**(RSS 2026)——1.6B 跨本体潜在世界-动作基础模型。**arXiv:2602.12215**。与**王亦洲†**合著。
  - TrackVLA/++、StereoVLA、Any3D-VLA(arXiv 2602.00807)、NavGSim、UrbanVLA 等。
  - 基础:SAPIEN(CVPR 2020)、UniDexGrasp/++(CVPR/ICCV 2023 Best Paper Finalist)、GAPartNet、DexGraspNet。
- **产业开放度**:最大。**北大-银河通用具身智能联合实验室**位于中关村鼎好大厦,**明确招实习**(王鹤主页)。https://cfcs.pku.edu.cn/news/42cfcs242090.htm
- **本项目相关度**:VLA/动作侧极高——感知-动作模型、灵巧交互、导航基础模型。渲染直接性弱,但 NavGSim(GS 导航仿真)与 4D-Rotor GS 涉及实时渲染。

## 4. 查红彬 Zha Hongbin ✅（PKU SLAM 领头）
- 教授,智能学院;教育部长江学者特聘。**机器感知与智能教育部重点实验室**主任;**跨媒体通用人工智能全国重点实验室**成员。页:https://sai.pku.edu.cn/info/1362/2247.htm
- **研究**:三维视觉几何、三维重建与环境几何建模、3D 物体/人脸识别、**3D 人脸动画**、动态场景理解、**SLAM/视觉里程计(在线学习 SLAM、数据流 SLAM)**。
- **已确认**:其团队发"环境自适应鲁棒 SLAM",明确用于"智能机器人、自动驾驶和**增强现实(AR)**"。https://sai.pku.edu.cn/info/1088/2370.htm —— PKU 最 AR 相关的 SLAM 组。
- **本项目相关度**:直接相关——视觉 SLAM、相机重定位、3D 人脸动画是任何空间计算角色场景的核心。

## 5. 钟方威 Fangwei Zhong ✅（王亦洲校友,现北师大 + PKU 博后）
- **副教授,北京师范大学人工智能学院**;兼**北京大学博雅博士后**。邮箱 **fangweizhong@bnu.edu.cn**。页:https://fangweizhong.xyz/ · https://ai.bnu.edu.cn/xygk/szdw/fgj/067725da68f443e09e675307b4e7132a.htm
- **研究**:具身AI、多智能体学习、机器人学习;**UnrealZoo** 与长期 **UnrealCV** 仿真生态创建者。王亦洲博士生,PKU 桥梁。UnrealZoo(arXiv 2412.20977)与 Detect-SLAM 一作。

## 6. 刘宏 Liu Hong ✅（人形/机器人老兵）
- 教授,国家级领军人才;**北京大学人工智能研究院具身智能与机器人中心**主任;科技部国家重点研发计划"智能机器人"总体专家组。驻**北京大学深圳研究生院(PKUSZ)**——robotics.pkusz.edu.cn/team/。长于计算机视觉与机器人;人形/双足行走。
- **本项目相关度**:渲染方面低;若虚拟角色需映射到物理人形平台则相关。

## 7. ACIR Lab — 情感与认知智能机器人实验室 ✅
- 全称 Lab for Affective and Cognitive Intelligent Robotics;2017 成立,在计算机学院。主任**王韬(Wang Tao)**研究员。页:https://acir.pku.edu.cn/
- **方向**:细粒度表情识别、注意力焦点分析、生理信号监测、情感-认知大模型、情感机器人原型(陪伴机器人"**爱瑟尔**",WRC 2024 展出)。
- **联合实验室**:**北大-国地共建具身智能机器人创新中心 情感智能应用联合实验室**(2024 首届学术委)。
- **本项目相关度**:对角色的**情感/表达层**高度相关——情绪识别、注意力建模、驱动可信 avatar 行为的情感-认知模型。

## 8. PKU 产业合作 / 联合实验室 ✅

| 联合实验室 | PKU 侧 | 合作方 | 时间 | 方向 / 链接 |
|---|---|---|---|---|
| 北大-银河通用具身智能联合实验室 | CFCS/王鹤 | Galbot | 2024-05 | 具身多模态"小脑"LLM;**招实习** https://cfcs.pku.edu.cn/news/42cfcs242090.htm |
| 北大-智平方具身智能联合实验室 | 计算机学院;**施柏鑫**长聘副教授共导 | 智平方(AgiBot,VLA 公司) | 2025-04-17 深圳 | 端到端具身 LLM、空间智能 https://cs.pku.edu.cn/info/1263/3701.htm |
| 北大-智元机器人联合实验室 | CFCS | 智元机器人(AgiBot) | 较早 | 机器人 https://hub.baai.ac.cn/view/34013 |
| 北大-国地共建...情感智能应用联合实验室 | ACIR(王韬) | 国家具身AI机器人创新中心 | 2024 | 情感机器人 https://acir.pku.edu.cn/lhsys1/lhsysjs/index.htm |

PKU 还有计算机学院/CFCS 暑期夏令营/实习项目,王亦洲组在招博雅博士后。产业合作结构性大开。

## 9. PKU 数字人 / neural avatar / talking-head——最薄弱

**没有可核验的专门 PKU "talking-head/neural avatar" 实验室**。最接近:
- 王亦洲/CVDA 组——人体动作生成(Human Motion Generation Survey TPAMI 2024)、**GeneMAN** 单图 3D 人体重建(NeurIPS 2025)、**OpenDance**(CVPR 2026)、**InteractAnything**、free-form 着装人体建模。属生成/重建,非实时 talking-head。
- 查红彬做 3D 人脸动画(较旧)。
- 施柏鑫——计算摄影/视觉;无可核验 talking-head 线。
- 北大数字人文中心(pkudh.org)是人文项目(经典文本 AI agent),非图形/avatar——勿混淆。
- **结论:neural-avatar/talking-head 实时渲染,PKU 弱于清华**(DFRF/GeneFace 谱系在清华/鉴智)。标记为 PKU 专门强项的空白。

## 10. Apple Vision Pro / 空间计算 @ PKU——很有限

- 临床:北大人民医院王俊院士团队完成 Vision Pro 首例胸腔镜手术应用。https://news.pku.edu.cn/xwzh/6642b86537fb429fa2828b2306248a3b.htm
- 教学:尹晓腾《基于Apple空间计算技术的创新场景与开发技术》课程(经 Doublerise)。https://doublerise.com/index/shows?catid=66&id=36 —— *非研究实验室。*
- **无可核验 PKU 研究实验室做 Vision Pro 原生虚拟角色。空白与潜在合作角度。**

## 11. PKU 消歧 / 修正
- **邬霞不在 PKU**。她在**北京理工大学计算机学院**(脑机接口与类脑智能研究中心主任),前北师大人工智能学院。做脑信号AI/类脑视觉。
- 网上有**两个"Yizhou Wang"**:PKU 教授(王亦洲,@pku.edu.cn)vs NVIDIA/UW 深度学习工程师。勿混。
- **EPIC 的 "He Wang"** 与 **CVDA 的 "Yizhou Wang"** 是不同人;合著(DyWA、LDA-1B、ScissorBot)且为 PKU 具身AI 双柱。

## 12. PKU 对 Vision Pro 项目相关度总结

| 需求 | PKU 最佳匹配 |
|---|---|
| 具身认知/因果世界模型/视觉认知大脑 | **王亦洲(CVDA,CFCS)**——智力契合最强 |
| VLA/感知-动作/灵巧交互 + 产业管线 | **王鹤(EPIC Lab)+ Galbot 联合实验室**——最接近产品化,收实习 |
| 视觉 SLAM/相机重定位/AR 3D 场景 | **查红彬(智能学院)**——明确 AR |
| 角色情感表达/情绪模型 | **ACIR Lab(王韬)** |
| AGI 框架/自主目标驱动 agent | **朱松纯(智能学院/人工智能研究院/BIGAI)** |
| 高保真虚拟世界/仿真 | **UnrealZoo(王亦洲组/钟方威)** |
| Talking-head/neural-avatar 实时渲染 | **PKU 空白**——需自建或合作(清华更强) |
| 原生 Vision Pro/visionOS 开发 | **PKU 空白**——空白机会 |

**最可执行入口**:北大-银河通用联合实验室(开放实习)与王亦洲博雅博士后(明确列具身AI/AGI/机器人)。

---

# 第三部分 · 上交 / 中科院 / 中科大（原 07）

## 1) 上海交通大学 SJTU

### 1a. MVIG — 机器视觉与智能组(卢策吾 Lu Cewu)✅ 高度相关
- SJTU MVIG(mvig.org),在新**人工智能学院**下。卢策吾兼**上海创智学院**全职导师,领衔机器人创业**穹彻智能 Noematrix**。
- **PI 卢策吾**:教授/博导,国家级海外高层次青年人才,MIT TR35 China(2018),科学探索奖。Stanford AI Lab 博后(导师 Fei-Fei Li、Leonidas Guibas)。
- **工作**:通用机器人/具身智能,明确围绕"C-3PO/R2 式智能机器人(真实+仿真)"。含机器人技能学习、手-物交互、sim-to-real,**以及"数字人和机器人的具身智能"**——直接命中项目。
- **代表作**:**RH20T**(大规模真实世界机器人操作数据集,11 万+ 接触富轨迹,多模态,人示教+机器对)。arXiv:**2307.00595**。https://rh20t.github.io/ ;RH20T-P arXiv:**2403.19622**;知名 demo"刮胡子机器人"。
- **产业开放度**:高。卢创办穹彻智能(具身"大脑"公司);实验室开放实习/招聘。
- **相关度**:具身认知+感知-动作闭环强;实验室公开把数字人与仿真 avatar 视为同一栈。
- 源:https://www.mvig.org/ ;["数字人和机器人的具身智能"报告](https://sai.sjtu.edu.cn/cn/show/222);[Noematrix](http://www.news.cn/enterprise/20250521/8f877af8b7a7469c9cd1f0c3f98da2aa/c.html)

### 1b. 严骏驰 Yan Junchi(SJTU 人工智能学院)✅
- 教授,国家优青,**IAPR Fellow(2024,当年唯一 40 岁以下 fellow)**,IET Fellow。前 IBM Research 首席科学家 + Amazon AI Lab 顾问(~10 年工业界)。
- **工作**:ML + 跨应用;**统一多模态表征学习**、图/结构学习、**量子 ML**、视觉识别,近期**端到端自动驾驶 + 世界模型**。理论深度强(ICML/NeurIPS/ICLR area chair)。
- **相关度**:世界模型 + 多模态统一表征正是"维持用户/环境内部模型"的 AI 角色基底。渲染/数字人导向弱于 MVIG。
- 源:https://soai.sjtu.edu.cn/cn/facultydetails/zzjs/yanjunchi ;[E2E AD+世界模型报告](https://ccf.org.cn/ncca2024/speaker_d_3170)

### 1c. 王延峰 Wang Yanfeng(SJTU)✅ — 偏领导/应用 CV
- 教授,**SJTU 人工智能学院执行院长**,SJTU 工业创新研究院理事长,上海市优秀学术带头人,国家级高层次人才。领衔 **MediaBrain** 团队。
- **工作**:ML、CV、**医学影像**、视频修复/增强、GNN/图信号、无人系统、语音/音频。视觉-媒体-医疗交叉与商业化。
- **相关度**:与数字人旁系;团队做感知/视频但非角色/具身AI 组。价值在作 **dean/门户**通往 SJTU AI 学院合作。
- **消歧**:另有云从科技的王延峰(b.1977)——勿混。

### 1d. 其他 SJTU 人形/VR/AR 实体 ⚠️（"SJTU-HVR" 未坐实）
找不到名为 **SJTU-HVR** 的实验室。最接近的已核验实体:
- **元知机器人研究院(MRI)** https://mri.sjtu.edu.cn/(跨学科机器人;办过"数字人和机器人的具身智能"报告)
- **ScaleLab(空间认知与机器人自主智能实验室)** 在虚拟世界做机器人"数字孪生"用于 sim-to-real https://scalelab-sjtu.github.io/
- **RL2 Lab** 情感具身交互/共情机器人 https://gaoyue.sjtu.edu.cn/
- **Digital ART Lab(软件学院)** VR/AR/MR + 图形 + 3D 视觉
- **俞凯 Yu Kai**(智能语音)——顶级语音/对话,已商业化——与角色的语音相关

## 2) 中科院自动化所 CASIA

### 2a. 紫东太初 Zidong Taichu + 王金桥 Wang Jinqiao ✅ 顶级相关
- **模型**:**紫东太初**——中国旗舰**全模态大模型**:1.0(2021,全球首个三模态 图/文/音);2.0 加视频/3D/传感器;3.0(2024-11)"像人一样思考";4.0 加推理。全国产 **华为昇腾 + MindSpore**。获 WAIC **SAIL 奖**;首批过国家网信办备案。
- **PI 王金桥**:CASIA 副总工程师,研究员,**紫东太初大模型研究中心主任**,武汉人工智能研究院院长,中科紫东太初公司董事长。领衔"2035 团队"。
- **相关度**:原生跨模态(视觉+语音+文本+3D)模型是端侧角色的显然认知骨干。开源权重/内部是你栈中最实用的中国基座模型。
- **产业开放度**:很高——~70 成员多模态人工智能产业联合体,与华工科技/九州通/新华社联合实验室。商业实体中科紫东太初。
- **开源/论文**:GitHub(MindSpore)https://github.com/mindspore-ai/zidongtaichu —— 指引引用 arXiv:**2107.00249**(基础多模态论文;arxiv.org 本会话被屏蔽,标题 **[unconfirmed]**);**Griffon**(grounding/检测)arXiv:**2503.18013**;门户 https://taichu.ia.ac.cn/
- 源:[王金桥](https://ia.cas.cn/rcdw/yjy/202404/t20240422_7129874.html);[3.0 发布](https://news.hubeidaily.net/pc/c_3376899.html)

### 2b. 王亮 Wang Liang ✅（仍主要 CASIA）
- CASIA 研究员(2010 起),**多模态人工智能系统全国重点实验室副主任**,IEEE Fellow(2019)、IAPR Fellow(2014)、CCF/CSIG/CAAI/CIE Fellow,**第十四届全国政协委员**。在上海科大 SIST 有交叉列,但主任何在 CASIA(UCAS 确认)。
- **工作**:视觉大数据——**视觉追踪、行人重识别 ReID、动作识别、SLAM/VR**;近期视觉基础模型(如 **Obj2Seq**,NeurIPS 2022)。
- **相关度**:感知骨干(追踪用户、识别动作/身份)用于 AR 角色——但他是视觉追踪 lead,非数字人生成 lead。

### 2c. 黄凯奇 Huang Kaiqi ✅（决策/RL,非数字人）
- CASIA 研究员(二级),**复杂系统认知与决策重点实验室副主任**。
- **工作**:**深度强化学习、多智能体学习、游戏 AI、分布式深度 RL**、LLM-agent 兵棋/决策。
- **相关度**:角色需**策略/动作选择或多智能体交互**时有用;非感知/渲染组。

> **其他 CASIA 具身 lead(搜索中发现,切题)**:**崔少伟 & 马云开**(智能机器人系统,触觉/灵巧操作,视触觉大模型 GelStereo);**张兆翔 Zhang Zhaoxiang**(Robot Vision Group,3D 场景重建/SLAM http://vision.ia.ac.cn/);**边桂彬 Bian Guibin**(与 SIA 共同主持具身智能机器人论坛)。

## 3) 中科院计算所 ICT-CAS — VIPL 组 ✅

VIPL(**视觉信息处理与学习**)1997 由**高文院士**创立。20+ 研究员,80+ 研究生。**5 家中最直接数字人相关**。

### 3a. 山世光 Shan Shiguang ✅ — "数字人脸"最强契合
- ICT 研究员/博导,所务委员;**智能信息处理重点实验室主任**;**智能算法安全全国重点实验室副主任**。**中科视拓 SeetaTech** 创始人/董事长/CTO(人脸技术公司,融过 pre-A)。
- **工作**:1999 起领衔**人脸组**。400+ 论文。**视觉情感计算与心理健康评估**:表情识别、**微表情分析**、面部动作单元检测、**注视估计/追踪**、唇读、生理信号测量 + AI 安全 + AI4Science。
- **相关度**:最契合**富有表情的数字人脸**——注视、微表情、lip-sync、情绪。AR 角色面部可信度与情绪环路直接映射该组产出。
- **产业开放度**:很高——已运营公司(中科视拓)商业化人脸技术。
- 源:https://vipl.ict.ac.cn/people/sgshan/ ;[ICT](https://www.ict.ac.cn/sourcedb/cn/jssrck/200909/t20090917_2496706.html)

### 3b. 陈熙霖 Chen Xilin ✅ — 资深领导/多模态 HCI
- ICT 研究员,**所长兼党委书记**,ACM/IEEE/IAPR/CCF Fellow,国家杰青。CV、模式识别、多媒体、**多模式人机接口**。
- **相关度**:定所级方向;VIPL 覆盖视觉/多媒体/HCI/情感计算。深度 ICT 合作的正确"前门"。渲染上手不如山世光。
- 源:https://www.ict.ac.cn/sourcedb/cn/jssrck/200909/t20090917_2496595.html ;[VIPL](https://vipl.ict.ac.cn/people/)

> VIPL 教研名单(已核验)还有:蒋树强(多模态/食物/识别)、王瑞平、常虹、王树徽、韩琥——按子领域选。https://vipl.ict.ac.cn/_people/

## 4) 中科院沈阳自动化所 SIA ✅ — 机器人底蕴,具身AI 上升

SIA 是 CAS 历史机器人所;运行**机器人与智能系统全国重点实验室**(rlab.sia.cas.cn)。强物理机器人(工业/手术/空间/海洋),日益具身智能。

### 4a. 韩志 Han Zhi ✅ — 具身智能 + 机器人视觉 lead
- 研究员/博导,**机器人学研究室副主任**,**机器智能课题组组长**。中组部"万人计划"青年拔尖,中科院青促会优秀会员,国家重点研发计划首席科学家。
- **工作**:**具身智能、类生命智能、机器人视觉、模式识别、ML**。组 2007-01 成立;覆盖感知/成像、图像视频分析、深度学习、自主机器人。
- **相关度**:具身智能体感知+动作;与 CASIA 共同主持 2026 中国科协"具身智能机器人与群智能体协作"论坛——即中国具身AI 社区的当前召集人。
- 源:http://sia.cas.cn/vision/ ;[UCAS](https://people.ucas.ac.cn/~hanzhi);[论坛](http://sia.cas.cn/xwzx/xshd/202607/t20260728_8256152.html)

### 4b. 于海斌 Yu Haibin(中国工程院院士)✅ — 战略 lead
- **中国工程院院士**,**工业人工智能研究所所长**。SIA 具身智能+智能机器人战略公开代言人。

### 4c. 其他已核验 SIA 具身 PI
- **刘浩 Liu Hao**——研究员,青A,**辽宁省微创手术机器人重点实验室主任**(手术机器人)。
- **刘连庆 Liu Lianqing**——研究员,论坛学术共主席(与于海斌)。
- 信号:SIA 一批**具身AI 导航+低秩张量论文被 ICLR 2026 接收**。
- ⚠️ **准确性注**:*Jinguo Liu* 常被称"具身AI与机器人研究所所长",但他现**在沈阳理工大学**,此前在 SIA——勿列为现任 SIA 教研。

**SIA 相关度警告**:物理机器人世界级,但对**虚拟/数字角色契合弱于** CASIA/ICT/MVIG。若你想让虚拟角色最终驱动物理人形或遥操作机器人,才找 SIA。

## 5) 中科大 USTC

### 5a. 张兰 Zhang Lan — ⚠️ 错配（已标）
中科大 CS 教授,但领域是**大数据、隐私保护、数据共享/交易**——非具身AI或数字人。大概率不是 Vision Pro 角色伙伴。

### 5b. 真正的 USTC 具身/多模态 PI ✅

**查正军 Zha Zhengjun — MEI-Lab(多模态具身智能实验室)** ★ USTC 最契合
- USTC **多模态具身智能实验室(MEI-Lab)主任**。教授/博导,信息科学技术学院。研究:**多模态智能、视觉感知与认知智能、具身智能**、多模态内容理解/生成、复杂媒体分析。
- Lab:https://ustc-milab.work/ ;profile https://iatyz.ustc.edu.cn/teacher/profile/name/查正军
- **相关度**:字面"多模态具身智能"——USTC 最接近认知角色栈(多模态感知+生成+具身)。

**张燕咏 Zhang Yanyong — 机器人 & 具身感知**
- **讲席教授,USTC 人工智能与数据科学学院执行院长**,兼 SIA 机器人学研究室。研究:**具身智能与机器人、无人系统智能感知、多模态融合感知 LLM、边缘-云信息物理系统、实时中间件/OS**。
- **相关度**:具身感知 + 实时/边缘部署——关心角色端侧延迟时有用。

**张世武 Zhang Shiwu — 人形机器人**
- **工程科学学院副院长,人形机器人研究院副院长**,教授/博导;国家高层次人才。仿生/软体机器人与人形机构。

**其他 USTC 信号**:**通用人工智能研究所**(2026-01 新成立);**具身智能系统与控制联合实验室**(与启智/Qizhi 机器人公司);视触觉感知工作(工程+ NUS)。

## 上交/中科院/中科大 对 Vision Pro 项目契合度排名

| 排名 | 团队 | PI | 原因 |
|---|---|---|---|
| 1 | ICT-CAS / VIPL 人脸组 | **山世光** | 注视、微表情、AU、lip-sync、情绪——正是面部/表情层;有商业化载体(中科视拓)|
| 2 | CASIA | **王金桥**(紫东太初)| 原生跨模态认知骨干(视觉+语音+文本+3D)、开源权重、强产业联合体 |
| 3 | SJTU / MVIG | **卢策吾** | 具身认知+感知-动作+明确"数字人&机器人具身AI";RH20T 数据;运营穹彻智能 |
| 4 | USTC / MEI-Lab | **查正军** | 以名"多模态具身智能"实验室;感知+生成+具身 |
| 5 | SJTU / 人工智能学院 | **严骏驰** | 多模态统一表征 + 世界模型(理论深度)|
| 6 | CASIA | **王亮** | 视觉追踪/ReID/动作——"感知用户"传感层 |
| 7 | SIA-CAS | **韩志/于海斌** | 仅当桥接物理人形/遥操作机器人时选 |

**降级/错配**:张兰(USTC 数据隐私——不切题);黄凯奇(CASIA RL/决策——窄契合);王延峰(SJTU——作 dean/门户有价值,非角色专家)。

**核验注意(接触前复查)**:
- arXiv:**2107.00249**(紫东太初基础论文)确切标题——本会话未能 fetch arxiv.org;MindSpore GitHub 指引引用它。`[title unconfirmed]`
- "SJTU-HVR" 作为实验室名——**未确认**;已列已核验 SJTU 人形/VR 实体替代。
- 未核验 AnyTeleop(arXiv:2307.04577)是否卢策吾的——基于搜索结果不安全归因 MVIG,已排除。

---

# 第四部分 · 上海 AI Lab / 智源 BAAI（原 09）

## 第一部分 — 上海人工智能实验室 SHLAB

### 机构与领导（已核验）
- 全称 Shanghai Artificial Intelligence Laboratory(SHLAB),2020-07 于 WAIC 揭牌。
- **主任 & 首席科学家:周伯文 Zhou Bowen**（已核验）。
- **创始人物(2023 故):汤晓鸥 Tang Xiao'ou**——CUHU 教授、**商汤创始人**(历史商汤关联,见下)。
- 其他资深:**姚期智 Andrew Yao**(图灵奖、CAS 院士,与周伯文共任 2026 浦江年会主席);**乔宇 Yu Qiao**(首席/leading scientist,兼 SIAT 多媒体实验室主任);**陈恺 Chen Kai**(大模型中心负责人,领衔 **Intern-S1** 科学多模态)。

### "书生 Intern" 家族

| 名 | 领域 | 状态 |
|---|---|---|
| **InternLM(书生·浦语)** | LLM | 多版本开源 |
| **InternVL** | 视觉-语言基座 | InternVL 3.5 最新开源 |
| **Intern-S1** | 科学多模态(1B–100B+) | "最佳开源多模态族",AI4S |
| **InternVLA-A1** | VLA:统一理解+生成+动作 | arXiv **2601.02456**(经 alphaxiv 聚合,建议直接核 arxiv)|
| **InternVLA-M1** | **空间引导 VLA**——空间 grounding+机器人控制 | arXiv **2510.13778** 已核验 |
| **InternVLA-N1** | 仿真相关 | **论文 ID unconfirmed**(仅 GitHub 描述)|
| **VeBrain(视觉具身大脑)** | "通用"具身脑:感知+空间推理+决策 | 2025-06 发布,多伙伴 |

> ⚠️ 你 prompt 里的 **"InternSpatial"** 非独立模型名——空间 grounding 工作以 **InternVLA-M1** 发表。"Landmark" 出现在 EchoMimic 的面部 landmark 条件,非机器人项目。

### OpenRobotLab / 浦器（具身AI 团队）
- **OpenRobotLab(浦器)**是 SHLAB **OpenXLab 浦源**开源生态(WAIC 2022 发布 9 子实验室)的机器人/具身支柱。GitHub org **`InternRobotics`**("Building inclusive infrastructure for Embodied AI, from Shanghai AI Lab")。
- **领衔 PI(带细微差别)**:
  - **卢策吾 Lu Cewu**——上交 CSE 教授;人民日报系源称"浦器团队负责人"。(与 SJTU MVIG 同人,见第三部分)
  - **庞江淼 Jiangmiao Pang**——"青年科学家/浦器 team leader"(CEAI 2024 官方议程 + 雷峰网)。站 https://oceanpang.github.io/ —"Research Scientist at Shanghai AI Laboratory. We go with Intern Robotics. Our mission is to develop Embodied AGI systems."
  - **曾嘉 Jia Zeng**——青年研究科学家;SJTU 博士 2023;领衔"虚实贯通"具身技术;做 VLA。https://zeng-jia.github.io/
  - **解读**:卢策吾 = 教研级 lead/SJTU 侧 PI;庞江淼 = 浦器现任 in-house 运营负责人;曾嘉 = VLA lead。

### 签名具身论文/项目
- **InternVLA-M1**——*"A Spatially Guided VLA Framework for Generalist Robot Policy"*,arXiv **2510.13778**(2025-10)。统一**空间 grounding + 机器人控制**;空间线索桥接 指令→动作。项目 https://internrobotics.github.io/internvla-m1.github.io/ ;GitHub https://github.com/InternRobotics/InternVLA-M1
- **InternVLA-A1**——统一理解/生成/动作,声称 arXiv 2601.02456,12 项真机+仿真基准超 π0。(ID 仅经聚合见,直接核 arxiv。)
- **VeBrain**——首个集成视觉感知+空间推理+具身决策的"通用"机器人脑(2025-06)。
- **书生具身全栈引擎**——与上海国人形机器人创新中心发布;书生具身操作大模型+数据引擎,sim-real 融合。

### 数字人/虚拟 avatar（对 Vision Pro 角色相关）
- **EchoMimic**——*"Lifelike Audio-Driven Portrait Animations through Editable Landmark Conditioning"*,arXiv **2407.08136**(2024-07),**AAAI 2025**,230+ 引。
  - ⚠️ **出处注**:canonical repo 是 **`antgroup/echomimic`**——主所有者是**蚂蚁集团**,SHLAB 相关者合著。按**蚂蚁集团主导合作**处理,非纯 SHLAB 产品。
  - 音频/面部 landmark/二者→肖像视频;V2 扩到半身对话动画。https://github.com/antgroup/echomimic
- SHLAB 数字人相关能力:多模态生成(InternVL/Intern-S1)、3D 视觉、EchoMimic 音驱动画。**未在已核验源中识别 SHLAB 主导的专门"虚拟人产品"** `[unconfirmed]`。

### 商汤/产业关系
- 关联是**历史性、人脉型,非公司**:SHLAB 已故创始人汤晓鸥亦是商汤创始人。SHLAB 是政府背景新型研发机构,非商汤子公司。与商汤等联合开源 InternLM 屡见中文媒体。勿把"SHLAB=商汤"简单化。

### 合作通道
- **OpenXLab 浦源**开源生态;书生·浦源大模型挑战赛、浦科 AI-for-Science 平台。
- **浦江 AI 学术年会**(SHLAB 主办,国际学术交流;2026 由姚期智+周伯文主持)——天然社交场。
- **浦江书院**(产业+学术双导师联培博士)、**联培博士**(2027 批,含上海大学等)。
- SHLAB 实验室级**未发现公开可单独寻址的"开放课题"**(不像 BAAI/同济/PKU 的国家重点实验室模式)。对外/个人开发者现实通道:(a) OpenXLab 开源贡献,(b) 会议/workshop 社交(浦江、Intern Robotics Workshop),(c) 经 SJTU/HKUST/CUHK 学术合作者合著。

### 对 Vision Pro 项目的意义
- **InternVL/Intern-S1**——感知骨干(对房间/用户/物体的视觉-语言理解)。
- **VeBrain + InternVLA-M1 空间模块**——直接相关于虚拟角色需对用户位置、物体位置、指向/反应的**空间 grounding**。
- **EchoMimic**——角色音驱面部/上身动画。
- **caveat**:以上都面向*物理*机器人。均非 Vision Pro/visionOS/Swift SDK——复用权重或思路,非即插库。代码多 Apache-2.0/MIT,逐 repo 核。

## 第二部分 — 北京智源人工智能研究院 BAAI

### 机构与领导（已核验）
- 2018-11 经北京智源行动计划成立;非营利新型研发机构,北京市政府支持。
- **理事长:黄铁军 Huang Tiejun**(北大教授)。
- **院长:王仲远 Wang Zhongyuan**(新华网访谈确认)。
- **学术副院长:唐杰 Tang Jie**(清华教授,**悟道 WuDao 奠基 lead**——与清华 §2.1 同人)。
- **具身智能大模型负责人:王鹏伟 Wang Pengwei**(已核验;领衔 RoboBrain、RoboOS、ORCA 世界模型;人大企业博导;前阿里达摩院、快手)。

### 四大研究支柱（官网）
1. 大语言模型 LLM——BGE、Tele-FLM。
2. 多模态大模型 MLM——Emu、**OmniGen**、EVA、**Painter**、**SegGPT**、See3D、Bunny、VideoXL。
3. 生命大模型 LSLM——OpenComplex、实时数字孪生心脏、**智源线虫(全生物体模型)**。
4. **具身大模型 ELM**——具身模型系统、应用方案、具身数据。**最相关。**

### 悟道 WuDao（旗舰历史）
- 悟道 1.0(2021-03,中国首个超大规模智能模型系统);悟道 2.0(2021-06,当时世界最大稠密模型 1.75T);悟道 3.0 后开源进 FlagAI(现入 Linux Foundation);WuDaoCorpora。2025 起**从"悟道(语言)"转向"悟界(世界)"**(第7届智源大会,2025-06,四方向:神经科学、具身AI脑、生命科学、统一模态)。

### 具身AI:RoboBrain 2.0、RoboOS 2.0、ORCA（旗舰）
- **RoboBrain 2.0(具身大脑)**——开源具身"大脑"LLM;模块化编解码,统一感知+推理+规划。多项空间推理/任务规划 SOTA(任务规划较 1.0 +74%)。**32B 版**。
- **RoboOS 2.0**——"世界首个具身AI SaaS 开源框架";集成 **MCP 协议**+serverless,跨本体部署;单机+云版。
- **2025-07-14 全开源**(权重+训练代码+评测)。
- RoboBrain-Dex(灵巧操控预训练)、RoboBrain-Dopamine、RoboBrain-SpatialTrace(2.0 Pro)。
- **ORCA**——BAAI"世界基座模型",王鹏伟具身模型研究中心。https://hub.baai.ac.cn/view/55796
- **RoboSkill**——具身能力"技能商店"。

### FlagOpen(飞智)——开源系统 + Linux Foundation
- **FlagOpen** = BAAI"大模型界的 Linux"开源技术体系:算法/模型/数据/工具/评测。https://flagopen.baai.ac.cn/
- 组件:**FlagOS**(跨芯片统一 AI 软件栈,支持 18 厂商 32+ 芯片)、FlagEval、FlagData、**FlagAI(模型框架,已加入 Linux Foundation)**。
- 智源学者 3.0(BAAI Scholars 3.0)2025 年 50 名学者。

### 智源大会（旗舰社交场）
- 年度,自 2019。第7届(2025-06-5~7)发布悟界。**第8届:2026-06-12~13**,200+ 顶级科学家 + 40+ AI 公司 CEO;track 含 Agent、World Models、**具身智能**、AI 自进化、深度推理、多模态。**具身开放日**与 40+ 生态伙伴共办。智源社区 ~19 万从业者。

### 合作通道（对外/外国开发者）
BAAI 在结构上比 SHLAB 更对外友好:
- **智源学者 3.0**——公开征集,年支持 50 名研究者(FlagOS、具身AI 全栈、多模态、跨学科)。
- **FlagOpen/FlagAI(Linux Foundation)**——个人(含非中国籍)最低摩擦路径:标准开源 PR,无需中国归属。
- BAAI 行动计划原 mandate 明含"共建联合实验室"+"开放服务平台"。
- **《国际人工智能开源合作倡议》**——BAAI 与**工信部**、OpenAtom、CSDN 共发,明确国际范围。
- **关联国家重点实验室(年度开放课题,~20–30 万/项)**——最像西方 PI grant 的"开放课题"机制:
  - 跨媒体通用人工智能全国重点实验室(北大)——2025 征集明确列具身AI题,含**"可泛化的人与物体的全身交互动作生成"**——直接相关虚拟角色。
  - 自主智能无人系统全国重点实验室(同济)。
  - 多模态人工智能系统全国重点实验室(CAS 自动化所)。
  - ⚠️ **caveat**:这些开放课题通常要求申请人持博士/副高职称于研究机构,并与实验室成员合申。**无学术归属的个人外国开发者不能直接申**——需经中国学术合作者,或走 FlagOpen。

### 对 Vision Pro 项目的意义
- **RoboBrain 2.0 + RoboOS 2.0**——模块化 感知→推理→规划 管线,正是具身虚拟角色所需;**RoboOS 的 MCP 支持**是现代 agent 式架构的干净集成点。
- **Emu/OmniGen/SegGPT/Painter**——强视觉生成/分割栈,用于角色渲染与场景理解。
- **ORCA 世界模型**——若要潜在"世界模拟器"驱动角色对房间的预期。
- **caveat**:同 SHLAB,具身栈面向物理机器人;大脑模块可复用,动作/传感层非 visionOS 原生。FlagOpen 无 Apple 平台特定移植(预期 Linux/CUDA 优先)。RTX Pro 6000 96GB 在这些模型推理画像内(RoboBrain 32B 舒适承载)。

## SHLAB vs BAAI 对比

| 维度 | 上海AI Lab | 智源 BAAI |
|---|---|---|
| 具身旗舰 | InternVLA-M1(-A1/-N1)、VeBrain | RoboBrain 2.0/RoboOS 2.0/ORCA |
| 空间智能 | InternVLA-M1 空间 grounding | RoboBrain-SpatialTrace、空间推理基准 |
| 数字人/avatar | EchoMimic(蚂蚁主导合作)| Emu/OmniGen(生成),无专门 avatar 产品 |
| 最强开源合作路径 | OpenXLab 贡献 + 经 SJTU 学术合著 | **FlagOpen(Linux Foundation)+ 智源学者 + 国家重点实验室开放课题** |
| 对外国个人友好? | 正式机制有限 | **更好**:FlagOpen 国际开放;智源大会双语友好 |
| 最相关 2026 活动 | 浦江 AI 学术年会 | **智源大会 2026(6/12-13)+ 具身开放日** |

## 置信度（原 09）
- **高**(≥2 源含官网核验):所有 PI 名/头衔;RoboBrain 2.0/RoboOS 2.0/ORCA;InternVLA-M1(arXiv 2510.13778);EchoMimic(arXiv 2407.08136,蚂蚁主导);悟道 1.0/2.0 历史;FlagOpen/Linux Foundation。
- **中**:卢策吾 vs 庞江淼确切现任头衔(皆称"浦器 head"——大概率卢=教研/SJTU 侧,庞=in-house 运营 lead)。
- **未确认/需直接核 arXiv**:InternVLA-A1 arXiv 2601.02456(仅聚合见);InternVLA-N1(无论文 ID,仅 repo);任何独立 SHLAB "InternSpatial"/"Landmark robot" 产品名(疑为 InternVLA-M1 空间模块的非正式称呼)。

---

# 第五部分 · 对接优先级总表

> 按"具身认知 + AR 渲染"对 Vision Pro 虚拟角色项目的综合契合度与可执行性排序。对接摩擦:低 = 有公开实习/开源入口可直接触达;中 = 需经学术关系或联合实验室;高 = 社交驱动、无公开 open-call。

## 按 PI / 团队

| 机构 | PI / 团队 | 强项 | 是否招实习 | 对接摩擦 | 联系入口 |
|---|---|---|---|---|---|
| 清华 AIR | **赵昊组** | talking-head/3DGS avatar/neural rendering/重打光/具身VLA 全栈 | ✅ 招博后+实习(≥6月) | **低** | zhaohao@air.tsinghua.edu.cn ; airhr@air.tsinghua.edu.cn |
| 清华 自动化系 | **刘烨斌** | 数字人重建/生成/全息/GS;DNA-Rendering;学生进 Meta Reality Labs/Apple | 发博后岗 | 中 | [自动化系页](https://www.au.tsinghua.edu.cn/info/1080/3160.htm) |
| 清华 SIGS | **王好谦(GUAVA)** | 单图→3DGS avatar ~0.1s,实时 >50FPS | 产业就绪产出 | 中 | arXiv 2505.03351 |
| 清华 AIR | **周谷越 DISCOVER Lab** | Real2Sim2Real/DISCOVERSE 仿真/AIRBOT | ✅(戴尔乐联合研究院) | 中 | [AIR DISCOVER](https://air.tsinghua.edu.cn/info/1046/1199.htm) |
| 清华 CS | **唐杰 / KEG** | GLM-5.2 开源 LLM;华智冰;智谱管线 | 智谱商业管线 | **低**(开源可得) | [KEG](https://keg.cs.tsinghua.edu.cn/jietang/) |
| 清华 CS | **贾珈** | 情感计算/数字人/语音交互;华智冰共创者 | — | 中 | hcsi.cs.tsinghua.edu.cn/jiajia |
| 清华 CS | **胡事民** | Jittor/JDHR 渲染基建;图形;院士 | 清华-腾讯长期联合实验室 | 中 | cg.cs.tsinghua.edu.cn |
| 清华 自动化系 | **黄高 LEAP** | 具身认知/操控/世界模型;NeurIPS 2025 Best Paper runner-up | ✅ 校企(华为/字节/阿里/腾讯) | 中 | gaohuang.net |
| 清华 IIIS | **高阳 VAR** | VLA/灵巧操控(ViLa/CoPa);千寻智能创业 | — | 中 | [IIIS VAR](https://iiis.tsinghua.edu.cn/kxyj/ktzjs/sjyjqrsys_VAR_.htm) |
| 清华 CS | **孙富春** | 触觉灵巧手/人形/知行体/EIR VP | — | 中 | fcsun@tsinghua.edu.cn |
| 清华(跨院系) | **EIR 研究院** | 机构入口(张涛院长) | — | 中 | **eir.tsinghua.edu.cn** |
| 北大 CFCS | **王亦洲 CVDA** | 因果世界模型/视觉认知/UnrealZoo/人体动作 | ✅ 博雅博士后 | 中 | **yizhou.wang@pku.edu.cn** |
| 北大 CFCS | **王鹤 EPIC Lab** | VLA/灵巧操控/导航;Galbot CTO | ✅ **招实习**(北大-银河通用联合实验室) | **低** | hughw19.github.io |
| 北大 智能学院 | **查红彬** | SLAM/AR 视觉/3D 人脸动画 | — | 中 | [智能学院](https://sai.pku.edu.cn/info/1362/2247.htm) |
| 北大 智能学院 | **朱松纯 / BIGAI** | AGI 框架/通通/认知推理 | — | 中 | **s.c.zhu@pku.edu.cn** |
| 北大 计算机学院 | **ACIR(王韬)** | 情感机器人/表情识别/情绪模型 | 联合实验室 | 中 | acir.pku.edu.cn |
| SJTU MVIG | **卢策吾** | 具身认知/操控/RH20T/"数字人和机器人的具身AI";穹彻智能 | ✅ 开放实习/招聘 | 中 | mvig.org |
| SJTU AI 学院 | **严骏驰** | 世界模型/多模态统一表征(理论深度) | — | 中 | soai.sjtu.edu.cn |
| SJTU AI 学院 | **王延峰(院长)** | MediaBrain/医学影像;dean 门户 | — | 中 | dean 入口 |
| CASIA | **王金桥(紫东太初)** | 全模态大模型(视觉+语音+文本+3D)/开源 | 产业联合体 | 中 | [ia.cas.cn](https://ia.cas.cn/rcdw/yjy/202404/t20240422_7129874.html) |
| ICT-CAS VIPL | **山世光** | 数字人脸/微表情/AU/lip-sync/注视;中科视拓 | ✅ 中科视拓商业化 | 中 | vipl.ict.ac.cn |
| ICT-CAS | **陈熙霖(所长)** | 多模态 HCI;VIPL 前门 | — | 中 | ICT 前门 |
| SIA-CAS | **韩志 / 于海斌** | 物理机器人/具身视觉(仅当驱动物理人形时) | — | 高 | sia.cas.cn |
| USTC | **查正军 MEI-Lab** | 多模态具身智能(感知+生成+具身) | — | 中 | ustc-milab.work |
| USTC | **张燕咏** | 具身感知/边缘部署/实时 OS | — | 中 | — |
| USTC | **张世武** | 人形机器人/仿生软体 | — | 中 | — |
| 上海AI Lab | **OpenRobotLab(庞江淼/曾嘉)** | InternVLA-M1 空间 grounding/VeBrain | 社交驱动 | **高** | InternRobotics GitHub;浦江会议 |
| 上海AI Lab | **EchoMimic** | 音驱肖像/半身动画 | 蚂蚁主导 | 高 | github.com/antgroup/echomimic |
| 智源 BAAI | **王鹏伟(RoboBrain/ORCA)** | RoboBrain 2.0/RoboOS 2.0(MCP)/FlagOpen | ✅ FlagOpen 开源(Linux Foundation) | **低** | **flagopen.baai.ac.cn**;智源大会 |

## 按需求维度（找谁）

| 项目需求 | 首选 | 次选 |
|---|---|---|
| 实时 GS / neural avatar 渲染 | 清华 **赵昊组**、**刘烨斌** | 清华 **GUAVA(王好谦)** |
| LLM / agent 认知大脑 | 清华 **唐杰/GLM**、智源 **BAAI RoboBrain** | CASIA **紫东太初(王金桥)** |
| 数字人脸 / 表情 / lip-sync | ICT **山世光** | 清华 **赵昊(SyncTalk++)**、北大 **ACIR(王韬)** |
| 具身认知 / 因果世界模型 | 北大 **王亦洲** | 北大 **朱松纯/BIGAI** |
| VLA / 灵巧操控 / 动作闭环 | 北大 **王鹤(EPIC)**、清华 **高阳(VAR)** | SJTU **卢策吾(MVIG)**、清华 **黄高(LEAP)** |
| 空间 grounding / 场景理解 | 上海AI Lab **InternVLA-M1/VeBrain** | 北大 **查红彬(SLAM/AR)** |
| 情感 / 情绪环路 | 北大 **ACIR(王韬)**、清华 **贾珈** | ICT **山世光** |
| 音驱肖像动画 | **EchoMimic(蚂蚁主导)** | 清华 **赵昊(SyncTalk++)** |
| Real2Sim2Real 仿真平台 | 清华 AIR **DISCOVER(DISCOVERSE)** | — |
| 原生 Vision Pro / visionOS | **行业空白**(清华/PKU 均无专门实验室) | 自建;框定为"首个学术 Vision Pro 空间 avatar 管线" |

## 最低摩擦对接路径（给外国/个人开发者）

1. **智源 BAAI FlagOpen(Linux Foundation)**——标准开源 PR,无需中国归属,国际开放。最低摩擦。
2. **清华赵昊组**——公开招实习/博后,直接邮箱 `zhaohao@air.tsinghua.edu.cn`。
3. **北大-银河通用联合实验室(王鹤)**——明确招实习,王鹤主页开放入口。
4. **智源大会 2026(6/12-13,北京)**——两大实验室与整个中国具身AI 生态俱在,双语友好。最该参加的单场活动。
5. **北大跨媒体通用人工智能全国重点实验室开放课题**——2025 征集明确列"可泛化的人与物体的全身交互动作生成"(直接相关);需经中国学术 co-PI 合申。
