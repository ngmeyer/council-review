---
name: council-review
description: "Run any question, plan, PR, or code through a council of 5 AI advisors who independently analyze it, peer-review each other anonymously, and synthesize a verdict. Use when: 'council this', 'run the council', 'council review', 'pressure-test this', 'stress-test this', 'war room this', or when facing a genuine decision with stakes and tradeoffs."
argument-hint: "[question, file path, PR number, GitHub URL, or plan description]"
---

# Council Review

Run any question, plan, or code through 5 independent advisors who argue, peer-review each other anonymously, and synthesize a verdict you can trust.

## When to Use

The council is for questions where **being wrong is expensive**.

**Good for:** Architecture decisions, implementation plans, PR reviews, product decisions, migration strategies, API design, naming, pricing, scope decisions
**Bad for:** Factual lookups, writing tasks, simple yes/no, anything with one obvious right answer

## The Five Advisors

| # | Advisor | Angle | Catches |
|---|---------|-------|---------|
| 1 | **The Contrarian** | What will fail? Assumes a fatal flaw exists | "Sounds great but..." gaps you skip when excited |
| 2 | **First Principles Thinker** | What are we actually solving? Strips every assumption | "You're optimizing the wrong variable" |
| 3 | **The Expansionist** | What upside are we missing? What could be bigger? | "You're thinking too small" |
| 4 | **The Outsider** | Zero context, fresh eyes only | Curse of knowledge blind spots |
| 5 | **The Executor** | What do you do Monday morning? Logistics and sequencing | Brilliant plans with no actionable first step |

**Natural tensions:** Contrarian vs Expansionist (downside vs upside), First Principles vs Executor (rethink vs ship it), Outsider keeps everyone honest.

---

## Execution Flow

### Step 0: Detect Input Type

<input_detection>

Look at `$ARGUMENTS` and classify:

1. **PR** — Numeric value, or URL containing `/pull/`. Fetch PR diff and description via `gh pr view`.
2. **File path** — String ending in a file extension or pointing to an existing file. Read the file contents.
3. **Plan/Decision/Question** — Everything else. Use as-is.

For PRs and files, read the actual content and include it in the framed question. Don't just pass a URL — advisors need the substance.

</input_detection>

### Step 1: Frame the Question

Before framing, scan for relevant context:
- Project README, CLAUDE.md, or architecture docs if they exist
- Any files the user referenced or that relate to the topic
- Git log or PR description if reviewing code

Reframe the raw input as a clear, neutral prompt:

```
QUESTION:
[Core decision, plan, or code being reviewed]

CONTEXT:
[Key context: what the project does, constraints, stakes, what prompted this]

WHAT'S AT STAKE:
[Why this matters — cost of getting it wrong]
```

Don't add your own opinion. Don't steer toward an answer. If too vague, ask ONE clarifying question before proceeding.

### Step 2: Convene the Council (5 agents in parallel)

Launch all 5 advisors **simultaneously** using the Agent tool. Each advisor runs in parallel. Use a lightweight model (`haiku`) for advisors — they're doing focused analysis, not complex reasoning.

**CRITICAL: Launch all 5 in a single message with 5 Agent tool calls.** Sequential execution lets earlier responses bleed into later ones and defeats the purpose.

Each advisor gets this prompt:

```
You are [ADVISOR NAME] on an LLM Council reviewing a decision.

Your thinking style: [ADVISOR DESCRIPTION — see table above]

A user has brought this to the council:
---
[framed question from Step 1]
---

Respond from YOUR perspective only. Be direct and specific. Don't hedge or try to be balanced — the other advisors cover your gaps. Lean fully into your assigned angle.

Rules:
- 150-300 words. No preamble. Straight into your analysis.
- Name specific risks, opportunities, or issues — not vague concerns.
- If reviewing code: cite specific files, functions, or patterns.
- If reviewing a plan: point to specific steps, gaps, or sequencing issues.
- End with your single strongest recommendation.
```

**Advisor-specific instructions:**

- **Contrarian:** "Assume there IS a fatal flaw. Find it. What looks safe but will break under pressure? What's the failure mode nobody's discussing?"
- **First Principles:** "Strip away every assumption. What problem are we actually solving? Is this the right abstraction level? Would you build it this way if starting from zero?"
- **Expansionist:** "What's the 10x version? What adjacent opportunity does this unlock? What would someone with unlimited ambition do differently? Where is this thinking too small?"
- **Outsider:** "You have zero context about this project. Based purely on what you see here, what's confusing? What smells wrong? What would you ask if you just joined the team?"
- **Executor:** "Forget whether this is a good idea. If we commit to this, what's the Monday morning action? What's the critical path? What blocks everything else? What takes 5 minutes but everyone will forget?"

### Step 3: Anonymous Peer Review (5 agents in parallel)

Collect all 5 advisor responses. **Randomize the mapping** — Advisor 1 should NOT always be Response A. Then launch 5 reviewer agents in parallel (use `haiku`).

Each reviewer sees all 5 anonymized responses:

```
You are reviewing the outputs of an LLM Council. Five advisors independently answered:

---
[framed question]
---

**Response A:** [randomized advisor response]
**Response B:** [randomized advisor response]
**Response C:** [randomized advisor response]
**Response D:** [randomized advisor response]
**Response E:** [randomized advisor response]

Answer these three questions. Be specific. Reference responses by letter.

1. Which response is strongest? Why? (one sentence)
2. Which has the biggest blind spot? What is it missing? (one sentence)
3. What did ALL five responses miss that the council should consider? (This is the most valuable question — think hard.)

Keep under 150 words. Be direct. No preamble.
```

### Step 4: Chairman Synthesis

One agent gets everything: the original question, all 5 advisor responses (de-anonymized with names), and all 5 peer reviews. Use the best available model for this (default — do not specify a lightweight model).

The chairman produces exactly this structure:

```
## Council Verdict: [Topic — 5 words max]

### Where the Council Agrees
[Points where multiple advisors converged independently — these are high-confidence signals]

### Where the Council Clashes
[Genuine disagreements. Present BOTH sides and why reasonable people disagree. Don't flatten the tension.]

### Blind Spots Revealed
[Things only the peer review caught — the "what did ALL five miss?" answers]

### Recommendation
[Clear, actionable recommendation. Not "it depends." Not "consider both options." A real answer with reasoning. The chairman CAN disagree with the majority if the dissenter's reasoning is strongest.]

### Do This First
[Single concrete next step. Not a list. Not three options. One thing to do right now.]
```

### Step 5: Present Results

Show the chairman's verdict directly in chat. Then provide the full transcript in a collapsible section or separate file if the user wants it.

---

## Cost Budget

Total: **11 sub-agent calls**
- 5 advisors (lightweight model)
- 5 peer reviewers (lightweight model)
- 1 chairman (best model)

Fast models for volume, good model for synthesis. The chairman's reasoning quality is what matters most.

## Gotchas

- **Always parallel spawn advisors.** Sequential lets earlier responses contaminate later ones.
- **Always anonymize for peer review.** Reviewers defer to "The Contrarian" or "First Principles" if they see the label. Shuffle the letters.
- **Chairman can override majority.** Quality of reasoning > vote count. If the Contrarian found a real flaw that everyone else missed, the chairman should side with them.
- **Don't council trivial questions.** If one right answer exists, just answer it. The council is for genuine tradeoffs.
- **Context is critical.** Generic input = generic output. Read project files, PR diffs, plan documents before framing.

## Credits

Original concept: Andrej Karpathy (LLM Council)
Claude Code adaptation: Ole Lehmann (@itsolelehmann)
Skill by: Neal Meyer
