---
name: gemini-review
description: Use when the user explicitly asks for a second-opinion review from Gemini on a freeform artifact — strategy doc, partner-offer PDF, ad copy, architecture proposal, positioning text, DM draft. Delegates to Gemini CLI via OAuth (no API tokens, free 1000 req/day). Returns Gemini's verbatim critique, then optional brief synthesis. Do NOT auto-invoke — only when user explicitly requests review/second-opinion/judge from Gemini, or for high-stakes external-facing artifacts where the user has signaled they want validation.
---

# Gemini second-opinion review

A thin wrapper that delegates structured review tasks to Gemini CLI. Gemini reads at full main-context level, with different RLHF priors than Claude — catches things Claude misses (authorship blind spots, sycophancy patterns, alternative framings).

## When to use

✅ User explicitly asks: "ask Gemini", "what does Gemini think", "second opinion", "judge this", "discuss with Gemini"

✅ Artifact qualifies:
- Freeform / subjective (strategy, copy, positioning, architecture, marketing)
- External-facing OR high-stakes (≥30 min rework cost if wrong)
- Has clear audience and goal you can articulate in one sentence each

✅ Stakes warrant second look:
- Will be sent to a real person (creator DM, partner pitch)
- Affects monetary decisions (pricing, budget allocation)
- Architecture decision that's hard to reverse

## When to SKIP (don't invoke)

❌ Code that compiles and has tests — tests are the judge

❌ Decisions where Claude already disagreed and user committed — calling Gemini will only add noise

❌ Same artifact within same session unless content materially changed — sycophancy ratchet risk

❌ User asks > 2 times in a row for "another opinion" on the same thing — they're sycophancy fishing, not seeking critique. Push back.

❌ Routine outputs (memory updates, code edits, data queries, day-to-day execution)

❌ Tasks where you cannot articulate criteria — Gemini has nothing to anchor to, returns mush

## Invocation pattern

```bash
gemini --yolo -p "<structured prompt>" 2>&1 | tail -80
```

