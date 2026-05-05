# LLM Council

A multi-model AI decision council for OpenClaw agents. Five specialized advisors independently analyze a question, peer-review each other's responses anonymously, and a chairman synthesizes a final verdict.

Based on [Karpathy's LLM Council](https://github.com/karpathy/llm-council) methodology, adapted as an OpenClaw agent skill.

---

## Overview

Instead of asking one AI and accepting whatever it says, the council runs your question through **5 distinct advisors**, each with a different thinking style and priority:

| Advisor | Focus |
|---------|-------|
| **Contrarian** | What's wrong, missing, or about to fail |
| **First Principles** | Are we asking the right question? |
| **Expansionist** | What's the upside everyone's missing? |
| **Outsider** | Fresh eyes -- whatconfuses people with no context? |
| **Executor** | Can it be done? What's the first step Monday morning? |

The advisors work in parallel, then anonymously review each other, then a chairman produces the final synthesis.

---

## Modes

| Mode | Advisors | Peer Review | Word Target | Token Est. |
|------|----------|-------------|-------------|------------|
| **Lite** | 3 (C, FP, Ex) | No | ~100 each | ~2,000 |
| **Full** | 5 | Yes | ~150 each | ~6,000 |
| **Deep** | 5 | Yes | ~200 each | ~9,000 |

- **Lite**: Fast pressure test. No peer review. Good for quick sanity checks.
- **Full**: Standard council run. Peer review enabled. For real decisions.
- **Deep**: High-stakes decisions. Full output + HTML report + vault index update.

---

## Setup

### 1. Install the skill

If using ClawHub:

```bash
clawhub install llm-council
```

Or place `SKILL.md` in your skills directory (`~/.openclaw/skills/llm-council/`).

### 2. Configure models (optional)

Create `~/.openclaw/llm-council-config.json` to assign different models per advisor:

```json
{
  "advisor_models": {
    "contrarian": "minimax/MiniMax-M2.7",
    "first-principles": "openai/gpt-4.1",
    "expansionist": "minimax/MiniMax-M2.7",
    "outsider": "anthropic/claude-sonnet-4.5",
    "executor": "minimax/MiniMax-M2.7",
    "chairman": "minimax/MiniMax-M2.7"
  },
  "default_model": "minimax/MiniMax-M2.7"
}
```

Each advisor runs as a separate sub-agent, so using different models gives genuinely different analytical perspectives. If a config is missing, the skill falls back to `default_model`.

Model IDs must match your OpenClaw provider's catalogue.

### 3. Vault path (for transcript storage)

The skill saves transcripts to a vault path. Update the path in `SKILL.md` Step 8 to match your Obsidian vault location, or replace with any directory you want council transcripts saved to.

---

## Usage

### Trigger phrases

**Lite mode:**
- "quick council this"
- "lite council"
- "brief this"
- "council lite"

**Full mode (default for real decisions):**
- "council this"
- "run the council"
- "war room this"
- "pressure-test this"
- "stress-test this"
- "should I X or Y"
- "which option"
- "what would you do"
- "validate this"
- "I can't decide"

**Deep mode:**
- "deep council this"
- "full council"

### Example sessions

**Lite:**
> "lite council: should I take that meeting or focus on shipping the feature?"

**Full:**
> "council this: I'm choosing between a $997 course or a $297 workshop as the anchor offer for my audience."

**Deep:**
> "deep council this: I'm thinking about going full-time on my consulting business. I have 6 months runway."

### What you get

- **Lite/Full**: Transcript saved to vault at `Memo/council-YYYY-MM-DD-[slug].md`
- **Deep**: Transcript saved to vault + HTML report saved to workspace + council index updated

The chairman always delivers:
- Where advisors agree
- Where they clash
- What you should actually do
- One concrete next action

---

## Architecture

```
User Question
     │
     ▼
[1] Duplicate Check  ──▶ If recent council exists, ask to revisit
     │
     ▼
[2] Context Scan      ──▶ Gather relevant workspace/vault files
     │
     ▼
[3] Frame Question    ──▶ Neutral prompt all advisors receive
     │
     ▼
[4] Spawn Advisors    ──▶ Parallel sub-agents (3 for Lite, 5 for Full/Deep)
     │                   Each using its configured model
     ▼
[5] Peer Review       ──▶ (Full/Deep only) Anonymized cross-review
     │                   Each advisor reviews all 5 responses
     ▼
[6] Chairman Synthesis──▶ Reads everything, produces final verdict
     │
     ▼
[7] Save Transcript   ──▶ Vault at Memo/council-YYYY-MM-DD-[slug].md
     │
     ▼
[8] Index + Report    ──▶ (Deep only) Update council-index.md + HTML report
```

---

## File Structure

```
llm-council/
├── SKILL.md                          # Main skill file
├── llm-council-config.example.json    # Model configuration example
└── README.md                         # This file
```

---

## Token Cost Estimation

A full Deep council (5 advisors + 5 peer reviews + chairman synthesis):
- 5 advisors × ~200 words = ~1,000 words output
- 5 peer reviews × (reading 5 responses + ~100 word review) = ~3,000 words input + ~500 output
- Chairman synthesis: ~300 words

Total output: ~1,800 words × model pricing. Input context is larger due to all responses being fed to each reviewer and chairman.

Lite mode is ~4 spawns and ~400 words output. Full is ~11 spawns and ~1,500 words output.

---

## Adapting for Other Users

The vault path is hardcoded in SKILL.md Steps 7 and 8. Before publishing or sharing, update it to your vault path. The skill is path-agnostic otherwise -- only the save location needs customization per installation.

Model configuration is per-user via `~/.openclaw/llm-council-config.json`. Share the example config with new users as a starting point.

---

## Changelog

### 1.1.0
- Added configurable per-advisor model selection via `llm-council-config.json`
- Added model usage tracking in transcripts
- Added model info to HTML report footer (Deep mode)

### 1.0.0
- Three-tier mode system (Lite/Full/Deep)
- Vault-based transcript storage with duplicate detection
- Council index for Deep mode cross-reference
- HTML report generation for Deep mode
- Original implementation