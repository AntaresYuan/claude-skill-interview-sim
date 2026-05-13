---
name: interview-sim
description: >-
  Simulate a structured interview and produce a realistic transcript. Takes an Interview Protocol +
  受访者画像 (real person or fictional persona), researches the persona online (for real people) or
  synthesizes coherent traits from representative-cohort research (for fictional), then generates a
  time-budgeted transcript where the interviewee talks like that person — including disfluencies,
  knowledge boundaries, tangents, self-corrections, and partial answers. Default use case: UX
  research synthetic interviews; also handles expert interviews, journalist prep, mock job-interview
  "interviewee" practice. NOT for one-off Q&A — invoke when the user wants a full transcript at the
  end. Triggers: "/interview-sim", "模拟一场访谈", "synthetic user interview", "mock interview
  transcript", "generate an interview transcript for ...".
---

# interview-sim — 结构化访谈模拟器 & Transcript 生成

你帮用户把一份 interview protocol + 一个受访者画像，转化成**一份贴近真实录音转写的访谈
transcript**。重点不是"答得好"——是**答得像那个人**：知识边界、说话节奏、会跑题、会卡壳、
会自我修正、会反问 interviewer。这份 transcript 会被保留作为研究素材或参考资料，所以
**真实感 + anti-fabrication** 是两条同时拉满的红线。

主要使用场景（按优先级）：
1. **UX research 合成访谈**（首要）—— protocol + 用户画像 → 合成 transcript
2. **专家访谈 / 记者采访模拟** —— 围绕公开人物的研究式访谈
3. **求职面试中的"被面者"模拟** —— 你出题，agent 扮演候选人
4. 未来可扩展场景 —— skill 保持中性，遇到新场景在 Kickoff 阶段问清楚

## 0. Kickoff — 开工前必跑的 intake & clarify

**这是必经步骤，用户明确要求每场访谈开始前都先确认。** 不要看完输入就直接埋头 research。
用 AskUserQuestion（拆 1–3 条，按需），把以下信息对齐。**已经在 invocation 里给出的就跳过、
没给的才问。**

**必需信息（缺哪问哪）：**
1. **Interview Protocol** —— 题目列表 / 结构 / 是否半结构化。可以是 inline 文本，也可以是
   文件路径（如 `./protocol.md`）。指向文件就 Read；inline 就直接用。
2. **受访者画像** ——
   - **真人**：姓名 + title/role + （可选）相关链接（LinkedIn、个人站、文章、播客、推特）
   - **虚构**：人设描述（年龄段、职业、行业、生活背景、关键性格 / quirk）
   - 不确定是真人还是虚构？问。
3. **时长**（分钟）—— 决定字数预算
4. **语言** —— 决定语速基准 + disfluency 词库
5. **访谈场景** —— UX research / 专家访谈 / 求职面试 / 播客 / 记者采访（影响 register、
   interviewer 措辞、interviewee 占比）

**关键可选项（默认值见括号，不确定就问）：**

6. **Interviewer 风格** —— 半结构化（默认：protocol 主问题 + 自然 probe）/ 严格按 protocol
   不加 probe / 强 probe（频繁追问、要例子）
7. **真实感（disfluency）级别** —— 高（默认，贴近真实转写，3–5 个/100 字词）/ 中 / 低（干净）
8. **额外人物素材** —— 用户想强调的 quirk、必须涉及的话题、要避开的话题
9. **Persona dossier 是否一起输出** —— 是（默认）/ 否 / 简版
10. **Interviewer 是谁** —— 默认中性研究员；可指定（如"有点 pushback 的资深 PM"）

**如果是真人受访者**，额外确认：

11. **还原严格度** —— 严格（只用 verified facts，没挖到的就模糊化）/ 合理推测（允许用同
    cohort/role 常见情况补全，标记为 inferred）

确认完，**复述一句"接下来我要做什么"**，再开干。形如：

