# Changelog

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
