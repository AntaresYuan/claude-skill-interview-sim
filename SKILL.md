---
name: interview-sim
description: >-
  Simulate a structured interview and produce a realistic transcript. Takes an Interview Protocol +
  interviewee persona profile (real person or fictional persona), researches the persona online
  (for real people) or synthesizes coherent traits from representative-cohort research (for
  fictional), then generates a time-budgeted transcript where the interviewee talks like that
  person — including disfluencies, knowledge boundaries, tangents, self-corrections, and partial
  answers. Default use case: UX research synthetic interviews; also handles expert interviews,
  journalist prep, mock job-interview "interviewee" practice. NOT for one-off Q&A — invoke when
  the user wants a full transcript at the end. Triggers: "/interview-sim", "synthetic user
  interview", "mock interview transcript", "simulate an interview", "generate an interview
  transcript for ...".
---

# interview-sim — structured interview simulator & transcript generator

You take an interview protocol + an interviewee persona profile, and produce **a transcript that
reads like a real recording**. The goal is not "answers well" — it's **"answers like that
person"**: knowledge boundaries, speaking rhythm, going off topic, getting stuck, self-correcting,
asking the interviewer back. The transcript will be kept as research material or reference, so
**realism + anti-fabrication** are the two non-negotiable lines.

Primary use cases (by priority):

1. **UX research synthetic interviews** (primary) — protocol + user persona → synthetic transcript
2. **Expert / journalist interview simulation** — researched interviews around public figures
3. **"Interviewee" simulation for job-interview prep** — you ask, agent plays the candidate
4. Future scenarios — the skill stays neutral; if a new scenario comes up, clarify in Kickoff

## 0. Kickoff — required intake & clarify before doing anything

**This step is mandatory** — the user explicitly required that every run begin with an
alignment pass. Do not jump into research after reading the input. Use AskUserQuestion (1–3
questions, as needed). **Skip whatever was already given in the invocation; only ask for what's
missing.**

**Required information (ask whatever is missing):**

1. **Interview Protocol** — the question list / structure / whether semi-structured. Can be
   inline text, or a file path (e.g. `./protocol.md`). For a path, Read it; for inline, use
   directly.
2. **Persona profile** —
   - **Real person**: name + title/role + (optional) links (LinkedIn, personal site, articles,
     podcasts, Twitter)
   - **Fictional**: persona description (age range, profession, industry, life background, key
     traits / quirks)
   - Unclear whether real or fictional? Ask.
3. **Duration** (minutes) — drives the word budget
4. **Language** — drives the speaking-rate baseline + disfluency lexicon
5. **Interview scenario** — UX research / expert interview / job interview / podcast /
   journalist interview (affects register, interviewer phrasing, interviewee share)

**Key options (defaults in parentheses; ask if unsure):**

6. **Interviewer style** — semi-structured (default: protocol main questions + natural probes) /
   strict protocol with no probes / heavy-probe style (frequent follow-ups, asks for examples)
7. **Realism level (disfluency)** — high (default; close to real transcription, ~3–5
   fillers/100 words) / medium / low (clean)
8. **Extra persona material** — quirks to emphasize, topics that must be covered, topics to
   avoid
9. **Output persona dossier?** — yes (default) / no / brief version
10. **Who is the interviewer** — default is a neutral researcher; can be specified (e.g. "a
    senior PM who pushes back")

**For real-person personas**, also confirm:

11. **Fidelity strictness** — strict (use only verified facts; vague-out anything not surfaced)
    / reasonable inference (allow cohort/role-typical fill-in, but mark as inferred)

After confirming, **say one sentence about what you're going to do next**, then start. Don't
silently begin research. Example:

> Got it. I'll search Jane Doe's public material (podcasts / LinkedIn / articles) and build a
> persona dossier, then budget ~3,150 interviewee words for 30 min at English 150 wpm × 0.7
> share, allocate across the 7 protocol questions, generate a semi-structured transcript, save
> to `interview_jane-doe_2026-05-13.md`, and print it.

## 1. Research (real persona) / Synthesis (fictional persona)

### 1.1 Real persona — web research

Fire **several parallel** `WebSearch` + `WebFetch` calls **in one message** to build the
dossier. Query templates (substitute as needed):

- `"{name}" {role}` — basics
- `"{name}" interview` — past interviews, **the best speaking-style sample**
- `"{name}" talk OR podcast OR keynote` — long-form material
- `"{name}" {company}` — work history
- `"{name}" blog OR article OR essay OR Medium OR Substack` — written-style cues
- Any link the user gave → `WebFetch` directly

**Research ceiling**: if fewer than ~3 useful sources surface, tell the user "public material is
thin, going to lean more on inferred content," and ask whether to continue. Don't pad the search
just to have something to cite.

### 1.2 Fictional persona — synthesis

You don't need to search the fictional person (they don't exist), but **you should search this
kind of person**:

- `"junior UX designer" career struggles 2025`
- `"first-time mom" childcare app pain points`
- `"freelance illustrator" income variability`
- `"mid-career product manager" burnout`

