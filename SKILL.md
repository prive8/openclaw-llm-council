---
name: llm-council
description: Multi-model AI decision council for OpenClaw. Five advisors with peer review and chairman synthesis. Based on Karpathy's LLM Council.
homepage: https://github.com/prive8/openclaw-llm-council
metadata:
  clawdbot:
    emoji: "🏛️"
    requires:
      env: []
      files: []
---

# LLM Council

Run decisions, ideas, and tradeoffs through a council of AI advisors who analyze, pressure-test, and synthesize a verdict. Based on Karpathy's LLM Council methodology.

---

## Modes Overview

| Mode | Advisors | Peer Review | Word Target | Token Est. |
|------|----------|-------------|-------------|------------|
| **Lite** | 3 (Contrarian, First Principles, Executor) | No | ~100 each | ~2,000 out |
| **Full** | 5 | Yes | ~150 each | ~6,000 out |
| **Deep** | 5 | Yes | ~200 each + full review | ~9,000 out |

Use **Lite** for fast pressure tests. Use **Full** for real decisions with tradeoffs. Use **Deep** for high-stakes bets you'll revisit.

---

## Triggers

### Lite Mode (fast, no peer review)
- "quick council this"
- "lite council"
- "brief this"
- "council lite"
- Any casual pressure-test with no explicit tier

### Full Mode (default for real decisions)
- "council this"
- "run the council"
- "war room this"
- "pressure-test this"
- "stress-test this"
- "debate this"
- "should I X or Y"
- "which option"
- "what would you do"
- "is this the right move"
- "validate this"
- "get multiple perspectives"
- "I can't decide"
- "I'm torn between"

### Deep Mode (high-stakes, full record)
- "deep council this"
- "full council"
- "council it deep"
- Any high-stakes decision with explicit request for thorough analysis

**Do NOT trigger on:** simple yes/no questions, factual lookups, casual questions without meaningful tradeoffs.

---

## Model Selection

Each advisor can run on a different model. Configure in `~/.openclaw/llm-council-config.json`:

```json
{
  "advisor_models": {
    "contrarian": "minimax/MiniMax-M2.7",
    "first-principles": "minimax/MiniMax-M2.7",
    "expansionist": "minimax/MiniMax-M2.7",
    "outsider": "minimax/MiniMax-M2.7",
    "executor": "minimax/MiniMax-M2.7",
    "chairman": "minimax/MiniMax-M2.7"
  },
  "default_model": "minimax/MiniMax-M2.7"
}
```

If the config file is missing or a model entry is absent, the skill falls back to the default model for that advisor. Model IDs must match your OpenClaw provider's model catalogue (e.g. `minimax/MiniMax-M2.7`, `openai/gpt-4.1`, `anthropic/claude-sonnet-4.5`).

---

## The Five Advisors

### 1. The Contrarian
Looks for what's wrong, what's missing, what will fail. Assumes there's a fatal flaw and tries to find it. Not a pessimist -- the friend who asks the question you're avoiding.

### 2. The First Principles Thinker
Asks "what are we actually trying to solve?" Strips assumptions, rebuilds from the ground. May say you're asking the wrong question entirely -- and that's the most valuable output.

### 3. The Expansionist
Looks for upside everyone else misses. What could be bigger? What adjacent opportunity is hiding? Doesn't care about risk -- that's the Contrarian's job.

### 4. The Outsider
Has zero context. Responds purely to what's in front of them. Catches the curse of knowledge: things obvious to you but confusing to everyone else. The most underrated advisor in the set.

### 5. The Executor
Only cares about "can this be done and what's the first step Monday morning?" Ignores theory and strategy. If an idea has no clear first step, says so.

**Why these five:** Contrarian vs Expansionist (downside vs upside). First Principles vs Executor (rethink vs do). Outsider keeps everyone honest with fresh eyes.

**Lite mode uses only:** Contrarian, First Principles, Executor.

---

## Step-by-Step Flow

### Step 1: Load Config

Read `~/.openclaw/llm-council-config.json` if present. Map each advisor to its configured model. Fall back to `default_model` for any missing entries.

### Step 2: Duplicate Check

Before doing anything, check the vault for recent councils on the same question:

```
Check: /path/to/vault/Memo/council-index.md
Check recent: /path/to/vault/Memo/Agent-OpenClaw/daily/  (last 7 days)
```

If the same or near-identical question was counciled recently, surface the previous transcript and ask if you want to rerun or revisit the existing verdict.

### Step 3: Context Scan

Quick scan of relevant workspace/vault files before framing:
- `SOUL.md`, `USER.md` in workspace if not already injected
- Any files explicitly referenced or attached
- Any relevant project/state files in the vault

If context is thin, ask one clarifying question before proceeding.

### Step 4: Frame the Question

Take the raw question + gathered context and reframe as a clear, neutral prompt all advisors receive. Include:
1. The core decision or question
2. Key context from the message
3. What's at stake (why this decision matters)
4. Any relevant constraints or timeline

Don't add your own opinion. Don't steer. Make it specific enough that advisors give grounded advice, not generic.

### Step 5: Convene Advisors (parallel)

Spawn all advisors simultaneously per the mode, each using its configured model from Step 1:

**Lite:** Contrarian, First Principles, Executor -- no peer review, skip to chairman
**Full/Deep:** All 5 advisors in parallel

Each gets their identity, the framed question, and clear instruction to respond independently without hedging. Lean fully into assigned perspective.

Word targets by mode:
- Lite: ~100 words per advisor
- Full: ~150 words per advisor
- Deep: ~200 words per advisor

### Step 6: Peer Review (Full and Deep only)

Collect all 5 responses. Anonymize as Response A-E (randomize mapping to prevent positional bias).

