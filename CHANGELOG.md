# Changelog

## 0.3.0 — 2026-08-01

**The skill is no longer distributed as a file. Connect the hosted MCP instead.**

- `vidmatic/record.md` is **removed**. The workflows are served as prompts by the
  hosted MCP at `https://api.vidmatic.ai/mcp`, so connecting the server is the
  whole install and you always run the current version.
- Install is one line:
  `claude mcp add --transport http --scope user vidmatic https://api.vidmatic.ai/mcp --header "Authorization: Bearer vk_sk_..."`
- Commands are now `/mcp__vidmatic__record` and `/mcp__vidmatic__audit_ux`
  (the read-only narrated UX critique — new here, shipped a while back).
- **Requirements corrected to Claude Code + Node.** Python and ffmpeg were listed
  as prerequisites and never should have been: Vidmatic transcodes and renders
  server-side.

### If you installed 0.1.0 or 0.2.0, migrate

The copy you made is stale and cannot be updated from here. It still hard-requires
ffmpeg, stops unless the project has a `CICD.md`, expects a committed
`@playwright/test` harness, and calls an upload API that has since changed — so it
cannot publish. Delete it (both locations the old README suggested) and connect the
MCP:

```bash
rm -rf .claude/commands/vidmatic ~/.claude/commands/vidmatic
```


## 0.2.0 — 2026-07-25

- ffmpeg is now **optional**: the skill uploads the raw `.webm` and Vidmatic
  transcodes it to MP4 server-side. Developers no longer need ffmpeg to publish.
  (ffmpeg, if present, is used only for a nicer local MP4 deliverable.)


## 0.1.0 — 2026-07-24

Initial public beta of the `/vidmatic:record` Claude Code skill.

- Records a narrated demo of recent work by driving the running app with
  Playwright (headless, video on), post-processing to MP4 with ffmpeg, and
  publishing to a shareable vidmatic.ai URL.
- Narration is timed to observed scene boundaries, not the storyboard's
  intended timing, so the voice never drifts off what's on screen.
- Degrades cleanly to a local silent MP4 when no API key / MCP server is
  present — never a partial upload.

Rolling out: the `vidmatic-mcp` uploader package and the `/vidmatic:init`
smoke-test command.