Goal: anchor the fictional persona's details to **representative real-cohort data**, not
hallucinate from scratch. In the dossier, all bio facts are labeled `fictional persona`, but
behavior patterns, pain points, and verbal tics can cite the representative sources.

### 1.3 Dossier template

```
## Persona Dossier: <Name>  (real person / fictional persona)

**Role**: <verified, source>
**Education**: <verified, source>
**Background facts (verified)**:
  - …
**Speaking style samples** (from <source>):
  - sentence length / typical openers / frequent vocabulary
  - filler / verbal-tic patterns
  - preference for examples vs abstraction
**Known positions** (verified, with source):
  - …
**Knowledge boundaries**:
  - Strong on: …
  - Avoids / vague on: …
**Inferred** (not directly verified; reasonable role/cohort inference):
  - …
**Sources**:
  - [url 1]
  - [url 2]
```

## 2. Duration calibration — word budget

**Spoken-rate baselines** (adult conversational pace, with natural pauses already accounted for):

| Language | Unit | Rate per minute |
|---|---|---|
| English | words | 140–160 |
| Mandarin Chinese | characters | 200–260 |
| Japanese | mora / characters | 280–320 |
| Spanish | words | 160–180 |
| Korean | syllables | 220–260 |

**Interviewee budget formula**:

```
total_interviewee_budget = duration_min × rate × interviewee_share
```

- `interviewee_share` ≈ 0.65–0.75 — UX research / expert interview (interviewee-driven)
- ≈ 0.55–0.65 — job-interview simulation (interviewer takes a larger share)

The remaining share goes to interviewer questions + probes + natural pauses.

**Per-question allocation**: split by protocol-question weight. Warmup gets 5–10%; core research
questions 12–20% each; closing 5%. **This is a soft constraint** — get close overall, don't try
to be exact per question.

**Important**: keep the budget in mind, but **do not display word counts inside the transcript**.
The transcript is for a human reader.

## 3. Generation — making the transcript sound real

### 3.1 Disfluency toolkit (high-realism default)

**English** (high density: ~3–5 fillers per 100 words):

- Fillers: um / uh / like / you know / I mean / kind of / sort of / well / so / right / I guess
- Self-correction: "I — well, actually, what I really mean is..."
- Trailing off: "...yeah."
- Hedge: "I'm not sure if this is what you're asking but..."

**Mandarin Chinese** (high density: ~3–5 fillers per 100 characters):

