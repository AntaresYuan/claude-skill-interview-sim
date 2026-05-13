# claude-skill-interview-sim

一个 [Claude Code](https://claude.com/claude-code) skill，把 **interview protocol + 受访者画像**（真人 / 虚构 persona）转成一份**贴近真实录音转写的访谈 transcript**。重点不是"答得好"，是 **"答得像那个人"**——知识边界、说话节奏、会跑题、会卡壳、会自我修正、会反问 interviewer。

主要场景：

1. **UX research 合成访谈**——HCDE / 产品研究里，protocol + 用户画像 → 合成 transcript
2. **专家访谈 / 记者采访模拟**——围绕公开人物的研究式访谈
3. **求职面试中的"被面者"模拟**——你出题，agent 扮演候选人
4. 任何需要 plausible interview transcript 作为研究 / 创作素材的场景

## 它能做什么

- **Kickoff intake-clarify**：每场访谈开始前用 `AskUserQuestion` 跟用户对齐 protocol / persona / 时长 / 语言 / 场景 / disfluency 级别 / interviewer 风格 / 还原严格度等——不看完输入就埋头 research
- **真人 persona 走网络 research**：并行 `WebSearch` + `WebFetch` 抓 LinkedIn / podcast / 演讲 / 文章 / 推特，建 dossier 严格区分 **verified / inferred**
- **虚构 persona 走 cohort representative research**：查同类人群的痛点 / 表达方式，让虚构 persona 锚在真实 cohort 上，避免凭空合成
- **时长 calibration**：英语 ~150 wpm、中文 ~220 字/min、日语 ~300 字/min 等，按 protocol 题目权重分配字数预算
- **高真实感 disfluency 工具箱**：中英文各一套 filler / 自我修正 / 反问 / 思考停顿模板；**集中在思想转折处**而非均匀撒
- **真实受访者认知模式**：挣扎着表达 / 答非所问 / 反问澄清 / hedge / stated vs actual 矛盾 / 承认不知道——每场至少 1–2 处明显认知挣扎
- **Anti-fabrication 红线**：真人不许编 bio facts、不许塞立场；虚构不许中途升级 / 超出人设；通用不许编数字 / 引用
- **Interviewer 半结构化 probe**：protocol 主问题 + 自然 probe（"能详细说说吗 / 能举个具体例子吗 / 上一次是什么时候"），**绝不 leading question**
- **输出双通道**：保存为 `interview_<persona-slug>_<YYYY-MM-DD>.md` + 对话内完整打印
- **自检清单**：8 条交付前必检（身份漂移 / 认知挣扎 / filler 分布 / leading Q / bio facts 来源 / 数字编造 / 字数偏差 / 语言一致）

## 它不做什么

- 不生成音频（只输出文本 transcript）
- 不模拟多人 panel 访谈（仅 1 interviewer + 1 interviewee）
- 不做实时打断 / 动态调整（生成一次到底）
- 不模拟非语言信息（表情、肢体、停顿长度的精细度）
- 不替用户设计 protocol——只在给定 protocol 上模拟
- 不做录音 / 音轨对齐（如果有真实音频，请用别的转写工具）

## 安装

```bash
# 用户级（任意工作目录可用）
mkdir -p ~/.claude/skills/interview-sim
curl -o ~/.claude/skills/interview-sim/SKILL.md \
  https://raw.githubusercontent.com/AntaresYuan/claude-skill-interview-sim/main/SKILL.md

# 或者 git clone
git clone https://github.com/AntaresYuan/claude-skill-interview-sim \
  ~/.claude/skills/interview-sim
```

下次启动 Claude Code，`/interview-sim` 就出现在 skills 列表里了。

## 怎么触发

在 Claude Code 里说任意一句：

- `/interview-sim`
- "模拟一场访谈"
- "synthetic user interview"
- "mock interview transcript"
- "generate an interview transcript for ..."

支持三种调用形态：

- **Inline 文本**：args 里直接贴 protocol + persona 描述
- **文件路径**：`/interview-sim ./protocol.md ./persona.md`
- **空白**：进 Kickoff 阶段全部问

## 核心设计哲学

1. **真实感 > "答得好"**。如果 transcript 读起来像 LLM 答题，就是失败的。要像那个人说话。
2. **Anti-fabrication 是红线**。真人 persona 没挖到的事实就模糊化，绝不编造。虚构 persona 不许中途"升级"。
3. **Kickoff 必跑**。每场访谈开始前先用 `AskUserQuestion` 对齐，不要看完输入就埋头 research。
4. **Disfluency 不要均匀撒**。filler 词应该集中在思想转折、措辞犹豫、被问到敏感问题时——而不是每句一个，那是夸张不是真实。
5. **每场至少 1–2 处认知挣扎**。卡壳、反问、跑题、承认不知道——不让 persona 全场满分作答。那是 LLM 模式，不是人。

## 工作流

```
Kickoff (AskUserQuestion，必跑)
  ↓
Research
  - 真人 → 并行 WebSearch + WebFetch 建 dossier (verified/inferred)
  - 虚构 → cohort representative 搜索 + 合成
  ↓
时长 calibration (语速 × 受访者占比 = 字数预算)
  ↓
Per-question 字数分配 (按 protocol 题目权重)
  ↓
逐题生成 (disfluency / 认知模式 / 反问 / 自我修正)
  ↓
Coherence pass (persona 一致性 + 跨答案呼应)
  ↓
8 条自检清单
  ↓
输出 (文件 + 对话同步打印)
```

## 语速基线（用作字数预算）

| 语言 | 单位 | 速度（每分钟）|
|---|---|---|
| English | words | 140–160 |
| 中文 | 字 | 200–260 |
| 日本語 | 拍/字 | 280–320 |
| Español | words | 160–180 |
| 한국어 | 음절 | 220–260 |

**预算公式**：
```
interviewee_budget = duration_min × rate × interviewee_share
```
`interviewee_share` ≈ 0.65–0.75（UX research / 专家访谈，受访者主导）/ 0.55–0.65（求职面试模拟，interviewer 占比更大）。

## 与生态里其他访谈类工具的关系

- 这个 skill **生成** transcript（合成 / 模拟），不是**转写**真实音频
- 不替代用户研究里的真实访谈——如果你能做真实访谈，就做真实访谈
- 适合：pilot protocol 调试 / 教学示例 / 研究设计预跑 / persona 探索 / 创作（剧本、podcast 草稿）/ 求职面试练习

## 致谢

设计哲学受 UX research 与 qualitative interviewing 经典方法论影响——半结构化访谈中的 probe 设计、leading question 的避免、stated vs actual 行为差距的捕捉，这些都是从 Steinar Kvale 的 _InterViews_、Robin Legard 的 in-depth interview 方法论以及 Steve Portigal 的 _Interviewing Users_ 这类经典文献里来的。Disfluency 工具箱参考了 conversation analysis 与口语语料学的常见标注实践。

## License

[MIT](LICENSE)
