# gemini-review skill

Cross-model independent review for Claude Code. Catches structural blind spots that single-model self-critique misses.

## Why

Self-critique converges inside RLHF priors. The author and reviewer share the same blind spots. Cross-model = different vendor = different things caught.

In production, this skill caught:

- **Paywall math bug** in a loyalty-credit design — I forgot that daily refills (50/day × 30) added 1500 invisible credits. Free tier was actually richer than the paid tier, structurally inverting subscription economics. Would have shipped.
- **Speed-flipping exploit** — completion was triggered by reaching the last page of a book. A user could swipe through a 200-page book in 60 seconds to farm 600 reward credits. I had not considered the abuse vector at all.
- **Missing founder presence** in a B2B partnership PDF — Claude had optimized brand voice and offer terms perfectly, but didn't notice the document had no me in it. Partners bet on people, not apps.
- **Mis-framing of premium** in a freemium architecture proposal — I had positioned translation as a premium feature; Gemini caught that for our user level (A2-B1) translation is table stakes, premium is pedagogy.

## When this helps

- Freeform artifacts (strategy, copy, design, architecture, economic models)
- No test or ground truth available
- Stakes ≥30 min rework cost OR external-facing
- Audience and goal articulable in 2 sentences

## When this does NOT help

- Code with tests (tests are the judge)
- Decisions you've already committed to (sycophancy fishing)
- Routine outputs (memory updates, day-to-day code)
- Artifacts without clear criteria → Gemini returns mush

## Installation

### Prerequisite — Gemini CLI

```bash
npm install -g @google/gemini-cli@latest
gemini auth login
```

`gemini auth login` will open a browser for Google OAuth. After that you're authenticated.

OAuth gives 1000 requests/day for free. No API keys needed.

### Install skill

```bash
mkdir -p .claude/skills/gemini-review
curl https://raw.githubusercontent.com/anydasa/claude-skills/main/gemini-review/SKILL.md \
  -o .claude/skills/gemini-review/SKILL.md
```

## Usage

In Claude Code, say:
- "ask Gemini" / "second opinion from Gemini"
- "judge this with Gemini"
- "обсуди с Gemini" / "что Gemini думает" (Russian)

Claude detects the skill, structures the review prompt, invokes Gemini, returns verbatim critique.

## Real example output

From the loyalty-credit design review (paywall math catch):

```
VERDICT — GOOD WITH FIXES

2. Paywall Risk: CRITICAL FAILURE. Max free potential
   = 250 (monthly) + 1500 (daily refills) + 2400 (4× 600 grants)
   = 4150 credits/mo. Your Premium Basic is 3000.
   You are literally paying users to NOT subscribe.
   Grant must not exceed 250-300 per week.

5. Abuse Vectors: You missed "Speed Flipping." If completion is just
   reaching the last page, a user will swipe through a 200p book in
   60 seconds to farm 600 credits. You need a minimum "time-on-book"
   or "unique pages viewed" (min 3 seconds/page) check before granting.

SINGLE MOST IMPORTANT FIX: Reduce the grant cap to 300 credits/week
to preserve the Premium Basic paywall.
```

Two critical bugs caught before shipping, in 30 seconds.

## Anti-patterns built into the skill

- ❌ Auto-invoking on every output (taste atrophy)
- ❌ "Claude wrote this" in prompt (authorship-label bias)
- ❌ Two rounds on same artifact in same session (sycophancy ratchet)
- ❌ Numeric scores (binary verdict + critique forces decisions)

## Track-record discipline

Maintain a log of `% suggestions applied`:
- **<60%** → judge is sycophancy-noisy, tighten criteria
- **>90%** → underusing own judgment
- **60-90%** → healthy zone

## Common gotchas

- **`--yolo` flag** is required for non-interactive use; otherwise gemini hangs on approval prompts
- **Output bloat** — pipe through `2>&1 | tail -120` to trim CLI banner and progress logs
- **Rate limit** — 1000 req/day shared across your Google account; `--all-files` on a big repo can blow it in one call

## Related work

- [Дементьев на Habr (RU) — original idea with Codex CLI](https://habr.com/ru/articles/1019588/)
- [Hamel Husain — LLM-as-judge guide](https://hamel.dev/blog/posts/llm-judge/)
- [Every.to compound-engineering plugin](https://github.com/EveryInc/compound-engineering-plugin)

## License

MIT — use freely. PRs welcome.

## Author

Artem Sykchin — [@anydasa](https://t.me/anydasa). Solo founder of [Osmo Lingo](https://osmolingo.app).
