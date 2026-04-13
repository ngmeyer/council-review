---
name: council-review
description: "Run any question, plan, PR, or code through a council of 5 AI advisors who independently analyze it, peer-review each other anonymously, and synthesize a verdict. Use when: 'council this', 'run the council', 'council review', 'pressure-test this', 'stress-test this', 'war room this', or when facing a genuine decision with stakes and tradeoffs."
argument-hint: "[question, file path, PR number, or GitHub URL] [--quick] [--adversarial]"
---

# Council Review

Run any question, plan, or code through 5 independent advisors who argue, peer-review each other anonymously, and synthesize a verdict you can trust.

## When to Use

The council is for questions where **being wrong is expensive**.

**Good for:** Architecture decisions, implementation plans, PR reviews, product decisions, migration strategies, API design, naming, pricing, scope decisions
**Bad for:** Factual lookups, writing tasks, simple yes/no, anything with one obvious right answer

## Flags

| Flag | Effect |
|------|--------|
| `--quick` | Lite mode: 3 advisors + chairman, no peer review (4 calls instead of 11) |
| `--adversarial` | Adversarial mode: 2 advocates FOR + 2 skeptics AGAINST + 1 neutral |

## The Five Advisors

| # | Advisor | Angle | Reasoning Method | Catches |
|---|---------|-------|------------------|---------|
| 1 | **The Contrarian** | What will fail? | **Inversion** — assume it shipped and failed, trace backward to the cause | "Sounds great but..." gaps you skip when excited |
| 2 | **First Principles Thinker** | What are we actually solving? | **Decomposition** — break into atomic claims, challenge each one | "You're optimizing the wrong variable" |
| 3 | **The Expansionist** | What upside are we missing? | **Analogy** — what adjacent domain solved this differently? | "You're thinking too small" |
| 4 | **The Outsider** | Zero context, fresh eyes only | **Naive questioning** — explain like you just joined; flag anything that requires insider knowledge to make sense | Curse of knowledge blind spots |
| 5 | **The Executor** | What do you do Monday morning? | **Dependency graphing** — what blocks what? What's the critical path? | Brilliant plans with no actionable first step |

**Natural tensions:** Contrarian vs Expansionist (downside vs upside), First Principles vs Executor (rethink vs ship it), Outsider keeps everyone honest.

---

## Execution Flow

### Step 0: Pre-flight

<pre_flight>

**Parse flags from `$ARGUMENTS`:**
- If `--quick` is present: use Lite Mode (see below)
- If `--adversarial` is present: use Adversarial Mode (see below)
- Remove flags from the input before classifying

**Scope validation:** Before convening the council, assess whether the input actually warrants it. If the question is purely factual, has one obvious right answer, or has no meaningful tradeoff, say so directly: "This doesn't need a council — [direct answer]. Use `/council-review` for decisions with genuine stakes and tradeoffs." Do not spawn agents for trivial questions.

**Classify the remaining input:**

1. **PR** — Numeric value, or URL containing `/pull/`. Fetch PR diff and description via `gh pr view`.
2. **File path** — String ending in a file extension or pointing to an existing file. Read the file contents.
3. **Plan/Decision/Question** — Everything else. Use as-is.

For PRs and files, read the actual content and include it in the framed question. Don't just pass a URL — advisors need the substance.

</pre_flight>

### Step 1: Gather Context and Frame

