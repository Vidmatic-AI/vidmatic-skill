# vidmatic-skill

A [Claude Code](https://claude.com/claude-code) skill that records a **narrated demo video** of what you just built — Claude Code drives your running app with Playwright, adds an AI voice-over timed to what's on screen, and publishes it to a shareable [vidmatic.ai](https://www.vidmatic.ai) URL.

The loop it's built for:

1. Build a feature in Claude Code.
2. Run `/vidmatic:record`.
3. Paste the URL to your product team — they watch the feature instead of reading about it.

> **Public beta.** Installing this skill works today. The `vidmatic-mcp` uploader package and the `/vidmatic:init` smoke-test command are still rolling out — [email us for early access](mailto:success@vidmatic.ai?subject=Vidmatic%20skill%20beta%20access). Full, always-current setup instructions live at
> **https://www.vidmatic.ai/use-cases/generate-demo-video**.

## Install

```bash
git clone https://github.com/Vidmatic-AI/vidmatic-skill.git /tmp/vidmatic-skill
mkdir -p .claude/commands && cp -r /tmp/vidmatic-skill/vidmatic .claude/commands/
```

That copies the `vidmatic/` command folder into your project. Restart Claude Code and `/vidmatic:record` is available.

Prefer it in every project? Copy into `~/.claude/commands/` instead — but pick **one**. If a user-level copy and a project-level copy both exist, the user-level one silently wins, and you can end up running a stale version without noticing.

## Requires

- Claude Code
- Node
- Python 3.11+ (for the `vidmatic-mcp` uploader)
- ffmpeg

The first run downloads a headless browser; everything else is already in place.

## Publishing (the narrated URL)

Recording produces a local MP4 with no setup at all. To publish it as a hosted, narrated demo you also need a Vidmatic API key and the `vidmatic-mcp` server connected. The setup page above walks through both; without them, `/vidmatic:record` degrades cleanly to a local silent MP4 — never a partial upload, never an error.

## Commands

| Command | What it does |
|---|---|
| `/vidmatic:record` | Records, narrates, and publishes a full demo of your recent work. |
| `/vidmatic:init` | *(rolling out)* One-shot smoke test — proves the whole pipeline with a short "Hello DEMO via vidmatic!" clip of your landing page. |

## License

MIT — see [LICENSE](./LICENSE).
