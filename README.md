# Claude Skills

Open-source [skills](https://docs.claude.com/en/docs/claude-code/skills) for Claude Code that I use myself as a solo founder. Sharing the patterns that pay off.

## Available Skills

### [gemini-review](./gemini-review/)

Cross-model independent review for high-stakes artifacts. Delegates review to Gemini CLI via OAuth (free, 1000 req/day, no API keys). Catches blind spots that single-model self-critique structurally misses.

**Use case:** strategy docs, design proposals, ad copy, architecture, economic models — anywhere RLHF priors might converge with the author's blind spots.

**Real catches in production:**
- Paywall math bug (forgot to count daily credit refills) — would have inverted free vs paid economics
- Speed-flipping exploit (swipe through 200-page book in 60s to farm credits)
- Missing founder presence in B2B partnership PDF
- Mis-framing of "translation" as premium feature when it's table stakes

**Quickstart:**

```bash
npm install -g @google/gemini-cli@latest
gemini auth login   # opens browser, OAuth via Google account

mkdir -p .claude/skills/gemini-review
curl https://raw.githubusercontent.com/anydasa/claude-skills/main/gemini-review/SKILL.md \
  -o .claude/skills/gemini-review/SKILL.md
```

In Claude Code: just say "ask Gemini" or "second opinion".

[Full documentation →](./gemini-review/README.md) · [Why & how (Habr, RU)](https://habr.com/ru/articles/1028104/)

## License

MIT

## Author

Artem Sykchin — solo founder of [Osmo Lingo](https://osmolingo.app), reading app for foreign-language classics.

- 🤖 Telegram Mini App: [t.me/OsmoLingoBot](https://t.me/OsmoLingoBot)
- 🌐 Web Application: [app.osmolingo.app](https://app.osmolingo.app)
- 💬 Contact: [@anydasa](https://t.me/anydasa)