**Auto-context gathering** — before framing, read these project files (skip any that don't exist):
- `README.md` — what the project does
- `CLAUDE.md` or `AGENTS.md` — conventions, architecture, patterns
- Recent git log (`git log --oneline -10`) — what's been happening
- Any files the user referenced or that relate to the topic
- PR diff and description if reviewing a PR

Reframe the raw input as a clear, neutral prompt:

```
QUESTION:
[Core decision, plan, or code being reviewed]

CONTEXT:
[Key context from project files: what the project does, constraints, recent changes, stakes]

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

Your angle: [ADVISOR ANGLE]
Your reasoning method: [ADVISOR REASONING METHOD — see table above]

A user has brought this to the council:
---
[framed question from Step 1]
---

Apply your assigned reasoning method rigorously. Don't just state opinions — show your work using your method.

Rules:
- 150-300 words. No preamble. Straight into your analysis.
- Name specific risks, opportunities, or issues — not vague concerns.
- If reviewing code: cite specific files, functions, or patterns.
- If reviewing a plan: point to specific steps, gaps, or sequencing issues.
- End with your single strongest recommendation.
```

**Advisor-specific instructions (include the reasoning method):**

- **Contrarian:** "Your method is INVERSION. Assume this shipped exactly as proposed — and failed. Work backward: what was the cause of failure? What looked safe but broke under pressure? What's the failure mode nobody is discussing? Show your inversion chain."
- **First Principles:** "Your method is DECOMPOSITION. Break this into its atomic claims and assumptions. List them. Challenge each one: is this actually true? Is it necessary? What would change if this assumption were wrong? Show which assumptions are load-bearing."
- **Expansionist:** "Your method is ANALOGY. What adjacent domain, product, or technology solved a similar problem differently? What would someone with 10x ambition do here? Where is this thinking too small? Name specific analogues and what they'd suggest."
- **Outsider:** "Your method is NAIVE QUESTIONING. You have zero context about this project. Based purely on what you see here, list every point that requires insider knowledge to understand. What's confusing? What jargon is unexplained? What would you ask if you just joined the team? If you can't follow the reasoning, say so."
- **Executor:** "Your method is DEPENDENCY GRAPHING. Map the dependencies: what blocks what? What's the critical path? What's the first thing that must happen, and what can't start until it finishes? What takes 5 minutes but everyone will forget? Show the execution sequence."

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

One agent gets everything: the original question, all 5 advisor responses (de-anonymized with names and reasoning methods), and all 5 peer reviews. Use the best available model for this (default — do not specify a lightweight model).

The chairman produces exactly this structure:

```
## Council Verdict: [Topic — 5 words max]

### Where the Council Agrees
[Points where multiple advisors converged independently — these are high-confidence signals]

### Where the Council Clashes

For each disagreement, classify it:

**[Value Tension]** — Both sides are valid; the right choice depends on priorities.
[Present both sides clearly. Name the tradeoff.]

**[Error Catch]** — One advisor found a real flaw the others missed.
[Name the flaw, who caught it, and why it matters.]

### Blind Spots Revealed
[Things only the peer review caught — the "what did ALL five miss?" answers]

### Recommendation
[Clear, actionable recommendation. Not "it depends." Not "consider both options." A real answer with reasoning. The chairman CAN disagree with the majority if the dissenter's reasoning is strongest.]

### What You Lose
[If you follow this recommendation, what does the strongest dissenting voice say you're giving up? Name the specific risk or missed opportunity. This is NOT a hedge — it's informed consent.]

### Do This First
[Single concrete next step. Not a list. Not three options. One thing to do right now.]

**How to verify:** [2-3 concrete checks to confirm the recommendation was right. What should you measure? What should you look for after N days/weeks?]
```

### Step 5: Present Results

Show the chairman's verdict directly in chat. Then provide the full transcript in a collapsible section or separate file if the user wants it.

---

## Lite Mode (`--quick`)

When `--quick` is passed, run a streamlined council:

1. **3 advisors only:** Contrarian, Executor, Outsider (the three most action-oriented perspectives)
2. **No peer review** — skip Step 3 entirely
3. **Chairman synthesis** from 3 responses instead of 5
4. **Same output format** but faster (4 agent calls instead of 11)

Use for routine decisions, quick gut-checks, or when time matters more than exhaustive coverage.

## Adversarial Mode (`--adversarial`)

When `--adversarial` is passed, restructure the advisor roles:

1. **2 Advocates** — Argue FOR the proposal regardless of personal opinion. Find every reason this is a good idea. Steel-man it.
2. **2 Skeptics** — Argue AGAINST the proposal regardless of personal opinion. Find every reason this will fail. Red-team it.
3. **1 Neutral Analyst** — No agenda. Evaluate the quality of arguments on both sides. Note which side has stronger evidence.

Peer review and chairman synthesis proceed as normal. The chairman explicitly notes which side made stronger arguments and why.

Use for proposals, pitches, "should we do X?" decisions, and any question with a clear yes/no framing.

---

## Cost Budget

| Mode | Agent Calls | Best For |
|------|-------------|----------|
| Full (default) | **11** (5 advisors + 5 reviewers + 1 chairman) | High-stakes decisions |
| Quick (`--quick`) | **4** (3 advisors + 1 chairman) | Routine decisions, gut-checks |
| Adversarial (`--adversarial`) | **11** (5 agents + 5 reviewers + 1 chairman) | Proposals, yes/no decisions |

Fast models for volume, good model for synthesis. The chairman's reasoning quality is what matters most.

## Gotchas

- **Always parallel spawn advisors.** Sequential lets earlier responses contaminate later ones.
- **Always anonymize for peer review.** Reviewers defer to "The Contrarian" or "First Principles" if they see the label. Shuffle the letters.
- **Chairman can override majority.** Quality of reasoning > vote count. If the Contrarian found a real flaw that everyone else missed, the chairman should side with them.
- **Don't council trivial questions.** The pre-flight check should catch these. If one right answer exists, just answer it.
- **Context is critical.** Generic input = generic output. The auto-context step reads project files so advisors aren't flying blind.
- **Same-model limitation.** This skill uses perspective diversity (different reasoning methods on the same model), not model diversity (Karpathy's original used different LLMs). For the highest-stakes decisions, consider also getting a second opinion from a different model family.

## Credits

Original concept: Andrej Karpathy (LLM Council)
Claude Code adaptation: Ole Lehmann (@itsolelehmann)
Reasoning method diversity: Informed by DMAD (ICLR 2025)
Skill by: Neal Meyer