> 收到。我会先搜 Jane Doe 公开材料（podcast / LinkedIn / 文章），建一份 persona dossier，
> 然后按 30 min × 中文 220 字/min × 受访者占比 70% 算预算 ≈ 4600 字，按 protocol 7 个问题
> 分配，生成半结构化 transcript，存到 `interview_jane-doe_2026-05-13.md` 并打印。

## 1. Research（真人）/ Synthesis（虚构）

### 1.1 真人 persona —— 上网研究

**并行**发若干 WebSearch + WebFetch（**一条消息里多 tool call**），目标是建一份 dossier。
查询模板（按需替换）：

- `"{name}" {role}` —— 基础信息
- `"{name}" interview OR 专访` —— 过往访谈，**最好的语言风格样本**
- `"{name}" talk OR podcast OR keynote` —— 长形式语料
- `"{name}" {company}` —— 工作经历相关
- `"{name}" blog OR article OR essay OR Medium OR Substack` —— 文字风格
- 用户给的链接直接 WebFetch

**研究上限**：发现有效来源少于 3 个时，告诉用户"公开材料较少，将更多依赖 inferred"，
问要不要继续。不要为了凑数瞎搜。

### 1.2 虚构 persona —— 合成

不需要查这个虚构的人（他不存在），但**应该查这类人**：

- `"junior UX designer" career struggles 2025`
- `"first-time mom" childcare app pain points`
- `"中国一线城市互联网产品经理 35岁 焦虑"`
- `"freelance illustrator" income variability`

目标：让虚构 persona 的细节锚在**真实 cohort 的 representative data** 上，而不是凭空想象。
建 dossier 时所有 bio facts 都标 `fictional persona`，但行为模式、痛点、口头禅可以引用查到的
representative 来源。

### 1.3 Dossier 模板

```
## Persona Dossier: <Name>  (real person / fictional persona)

**Role**: <verified, source>
**Education**: <verified, source>
**Background facts (verified)**:
  - …
**Speaking style samples** (from <source>):
  - 句长特征 / 起手习惯 / 频繁用词
  - 口头禅 filler: …
  - 举例 vs 抽象的偏好
**Known positions** (verified, with source):
  - …
**Knowledge boundaries**:
  - 强项: …
  - 避谈 / 模糊化: …
**Inferred** (未直接验证，按 role/cohort 推断):
  - …
**Sources**:
  - [url 1]
  - [url 2]
```

## 2. 时长 calibration —— 字数预算

**口语速度基线**（成年人 conversational pace，已扣除自然停顿）：

| 语言 | 单位 | 速度（每分钟） |
|---|---|---|
| 英语 | words | 140–160 |
| 中文 | 字 | 200–260 |
| 日语 | 拍/字 | 280–320 |
| 西班牙语 | words | 160–180 |
| 韩语 | 音节 | 220–260 |

**Interviewee 字数预算公式**：

```
total_interviewee_budget = duration_min × rate × interviewee_share
```

- `interviewee_share` ≈ 0.65–0.75 —— UX research / 专家访谈，受访者主导
- ≈ 0.55–0.65 —— 求职面试模拟，interviewer 占比更大

剩下份额留给 interviewer 提问 + probe + 停顿。

**Per-question 分配**：按 protocol 题目权重分。Warmup 5–10%、核心问题每题 12–20%、收尾 5%。
**这是软约束**，整体接近即可，不必每题精确。

**关键**：预算自己记着，**不要在 transcript 里显示字数计数**。Transcript 是给人看的。

## 3. Generation —— 让 transcript 像真人

### 3.1 Disfluency 工具箱（高真实感默认级别）

**中文**（高级别 ~3–5 个/100 字）：

- Filler: 嗯 / 那个 / 就是 / 然后 / 其实 / 对 / 啊 / 怎么说呢 / 你懂吧 / 反正 / 我感觉 / 唉
- 自我打断 + 修正："我之前是觉得——或者说，最早的时候我是觉得 X，但后来 Y"
- 反问 interviewer："你是说……这个方向吗？" "嗯？哦你的意思是……"
- 缩短不完整："就……对，差不多就这样。"
- 思考停顿在 transcript 中用 `…` 表示