Critical flags:
- `--yolo`: bypasses interactive approval. Without it, gemini hangs in non-interactive mode (gemini-cli#21052, #16567).
- `2>&1 | tail -80`: trims CLI banner and progress output; keeps actual response.

For artifacts that are files, reference via `@`-syntax. Gemini natively reads:
- Markdown / text files
- PDF up to 1000 pages
- Images (PNG, JPG)
- Code files

```bash
gemini --yolo -p "Review @/abs/path/to/file.pdf for X" 2>&1 | tail -80
```

Pass paths, not pasted content. Gemini's `read_file` is more efficient than embedding huge text in the prompt.

## Prompt structure (load-bearing — don't skip these)

```
[CONTEXT — what the artifact is, 2-3 sentences]
- What product/business is this for
- What stage / status

[AUDIENCE — specific persona who will read/use it]
- Not "users" — concrete: "Russian-speaking philological Telegram creator with 11k followers selling reading-in-original webinars"

[GOAL — what success looks like]
- Concrete: "would convert this specific creator to recommend the app to her audience"
- NOT abstract: "be good"

[EVALUATE — 3-5 specific dimensions]
1. [Specific dimension 1]
2. [Specific dimension 2]
3. ...

[ANTI-CRITERIA — explicit "don't do this"]
- "Don't be diplomatic"
- "Don't hedge"
- "Don't praise — hunt for what's wrong"

[FORMAT — verdict + sections + single most-important fix]
Top: one-line verdict (e.g., CONVERTS / CONVERTS WITH FIXES / DOES NOT CONVERT)
Body: numbered sections matching evaluation dimensions
Tail: ONE most-important change to make

[ARTIFACT REFERENCE]
Either @/path/to/file or pasted content
```

**Authorship neutralization (important):** Don't tell Gemini "Claude wrote this draft" — measurable negative bias on Claude-labeled content (arXiv 2508.21164). Use neutral framing:
- ✅ "Review this artifact"
- ✅ "I drafted this"
- ❌ "Claude wrote this"

## Output handling

After Gemini responds:

1. **Return Gemini's critique verbatim** to the user, prefixed clearly: "**Gemini reviewer says:**" or similar attribution.

2. **Don't paraphrase Gemini's findings.** The user wants the actual second voice, not your summary.

3. **Optionally add a brief Claude synthesis** AFTER the verbatim critique IF user asks or if the criticism conflicts with prior decisions in this conversation.

4. **Distinguish three types of Gemini feedback:**
   - **Auto-fixable** (typos, jargon, robotic phrasing) — apply silently while reporting
   - **User-input fixes** (need their judgment, photo, story) — surface clearly
   - **Architectural disagreements** — present both views, let user decide. Don't silently override Claude's earlier reasoning.

## Failure handling

| Symptom | Diagnosis | Action |
|---|---|---|
| Exit code != 0 | CLI error or auth issue | Surface stderr, STOP. Do not retry — quota is shared 1000/day. |
| Empty `response` | Bad prompt | Tell user prompt was too vague, ask to refine. Don't guess. |
| Generic praise / no concerns | Sycophancy degradation | Flag "judge appears degraded on this artifact", don't apply mechanically. |
| Self-contradiction across sections | Model confusion | Same — flag and don't apply blindly. |
| Hangs > 60s | Forgot `--yolo` flag | Kill with timeout; rerun with `--yolo`. |

## Cost / rate limits

- **OAuth quota: 1000 req/day, 60 RPM** — resets midnight Pacific. ~$0 marginal cost.
- Single review = single request typically. Don't burn quota on routine outputs.
- If user has run > 5 reviews in one day, mention quota awareness in case of need to cut.

## Anti-patterns explicitly to avoid

| Anti-pattern | Why it fails |
|---|---|
| Auto-invoking on every artifact | Routinizes the trigger → taste atrophy + convergent blandness toward consensus middle |
| Two rounds with hint of disagreement | Position bias collapses; Gemini mirrors instead of taking a stance |
| "Claude wrote this" in prompt | Authorship-label bias documented (arXiv 2508.21164) |
| Asking for 1-10 score | Numeric scales degenerate; binary verdict + critique forces decisions (Hamel Husain) |
| Skipping audience/goal in prompt | Generic feedback, no actionable signal |
| Using Gemini to validate post-hoc decisions | "I had it Gemini-reviewed" becomes pseudo-rigor justification, not check |

## Example invocation (for reference)

```bash
gemini --yolo -p "Review the partner offer PDF at @distribution/partner-program/partner-offer-2026-04.pdf

CONTEXT: Solo founder partner-program proposal for a Telegram-based reading-classics app (970 users, 5 paying, MRR \$50, 4 months in). Goal: recruit 2-3 first creator partners.

AUDIENCE: Russian-speaking philological Telegram creators (~5-15k followers, examples: Ekaterina Baeva @mrdarcyandballs who sells 'reading in original' webinars).

GOAL: Would this PDF convert one such creator to test the app + recommend to their audience?

EVALUATE:
1. First-impression on phone (Telegram preview)
2. Tone calibration — reader-celebration positioning vs startup-deck vibe
3. Honesty/numbers gamble — does it filter right partners or repel them?
4. Concrete weaknesses (phrasing, design, missing info)
5. Pitch strength — is 30% revshare + free Pro + manual report compelling?

FORMAT: One-line verdict (CONVERTS / CONVERTS WITH FIXES / DOES NOT CONVERT) at top, then 5 numbered sections, end with single most-important fix.

Don't be diplomatic. Direct language only." 2>&1 | tail -80
```

## Track effectiveness over time

Maintain a simple track-record in memory: when you invoke gemini-review, note in `reference_gemini_review_workflow.md`:
- What was reviewed
- Verdict
- % of suggestions applied
- Whether the change visibly improved outcomes

Calibration targets:
- < 60% of suggestions applied → judge is sycophancy-noisy. Tighten criteria, or skip more.
- > 90% applied → either prompts are great OR you're under-using your own judgment. Audit.
- 60-90% applied → healthy zone.

## Notes on this stack specifically

- Gemini CLI version on this machine: check with `gemini --version`. Older versions (pre 0.6.1) have JSON-output flakiness — stick with plain text output for now.
- Gemini OAuth credentials live at `~/.gemini/oauth_creds.json` — don't touch.
- For very long context (entire repo / multi-file), `--all-files` exists but burns quota fast. Prefer specific `@`-references.