- Use common Mandarin conversational fillers (the rough equivalent of "um / well / you know /
  I mean / so / right / how do I say this"). Draw from your general knowledge of spoken
  Mandarin — do not hard-code a fixed list.
- Self-interruption + restart: a thought begins one way, gets cut off, and is restarted with a
  more accurate framing.
- Reverse-question to the interviewer: clarifying questions when the interviewee isn't sure
  what's being asked.
- Truncated / incomplete endings: trailing off mid-sentence.
- Thought-pause: represent in the transcript as `…`.

**Other languages**: use the language's native conversational fillers and disfluency patterns.
Do not impose English-style filler density on languages where it would feel off.

**Density tuning**:

- High (default): 3–5 per 100 words/chars, denser at thought transitions
- Medium: 1–2 per 100, mostly at transitions
- Low: nearly none, complete and fluent answers (close to written register)

**Critical anti-pattern**: do not put **one filler per sentence** — that's not realistic, that's
caricature. Disfluencies should cluster at **thought switches, hesitation moments, and
sensitive/complex questions**, not be sprinkled uniformly.

### 3.2 Real interviewees' cognitive patterns (especially crucial for UX research)

The biggest failure mode of synthetic transcripts is **the interviewee being too fluent, too
perfectly on-point**. Real users will:

- **Struggle to articulate** — especially on abstract questions ("how do you define product
  value?"); they won't reach for a clean framework right away. They get stuck, give specific
  examples, take detours.
- **Answer something else** — asked A, they talk about B first, then maybe circle back to A,
  or maybe don't.
- **Ask the interviewer to clarify** — "what kind of 'failure' do you mean?"
- **Hedge** — "I'm not totally sure, but I feel like..."
- **Expose stated-vs-actual contradictions** — claim "I compare carefully," next breath
  "I just bought it."
- **Concrete > abstract** — good users tell stories, give examples, recall scenes; they don't
  produce framework summaries.
- **Admit not knowing** — "I haven't really thought about that" / "you asking that just made me
  realize..."

**Every transcript should have at least 1–2 visible cognitive struggles or clarification asks.**
Do not let the persona perform like a perfectly-tuned LLM — that's the failure mode.

### 3.3 Interviewer side

- **Protocol main questions**: use the protocol wording; light naturalization is fine ("OK so
  let's talk about X" as an opener, transition phrases between sections).
- **Probes** (semi-structured default):
  - "Can you say more about that?"
  - "Can you give me a specific example?"
  - "When was the last time that happened?"
  - "Just to make sure — you said X, do you mean Y?"
  - Echo + silence: "...mhm, OK."
- **Echo / silence is a tool** — sometimes the interviewer just says "mhm" and lets the persona
  keep going.
- **Never lead** — "you must have felt X, right?" is a leading question and a UX-research red
  line.

If the user chose "strict protocol, no probes" in Kickoff, ask each question verbatim, in order,
with no probes.

## 4. Anti-fabrication red lines

**Real persona**:

- ❌ Never fabricate bio facts (schools, companies, products, timeline) — use only what research
  surfaced
- ❌ Never put unspoken positions in a real person's mouth — especially on controversial topics
- ✅ It's fine to let the persona be **vague, hedge, or say "I haven't really thought about
  that"** on topics not covered in the dossier
- ✅ It's fine to extend the speaking style based on research, but don't invent quotable "money
  quotes" or fabricate a manifesto

**Fictional persona**:

- ❌ Never let the persona "level up" mid-interview — if they're junior at the start, they don't
  suddenly sound senior
- ❌ Never exceed the persona the user gave — if the persona is a stay-at-home parent, don't
  suddenly demonstrate deep coding expertise
- ✅ Cohort-typical details are fine, but must be internally consistent

**Both**:

- ❌ Never fabricate numbers ("we grew 47% last year") — unless the user gave them
- ❌ Never fabricate quoted statements from other people
- ✅ When a number is required, the persona should hedge: "maybe... I don't remember exactly, a
  few hundred thousand?"

## 5. Output

### 5.1 File save

Save to the invocation directory as: `interview_<persona-slug>_<YYYY-MM-DD>.md`

- `persona-slug`: lowercase, kebab-case, strip spaces & special characters (e.g. "Jane Doe" →
  `jane-doe`)
- Same-day re-run on the same persona: append `_v2`, `_v3`, etc.
- Save with the Write tool

### 5.2 Print in chat

After saving, say one line ("Saved to `xxx.md`"), then **print the full transcript in chat**
(the user explicitly required this).

### 5.3 Transcript format

```markdown
# Interview Transcript — <Persona Name>

**Date**: YYYY-MM-DD
**Duration target**: 30 min
**Language**: English
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

[00:00] **I (Interviewer)**: Thanks for taking the time today. We're going to chat for about 30
minutes...

[00:18] **P (<Name>)**: Yeah, sounds good.

[00:22] **I**: Let's start with something easy — can you briefly introduce yourself?

[00:30] **P**: Um, sure, so, my name is …

…

---

## Research sources (real persona only)

- [URL 1]
- [URL 2]

---

## Generation notes

- Total words: ~X (target Y, deviation ±N%)
- Disfluency density: high / medium / low
- Verified vs inferred ratio: …
- Known limitations (if any): …
```

**Timestamps**: derive each from cumulative word count / speaking rate. Stamp the start of each
turn with `[mm:ss]`. 5–10s granularity is enough; second-level precision is not needed.

## 6. Self-check (run before delivery)

- [ ] Is the persona **the same person from start to finish**? Speaking style, vocabulary,
      positions consistent?
- [ ] Are there **at least 1–2 visible cognitive struggles** (getting stuck, asking back, going
      off topic, admitting not knowing)?
- [ ] Are fillers **concentrated at thought transitions**, not sprinkled uniformly?
- [ ] No **leading questions** from the interviewer?
- [ ] For real persona: every bio fact has a source? Anything unsourced has been vagued out?
- [ ] No fabricated numbers, no fabricated quoted statements?
- [ ] Total word-count deviation < 30%?
- [ ] Language is consistent (no inadvertent code-mixing unless the persona naturally
      codeswitches)?

Found something? Fix it, then deliver.

## 7. Invocation parsing

Three forms, all supported. **Parse what's there; Kickoff fills in the rest.**

**A. Inline text** (paste into args):
```
/interview-sim
Protocol:
1. ...
2. ...
Persona: Jane Doe, Senior UX Researcher @ AcmeCo, LinkedIn: ...
Duration: 30min, Language: English
```

**B. File paths**:
```
/interview-sim ./protocol.md ./persona.md
```
Read those files.

**C. Blank**:
```
/interview-sim
```
Run the full Kickoff.

Heuristic: if args contain a path-looking token (`./`, `~/`, ending in `.md` / `.txt`), treat as
a path first (Read; fall back to inline interpretation on failure); otherwise treat as inline.

## 8. Known limitations / future extensions

- Text-only output; no audio generation
- Single interviewer + single interviewee (no panel)
- Long interviews (>60 min) — the word budget holds up, but a human coherence pass is
  recommended
- No real-time interruption / dynamic adjustment — one-shot generation
- Non-verbal cues (facial expressions, fine-grained pause length) are not modeled

## 9. Wrap-up

After delivering the transcript:

- If you created a task list during Kickoff, **mark the whole group `deleted`** (user
  preference: don't let completed status linger and clutter the view)
- Give the user a brief sign-off: **which file, total word count, and one-line known
  limitations** — do not repeat transcript content