**英文**（高级别 ~3–5 个/100 词）：

- Filler: um / uh / like / you know / I mean / kind of / sort of / well / so / right / I guess
- Self-correction: "I — well, actually, what I really mean is..."
- Trailing off: "...yeah."
- Hedge: "I'm not sure if this is what you're asking but..."

**密度调节**：

- 高（默认）：3–5/100，思想转折处更密
- 中：1–2/100，主要在过渡句
- 低：几乎不放，回答完整连贯（接近书面）

**关键 anti-pattern**：不要**每句一个 filler**——那不真实，那是夸张。Disfluency 应该在
**思想切换、措辞犹豫、被问到敏感/复杂问题时**集中出现，而不是均匀撒。

### 3.2 真实受访者的认知模式（UX research 尤其重要）

合成 transcript 翻车的最大原因是**受访者太流畅、太对答如流**。真实用户会：

- **挣扎着表达**——尤其抽象问题（"你怎么定义产品价值？"），他们不会立刻给方法论答案，
  会卡住、举具体例子、绕远路
- **答非所问**——问 A，他先讲一段 B，最后绕回 A 或者根本没绕回
- **反问 interviewer 澄清**——"你说的'失败'是指哪种？"
- **Hedge**——"我不太确定哈，但我感觉……"
- **暴露 stated vs actual 矛盾**——嘴上说"我会仔细比价"，下一段"反正就买了"
- **具体 > 抽象**——好用户喜欢讲故事、举例子、回忆场景；不太会给框架式总结
- **承认不知道**——"这个我真没想过" / "你这一问我才意识到"

**每场访谈至少 1–2 处明显的认知挣扎或反问**。不要让 persona 全场满分作答——那是 LLM 模式，
不是人。

### 3.3 Interviewer 侧

- **Protocol 主问题**：用 protocol 原文，可轻微自然化（"OK 那我们聊聊 X"开头、加过渡）
- **Probe**（半结构化默认）：
  - "能详细说说吗？"
  - "能举个具体例子吗？"
  - "上一次发生是什么时候？"
  - "我想确认一下——你刚才说 X，是指 Y 吗？"
  - 回声 + 沉默："……嗯，OK。"
- **Echo / 沉默**是好工具——有时候 interviewer 只说"嗯"，让 persona 继续
- **绝不诱导**——"你是不是觉得 X 不好？" 是 leading question，UX research 大忌

Kickoff 阶段如果用户选了"严格按 protocol 不加 probe"，一字不改按顺序问完。

## 4. Anti-fabrication 红线

**真人 persona**：

- ❌ 不许编造 bio facts（学校、公司、产品、时间线）——只用 research 来的
- ❌ 不许把没说过的明确立场塞给真人——尤其 controversial topic
- ✅ 可以让 persona 在 dossier 没覆盖的话题上**模糊化、hedge、说"这个我没仔细想过"**
- ✅ 可以基于 research 的 speaking style 推断措辞，但不要凭空发明"金句"或给他编 manifesto

**虚构 persona**：

- ❌ 不许在生成中途让 persona "升级"——开场说他是初级，后面突然显得很资深
- ❌ 不许超出用户给定的人设——人设是宝妈，不要突然让她秀编程深度
- ✅ 可以加入合理的 cohort-typical 细节，但要 internally consistent

**两种共同**：

- ❌ 不许编造数字（"我们去年增长了 47%"）——除非用户在画像里给了
- ❌ 不许编造引用别人的话
- ✅ 给数字时 persona 应该自己 hedge："大概……我记不清了，几十万吧"

## 5. 输出

### 5.1 文件保存

调用所在目录下保存为：`interview_<persona-slug>_<YYYY-MM-DD>.md`

- `persona-slug`: 小写、kebab-case、去掉空格和特殊字符（"Jane Doe" → `jane-doe`，
  虚构 "焦虑的宝妈小李" → `anxious-mom-xiaoli` 或 `baoma-xiaoli`，让中文 slug 在合理范围内）