Spawn 5 reviewer agents -- one per original advisor role, each using their configured model. Each reviewer reads all 5 anonymized responses and answers:
1. Which response is strongest and why?
2. Which response has the biggest blind spot?
3. What did ALL responses miss?

### Step 7: Chairman Synthesis

Spawn the chairman agent using its configured model. Reads all original responses (+ peer reviews for Full/Deep) and produces:
- Where advisors agree
- Where they clash
- What you should actually do
- One concrete next action

**The chairman can disagree with the majority.** If the minority reasoning is strongest, say so and explain why.

### Step 8: Save Transcript to Vault

Save to: `/path/to/vault/Memo/council-[timestamp].md`

Filename format: `council-YYYY-MM-DD-[slug].md`

Transcript includes:
- Original question
- Framed question
- Mode used
- Models used per advisor
- All advisor responses (identified by role)
- Peer reviews (Full/Deep only)
- Chairman synthesis + verdict

### Step 9: Update Council Index (Deep only)

After a Deep council, append to `/path/to/vault/Memo/council-index.md`:
```
## YYYY-MM-DD: [one-line question summary]
Mode: deep | Verdict: [one line]
File: council-YYYY-MM-DD-[slug].md
```

Lite and Full sessions don't need index entries. Deep sessions are the ones worth indexing for cross-reference.

### Step 10: HTML Report (Deep only)

Generate a clean, scannable HTML report for Deep mode only. Save to workspace as `council-report-[timestamp].html`.

Report contains:
1. The question at top
2. Chairman's verdict (prominent)
3. One-line advisor positions summary
4. Collapsible full advisor responses
5. Collapsible peer review highlights (Full/Deep)
6. Footer with timestamp + models used

For Lite and Full: skip the HTML. The vault transcript is the artifact.

---

## Output Summary by Mode

| Mode | Files Saved | Report |
|------|-------------|--------|
| Lite | Vault transcript only | No |
| Full | Vault transcript only | No |
| Deep | Vault transcript + HTML report | Yes |

---

## Example Runs

### Lite Council

**User:** "lite council: should I take the consulting meeting with the university or focus on the automation project?"

*Spawns 3 advisors in parallel (Contrarian, First Principles, Executor). No peer review. Chairman synthesizes. Saves transcript to vault.*

**Chairman verdict:** The consulting meeting is likely a rabbit hole. Unless there's a specific deliverable, it's a time trap.

### Full Council

**User:** "run the council: I'm deciding between a $997 course or a $297 workshop as the anchor offer."

*Spawns 5 advisors in parallel. Peer review. Chairman synthesis. Saves transcript to vault.*

### Deep Council

**User:** "deep council this: I'm thinking about leaving my job and going full-time on my consulting business. I have 6 months runway."

*Full 5+5+1 run. Vault transcript saved. HTML report generated. Council index updated.*

---

## External Endpoints

This skill does not call any external APIs or endpoints. All council deliberation happens via OpenClaw's built-in `sessions_spawn` primitive, which creates isolated sub-agent sessions using the configured local model provider.

- **No external data transmission.** This skill operates entirely within the OpenClaw agent runtime.
- **No user data leaves the machine.** All processing is local via sub-agent spawning.

---

## Security & Privacy

- **Data scope:** This skill processes only the user's question and attached context. No personal data is collected, stored externally, or transmitted outside the local OpenClaw runtime.
- **Sub-agent isolation:** Each advisor runs as a separate `sessions_spawn` sub-agent. Sub-agents have no filesystem access, no network access, and no ability to make external calls beyond their LLM provider response.
- **No credential storage:** This skill does not store or transmit credentials. Model configuration uses only local file paths and model identifiers.
- **No shell execution:** This skill does not execute shell commands, run scripts, or modify the filesystem beyond writing transcript files to the designated vault path.
- **User input handling:** The skill accepts free-text questions from the user and passes them to advisor prompts. Sanitize the vault path in Steps 8 and 9 before use.

---

## Trust Statement

By using this skill, data stays entirely within your OpenClaw agent runtime. No information is sent to third parties. The skill's behavior is deterministic -- it runs council deliberations based on your questions and saves transcripts to your local vault. Only install if you trust your OpenClaw configuration and local model provider.

---

## Model Invocation Note

This skill invokes models autonomously during council sessions. Each council spawns 4-11 sub-agents depending on mode (Lite=4, Full/Deep=11). Each sub-agent call uses the configured model provider. If you want to limit model usage, use the `default_model` in `llm-council-config.json` to route all advisors through a single provider, or disable council triggers if autonomous invocation is not desired.

---

## Important Notes

- **Always spawn advisors in parallel.** Sequential spawning lets earlier responses bleed in.
- **Anonymize for peer review.** Position bias ruins the review quality.
- **Model selection is per-advisor.** Different models = genuinely different perspectives. Using the same model for all 5 advisors defeats the purpose.
- **Lite mode is the default for ambiguous triggers.** Default to lite unless stakes are clearly high.
- **Duplicate detection saves time and token waste.** Confirm new information or changed circumstances before re-counciling.
- **One next action is mandatory from the chairman.** Not "here are some options" -- one thing to do next, in one sentence.

---

## File Structure

```
llm-council/
├── SKILL.md                          # This file
├── llm-council-config.example.json   # Model configuration example
└── README.md                         # Setup guide and usage
```

---

## Changelog

### 1.1.0
- Added configurable per-advisor model selection via `llm-council-config.json`
- Added model usage tracking in transcripts and HTML reports
- Added External Endpoints, Security & Privacy, Trust Statement sections
- Added Model Invocation Note section
- Three-tier mode system (Lite/Full/Deep)

### 1.0.0
- Initial implementation