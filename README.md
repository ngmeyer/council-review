# Council Review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blueviolet?logo=anthropic)](https://claude.ai/code)
[![GitHub Stars](https://img.shields.io/github/stars/ngmeyer/council-review?style=social)](https://github.com/ngmeyer/council-review)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey)]()

A Claude Code skill that runs decisions, code, plans, and PRs through a council of 5 independent AI advisors who argue, peer-review each other anonymously, and synthesize a verdict you can trust.

Based on [Andrej Karpathy's LLM Council](https://x.com/karpathy) methodology, adapted from [Ole Lehmann's](https://x.com/itsolelehmann) implementation. Reasoning method diversity informed by [DMAD (ICLR 2025)](https://openreview.net/forum?id=t6QHYUOQL7).

## How It Works

```
You -> Question/Code/Plan
         |
   5 Advisors (parallel, independent, distinct reasoning methods)
         |
   5 Peer Reviews (anonymous, cross-review)
         |
   Chairman Synthesis (verdict + dissent preservation + verification)
         |
   Report + Recommendation + "What You Lose"
```

**The 5 Advisors:**

| Advisor | Angle | Reasoning Method |
|---------|-------|-----------------|
| The Contrarian | What will fail? | **Inversion** -- assume it failed, trace backward |
| First Principles | What are we actually solving? | **Decomposition** -- break into atomic claims, challenge each |
| The Expansionist | What upside are we missing? | **Analogy** -- what adjacent domain solved this differently? |
| The Outsider | Zero context, fresh eyes | **Naive questioning** -- flag anything requiring insider knowledge |
| The Executor | What do you do Monday morning? | **Dependency graphing** -- what blocks what? |

Each advisor uses a different *reasoning method*, not just a different angle. This is the key insight from [DMAD research](https://openreview.net/forum?id=t6QHYUOQL7): same-model councils need method diversity to avoid converging on the same reasoning patterns.

## Install

Copy the skill file into your Claude Code commands directory:

```bash
# Global (all projects)
cp council-review.md ~/.claude/commands/council-review.md

# Per-project
cp council-review.md .claude/commands/council-review.md
```

Then use it in Claude Code:

```
/council-review Should we rewrite our auth layer in Rust?
/council-review docs/plans/v2-migration.md
/council-review https://github.com/org/repo/pull/42
/council-review --quick Is this naming convention worth changing?
/council-review --adversarial Should we adopt microservices?
```

## Modes

| Mode | Flag | Calls | Best For |
|------|------|-------|----------|
| **Full** (default) | none | 11 | High-stakes decisions |
| **Quick** | `--quick` | 4 | Routine decisions, gut-checks |
| **Adversarial** | `--adversarial` | 11 | Proposals, yes/no decisions |

**Quick mode** runs 3 advisors (Contrarian, Executor, Outsider) + chairman. No peer review. Fast.

**Adversarial mode** assigns 2 advocates (argue FOR), 2 skeptics (argue AGAINST), and 1 neutral analyst. Forces genuine opposition instead of balanced perspectives.

## What It Reviews

| Input | What Happens |
|-------|-------------|
| **A question or decision** | 5 advisors independently analyze tradeoffs, peer-review each other, synthesize a recommendation |
| **An implementation plan** | Advisors stress-test feasibility, scope, risks, missing pieces, and execution order |
| **A PR or code change** | Advisors review from security, architecture, performance, usability, and pragmatism angles |
| **A file path** | Reads the file and councils its contents |

## Why This Works

1. **Method diversity** -- Each advisor uses a distinct reasoning method (inversion, decomposition, analogy, naive questioning, dependency graphing). Same-model councils converge without this.
2. **Parallel independence** -- Advisors don't see each other's responses. No groupthink.
3. **Anonymous peer review** -- Responses are shuffled as A-E before review. No deference to roles.
4. **Forced tension** -- The Contrarian *must* find flaws. The Expansionist *must* find upside. Coverage is structural.
5. **Disagreement classification** -- Chairman distinguishes "value tensions" (both sides valid) from "error catches" (one advisor found a real flaw).
6. **Dissent preservation** -- "What You Lose" section explicitly names the cost of ignoring the minority view.
7. **Chairman override** -- The synthesizer can side with a minority if their reasoning is strongest.
8. **"What did ALL five miss?"** -- The most valuable question in the peer review.

## Output Format

```
## Council Verdict: [Topic]

### Where the Council Agrees
[High-confidence signals -- multiple advisors converged independently]

### Where the Council Clashes
[Value Tension] Both sides valid, depends on priorities...
[Error Catch] One advisor found a real flaw others missed...

### Blind Spots Revealed
[Things only the peer review caught]

### Recommendation
[Clear, actionable. Not "it depends." A real answer.]

### What You Lose
[Cost of following this recommendation. The strongest dissent, preserved.]

### Do This First
[One concrete next step.]
How to verify: [2-3 checks to confirm the recommendation was right]
```

## Auto-Context

The skill automatically reads project files before framing the question:
- `README.md`, `CLAUDE.md` / `AGENTS.md`
- Recent git log
- PR diff (if reviewing a PR)
- Referenced files

No manual context-pasting needed. Advisors see your project, not a blank slate.

## Credits

- Original concept: **Andrej Karpathy** ([LLM Council](https://github.com/karpathy/llm-council))
- Claude Code adaptation: **Ole Lehmann** ([@itsolelehmann](https://x.com/itsolelehmann))
- Reasoning method diversity: **DMAD** ([ICLR 2025](https://openreview.net/forum?id=t6QHYUOQL7))
- Skill by: **Neal Meyer**

## License

MIT
