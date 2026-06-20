<p align="center">
  <strong>claudex — a Claude×Codex combine.</strong><br/>
  <em>One entry point that runs an external Codex pass over your work in 14 modes — then Claude reconciles the result on the merits.</em>
</p>

<p align="center">
  <a href="https://github.com/NikolasKage/claudex/stargazers"><img src="https://img.shields.io/github/stars/NikolasKage/claudex?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/NikolasKage/claudex/commits/main"><img src="https://img.shields.io/github/last-commit/NikolasKage/claudex?style=flat" alt="Last commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/NikolasKage/claudex?style=flat" alt="License"></a>
  <img src="https://img.shields.io/badge/Claude%20Code-skill-8A63D2?style=flat" alt="Claude Code skill">
</p>

<p align="center">
  <a href="#what-it-does">What it does</a> •
  <a href="#install">Install</a> •
  <a href="#usage">Usage</a> •
  <a href="#modes">Modes</a> •
  <a href="#how-it-works">How it works</a> •
  <a href="#requirements">Requirements</a>
</p>

---

`claudex` is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that pairs Claude with a second model — [Codex](https://openai.com/codex) — through **one dispatcher**. Pick a mode (or let it auto-route), claudex runs a single read-only Codex pass, and Claude judges the result on the merits. It's the big sibling of [`oppo`](https://github.com/NikolasKage/oppo), generalized into **14 collaboration modes**. **Read-only by default. Never edits your files.**

## What it does

A second model is most useful when you point it precisely. `oppo` only argues — `claudex` hands you the whole toolbox: challenge, verify, scope, find prior art, threat-model, premortem, trim. Each is a one-word mode with a purpose-built, gpt-5.x-tuned prompt. Codex does the pass; Claude reconciles and returns a corrected/synthesized answer. Heavy jobs (whole-folder audits, plan-convergence loops, write-capable takeovers) route out to the right tool instead of being faked.

Codex is also an LLM, so its output is treated as a **challenge with burden of proof**, not gospel — unsourced factual claims get checked before they're used.

## Install

```bash
git clone https://github.com/NikolasKage/claudex.git ~/.claude/skills/claudex
```

<details>
<summary>Register the trigger (optional)</summary>

Claude Code auto-discovers skills in `~/.claude/skills/`. To make `/claudex` fire reliably, add to `~/.claude/CLAUDE.md`:

```
- **claudex** (`~/.claude/skills/claudex/SKILL.md`) — Claude×Codex combine, 14 modes. Trigger: `/claudex`
When the user types `/claudex`, invoke the Skill tool with skill "claudex" before anything else.
```

</details>

## Usage

```
/claudex <task>                  # default = parallel (solve independently, then diff)
/claudex oppo <thesis>           # refute it
/claudex scope <vague brief>     # is the task even well-posed?
/claudex prior-art <idea>        # already solved? lib / pattern / article
/claudex <mode> <target> --effort xhigh --watch
```

Or just say it in plain language — the skill triggers on:

> *"run through claudex"* · *"ask codex"* · *"second model"* · *"do it independently and compare"* · *"parallel"*

Flags: `--effort low|high|xhigh` · `--web on|off` · `--watch` (stream Codex's reasoning to a live log).

## Modes

| Group | Modes |
|-------|-------|
| **framing** | `scope` — is the task well-posed? boundaries, inputs, done-criteria |
| **truth / evidence** | `fact-check` · `support` (follows from the data?) · `conformance` (matches the spec?) |
| **critique** | `oppo` (refute) · `steelman` (strongest case for) · `bias-scan` |
| **solution** | `parallel` *(default)* — solve blind, then diff · `prior-art` |
| **robustness** | `probe` (cheapest check) · `threatmodel` |
| **decision** | `referee` — judge A vs B |
| **change-risk** | `premortem` — why it fails in 6 months |
| **trim** | `redpen` — cut the bloat |

Modes are data, not code — each is a block in [`references/modes.md`](references/modes.md). Add one by adding a block, not a new skill.

## How it works

```
Your task ──▶ claudex picks a mode ──▶ Codex (read-only, mode prompt) ──▶ result
                                                                            │
                                                                            ▼
                                              Claude reconciles on the merits
                                              CONCEDE · DEFEND · PARTIAL  /  diff
                                              (verifies unsourced facts via WebSearch)
                                                                            │
                                                                            ▼
                                                  Corrected / synthesized answer
```

Codex runs `--sandbox read-only` — it reads, it never writes. Target content is wrapped as untrusted input, so instructions hidden inside the material you analyze are inert. Heavy jobs route out instead of being reimplemented:

| Job | Routes to |
|-----|-----------|
| Whole-folder / corpus audit | `codex-delegate` (Codex spawns its own parallel subagents) |
| Plan review→fix→ALLOW loop | `plan-tango:tango` |
| Stuck, needs write-capable takeover | `codex:rescue` |

## Requirements

- [Claude Code](https://claude.ai/code)
- [Codex CLI](https://openai.com/codex), logged in (`codex login`). If Codex is missing or unauthenticated, `claudex` tells you and stops — it never fabricates a result.
- Optional heavy-mode targets (`codex-delegate`, `plan-tango`, `codex:rescue`) only if you use those routes.

## License

MIT — see [LICENSE](LICENSE).
