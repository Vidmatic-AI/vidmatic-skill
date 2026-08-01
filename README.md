# vidmatic-skill

Vidmatic gives [Claude Code](https://claude.com/claude-code) two workflows that end in a shareable,
narrated video of your own app:

| Command | What it does |
|---|---|
| `/mcp__vidmatic__record` | Records a **narrated demo** of what you just built — drives your running app with Playwright, adds an AI voice-over pinned to what is on screen, publishes a public [vidmatic.ai](https://www.vidmatic.ai) URL. |
| `/mcp__vidmatic__audit_ux` | Records a **narrated UX critique** of any page — judges it against a five-criteria rubric from the rendered pixels and circles each finding on screen. **Read-only by contract:** it never edits code, so you can aim it at production or at somebody else's site. |

The loop it is built for:

1. Build a feature in Claude Code.
2. Run `/mcp__vidmatic__record`.
3. Paste the URL to your product team — they watch the feature instead of reading about it.

## Install

One line, once per machine. Create a key at **Settings → Developer** in the app — that page builds
this command with your key already in it:

```bash
claude mcp add --transport http --scope user vidmatic https://api.vidmatic.ai/mcp \
  --header "Authorization: Bearer vk_sk_..."
```

Restart Claude Code and both commands are there. The `Authorization` header is **required** — without
a valid key the server answers `401` and no commands appear.

Full, always-current setup walkthrough: **https://www.vidmatic.ai/use-cases/generate-demo-video**

## Requires

- Claude Code
- Node

That is the whole list. You do **not** need ffmpeg and you do **not** need Python — Vidmatic
transcodes the video and renders the narration server-side. (Earlier versions of this page asked for
both; if you installed them for Vidmatic, nothing here uses them.) The first run downloads a headless
browser; everything else is already on your machine.

## Why there is no skill file in this repo

There used to be one, and that was the bug.

The workflows are served **by the MCP itself**, as prompts. Connecting the server is the install —
you always get the current version, and there is no copy to keep in sync. When this repo shipped a
`vidmatic/record.md` for you to copy into `.claude/commands/`, that copy drifted: it went on
demanding ffmpeg and a `CICD.md` file long after neither was needed, and it called an upload API that
had already changed. A local copy cannot be fixed by us. A hosted prompt is fixed the moment we fix it.

### Already installed the old way?

If you ran the old `git clone … && cp -r … .claude/commands/` instructions, delete the copy — **both
places**, since the old README suggested either one:

```bash
rm -rf .claude/commands/vidmatic ~/.claude/commands/vidmatic
```

Then run the `claude mcp add` line above. A stale local copy will keep you on the broken flow
indefinitely; nothing upstream can reach it.

## License

MIT — see [LICENSE](./LICENSE).
