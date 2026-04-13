# Council Review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blueviolet?logo=anthropic)](https://claude.ai/code)
[![GitHub Stars](https://img.shields.io/github/stars/ngmeyer/council-review?style=social)](https://github.com/ngmeyer/council-review)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey)]()

A Claude Code skill that runs decisions, code, plans, and PRs through a council of 5 independent AI advisors who argue, peer-review each other anonymously, and synthesize a verdict you can trust.

Based on [Andrej Karpathy's LLM Council](https://x.com/karpathy) methodology, adapted from [Ole Lehmann's](https://x.com/itsolelehmann) implementation.

## How It Works

```
You → Question/Code/Plan
         ↓
   5 Advisors (parallel, independent)
         ↓
   5 Peer Reviews (anonymous, cross-review)
         ↓
   Chairman Synthesis (final verdict)
         ↓
   Report + Recommendation
```

**The 5 Advisors:**

| Advisor | Angle | Catches |
|---------|-------|---------|
| The Contrarian | What will fail? Assumes a fatal flaw exists | "Sounds great but..." gaps |
| First Principles | What are we actually solving? | "You're optimizing the wrong variable" |
| The Expansionist | What upside are we missing? | "You're thinking too small" |
| The Outsider | Zero context, fresh eyes only | Curse of knowledge blind spots |
| The Executor | What do you do Monday morning? | Brilliant plans with no first step |

**Natural tensions:** Contrarian vs Expansionist (downside vs upside), First Principles vs Executor (rethink vs ship it), Outsider keeps everyone honest.

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
/council-review Review the implementation plan in docs/plans/v2-migration.md
/council-review https://github.com/org/repo/pull/42
```

## What It Reviews

| Input | What Happens |
|-------|-------------|
| **A question or decision** | 5 advisors independently analyze tradeoffs, peer-review each other, synthesize a recommendation |
| **An implementation plan** | Advisors stress-test feasibility, scope, risks, missing pieces, and execution order |
| **A PR or code change** | Advisors review from security, architecture, performance, usability, and pragmatism angles |
| **A file path** | Reads the file and councils its contents |

## Why This Works

1. **Parallel independence** — Advisors don't see each other's responses. No groupthink.
2. **Anonymous peer review** — Responses are shuffled as A-E before review. No deference to roles.
3. **Forced tension** — The Contrarian *must* find flaws. The Expansionist *must* find upside. Balanced coverage is structural, not aspirational.
4. **Chairman override** — The synthesizer can side with a minority if their reasoning is strongest. Quality of argument beats vote counting.
5. **"What did ALL five miss?"** — The most valuable question in the peer review. Surfaces blind spots no single perspective catches.

## Configuration

The skill works with zero configuration. To customize:

- **Model selection**: Advisors use lightweight models by default. The chairman uses the best available model.
- **Review scope**: Automatically detects whether input is a question, plan, PR, or file path.
- **Output**: Generates a markdown report with full transcript.

## Output Format

```
## Council Verdict: [Topic]

### Where the Council Agrees
[High-confidence signals — multiple advisors converged independently]

### Where the Council Clashes  
[Genuine disagreements with both sides presented]

### Blind Spots Revealed
[Things only the peer review caught]

### Recommendation
[Clear, actionable. Not "it depends." A real answer.]

### Do This First
[Single concrete next step. Not a list. One thing.]
```

## Credits

- Original concept: **Andrej Karpathy** (LLM Council)
- Claude Code adaptation: **Ole Lehmann** ([@itsolelehmann](https://x.com/itsolelehmann))
- Skill packaging: **Neal Meyer**

## License

MIT
