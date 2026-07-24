# Changelog

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
