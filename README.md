# claude-skill-interview-sim

A [Claude Code](https://claude.com/claude-code) skill that turns an **interview protocol + an
interviewee persona profile** (real person or fictional) into **a transcript that reads like a
real recording**. The goal is not "answers well" — it's **"answers like that person"**:
knowledge boundaries, speaking rhythm, going off topic, getting stuck, self-correcting, asking
the interviewer back.

Primary use cases:

1. **UX research synthetic interviews** — protocol + user persona → synthetic transcript for HCDE
   / product research
2. **Expert / journalist interview simulation** — researched interviews around public figures
3. **"Interviewee" simulation for job-interview prep** — you play interviewer, agent plays
   candidate
4. Any scenario that needs a plausible interview transcript as research or creative material

## What it does

- **Kickoff intake-clarify**: every run begins with an `AskUserQuestion` pass aligning protocol /
  persona / duration / language / scenario / disfluency level / interviewer style / fidelity
  strictness — the skill does not jump straight into research after reading the input
- **Real-person research**: parallel `WebSearch` + `WebFetch` against LinkedIn / podcasts / talks /
  articles / Twitter, building a dossier that strictly separates **verified vs inferred**
- **Fictional-persona synthesis**: search representative-cohort data (pain points, verbal style
  for the persona's demographic and role), anchor the fictional persona to real cohort signal
  rather than hallucinating from scratch
- **Duration calibration**: English ~150 wpm, Mandarin Chinese ~220 chars/min, Japanese ~300
  chars/min, etc.; allocate word budget across protocol questions by weight
- **High-realism disfluency toolkit**: native filler / self-correction / reverse-question /
  thought-pause patterns per language, **clustered at thought transitions** rather than
  sprinkled uniformly
- **Real-user cognitive patterns**: articulation struggle, answering something else, asking the
  interviewer to clarify, hedging, stated-vs-actual contradictions, admitting not knowing —
  every transcript has at least 1–2 visible cognitive struggles
- **Anti-fabrication red lines**: real persona — never fabricate bio facts or unspoken
  positions; fictional persona — no mid-interview "level-up", no exceeding the given profile;
  both — no fabricated numbers, no fabricated quoted statements
- **Semi-structured interviewer probes**: protocol main questions + natural probes ("can you say
  more", "can you give a specific example"); **never leading questions**
- **Dual output**: saves `interview_<persona-slug>_<YYYY-MM-DD>.md` to the working directory and
  also prints the full transcript in chat
- **Self-check pass**: 8-point checklist before delivery (persona consistency / cognitive
  struggle present / filler distribution / leading questions / source coverage / fabricated
  numbers / word-count deviation / language consistency)

## What it does NOT do

- No audio generation (text transcript only)
- No panel simulation (one interviewer + one interviewee)
- No real-time interruption or dynamic adjustment (one-shot generation)
- No modeling of non-verbal cues (facial expressions, fine-grained pause length)
- Does not design the protocol for you — only simulates against a given protocol
- Not a transcription tool — if you have real audio, use a real transcriber

## Install

```bash
# User-level (works from any working directory)
mkdir -p ~/.claude/skills/interview-sim
curl -o ~/.claude/skills/interview-sim/SKILL.md \
  https://raw.githubusercontent.com/AntaresYuan/claude-skill-interview-sim/main/SKILL.md

# Or git clone
git clone https://github.com/AntaresYuan/claude-skill-interview-sim \
  ~/.claude/skills/interview-sim
```

Restart Claude Code and `/interview-sim` will show up in the skills list.

## How to trigger

In Claude Code, say any of:

- `/interview-sim`
- "Simulate an interview"
- "Synthetic user interview"
- "Mock interview transcript"
- "Generate an interview transcript for ..."

Three invocation forms:

- **Inline text**: paste the protocol + persona directly into the args
- **File paths**: `/interview-sim ./protocol.md ./persona.md`
- **Blank**: enter the full Kickoff and answer the questions

## Core design philosophy

1. **Realism beats correctness.** If the transcript reads like an LLM answering questions, it
   has failed. It needs to read like that person talking.
2. **Anti-fabrication is non-negotiable.** Real persona, no surfaced source → vague-out, never
   fabricate. Fictional persona, no level-up mid-interview.
3. **Kickoff is mandatory.** Every run starts with `AskUserQuestion` to align — do not start
   research before clarifying.
4. **Do not sprinkle disfluencies uniformly.** Fillers cluster at thought transitions,
   hesitation moments, and sensitive/complex questions — one filler per sentence is caricature,
   not realism.
5. **At least 1–2 cognitive struggles per transcript.** Getting stuck, asking back, going off
   topic, admitting not knowing — these are what separate "real person" from "LLM mode".

## Workflow

```
Kickoff (AskUserQuestion, mandatory)
  ↓
Research
  - Real persona  → parallel WebSearch + WebFetch → dossier (verified/inferred)
  - Fictional     → representative-cohort search → synthesized dossier
  ↓
Duration calibration (rate × interviewee share = word budget)
  ↓
Per-question allocation (by protocol weight)
  ↓
Per-question generation (disfluency / cognitive patterns / probes / corrections)
  ↓
Coherence pass (persona consistency + cross-answer references)
  ↓
8-point self-check
  ↓
Output (file + full chat print)
```

## Speaking-rate baselines (used for word budget)

| Language | Unit | Rate per minute |
|---|---|---|
| English | words | 140–160 |
| Mandarin Chinese | characters | 200–260 |
| Japanese | mora / characters | 280–320 |
| Spanish | words | 160–180 |
| Korean | syllables | 220–260 |

**Budget formula**:

```
interviewee_budget = duration_min × rate × interviewee_share
```

`interviewee_share` ≈ 0.65–0.75 for UX research / expert interviews (interviewee-driven), 0.55–
0.65 for job-interview simulations (interviewer takes more time).

## Relationship to other interview tools

- This skill **generates** transcripts (synthesizes / simulates). It is not a real-audio
  transcription tool.
- It is not a replacement for real user research — if you can do a real interview, do a real
  interview.
- Good fits: pilot-protocol debugging, teaching examples, research-design dry-runs, persona
  exploration, creative writing (screenplay / podcast drafts), job-interview practice.

## Acknowledgments

Design philosophy draws on qualitative-interviewing methodology — half-structured-probe design,
avoidance of leading questions, capturing the stated-vs-actual behavior gap — from sources like
Steinar Kvale's _InterViews_, Robin Legard's in-depth interview methodology, and Steve
Portigal's _Interviewing Users_. The disfluency toolkit references common practices from
conversation analysis and spoken-corpus annotation.

## License

[MIT](LICENSE)