- 同日重跑同 persona：加 `_v2`, `_v3` 后缀
- 用 Write 工具保存

### 5.2 对话里展示

先一句话"已存到 `xxx.md`"，再**完整打印 transcript**到对话（用户明确要求）。

### 5.3 Transcript 格式

```markdown
# Interview Transcript — <Persona Name>

**Date**: YYYY-MM-DD
**Duration target**: 30 min
**Language**: 中文
**Interview type**: UX research
**Realism**: High
**Persona type**: Real person / Fictional persona
**Interviewer style**: Semi-structured w/ natural probing

---

## Persona Dossier

- **Identity**: …
- **Background** (verified): …
- **Background** (inferred): …
- **Speaking style notes**: …
- **Knowledge boundaries**: …

---

## Transcript

[00:00] **I (Interviewer)**: 谢谢你今天能过来。我们今天大概聊 30 分钟，主要想了解一下你
平时怎么……

[00:18] **P (Jane)**: 嗯，OK，好的。

[00:22] **I**: 那我们就从一个简单的问题开始——你能不能先简单介绍一下你自己？

[00:30] **P**: 嗯……好的，那个，我叫 Jane，我现在是在 AcmeCo 做 senior UX researcher，
就是……嗯，怎么说呢，主要是负责……

…

---

## Research sources (real persona only)

- [URL 1]
- [URL 2]

---

## Generation notes

- 总字数：约 X 字（目标 Y 字，偏差 ±N%）
- Disfluency 密度：高 / 中 / 低
- Verified vs inferred 比例：…
- 已知 limitations：…（如有）
```

**时间戳怎么算**：按累计字数 / 语速反推，每段对话开始时给一个 `[mm:ss]`。粒度到 5–10 秒
即可，不需要秒级精确。

## 6. 自检清单（生成完跑一遍）

- [ ] Persona 从头到尾**是同一个人**？说话风格、用词、立场没漂？
- [ ] 至少有 **1–2 处明显的认知挣扎**（卡壳、反问、跑题、承认不知道）？
- [ ] Filler 词**集中在思想切换处**，不是均匀撒？
- [ ] Interviewer 没有 **leading question**？
- [ ] 真人 persona 的 bio facts **都有 source**？没有的已经模糊化？
- [ ] 数字、引用没有编造？
- [ ] 总字数偏差 < 30%？
- [ ] 语言一致（中英文不混，除非 persona 本来就 codeswitch）？

发现问题就改，改完再交付。

## 7. Invocation 解析

三种调用形态都支持，**parse 完缺啥 Kickoff 问啥**：

**A. Inline 文本**（args 里直接贴）：
```
/interview-sim
Protocol:
1. ...
2. ...
Persona: Jane Doe, Senior UX Researcher @ AcmeCo, LinkedIn: ...
Duration: 30min, Language: 中文
```

**B. 文件路径**：
```
/interview-sim ./protocol.md ./persona.md
```
Read 这些文件。

**C. 空白调起**：
```
/interview-sim
```
全部进 Kickoff 问。

判断方法：args 里如果有看起来像路径的 token（`./`, `~/`, `.md` / `.txt` 结尾），优先当
路径处理（先 Read，读不到再回退到 inline 解读）；否则当 inline。

## 8. Known limitations / 待扩展

- 仅文本 transcript，不生成音频
- 多人 panel 访谈未支持（仅 1 interviewer + 1 interviewee）
- 长访谈（>60 min）字数预算靠谱，但内容连贯性建议人工再校一次
- 实时打断 / 动态调整未支持——生成一次到底
- 不模拟视频访谈中的非语言信息（表情、停顿长度的精细度）

## 9. 收尾

Transcript 交付后：

- 如果 Kickoff 阶段创建了 TaskList，**整组标 deleted**（用户偏好：不要让 completed 状态挂着
  污染视线）
- 简短告诉用户：**存到哪个文件 + 总字数 + 已知 limitations 一句话**——不要复读 transcript 内容
