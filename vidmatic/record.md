Record a polished, **narrated** demo video of recently-completed work: drive the running app with Playwright (headless, video on), post-process to MP4 with ffmpeg, then publish it to **vidmatic.ai** with an AI voice-over pinned to what is on screen. Fully autonomous — designs the journey from what was just built, records it, narrates it, and prints a public URL.

Optional `$ARGUMENTS` names a focus (e.g. `vidmatic:record dashboard`); if empty, demo the headline features of the most recent work.

**Two things make or break this.** First, the video is recorded **to fit the script**, not the other way round — the script is written and fit-checked BEFORE recording, and scene dwell times are widened to match. Second, narration is timed to **observed** scene boundaries measured during the run, never to the storyboard's intended `dwellMs`: the real run always drifts, and the drift is cumulative, so by the last scene an intention-timed voice is describing the wrong thing.

If no vidmatic API key is configured, this degrades cleanly to a local silent MP4 — never a partial upload, never an error.

---

## Step 0: Environment

Read `CICD.md` in the project root. From **Local Development**, extract (do NOT hardcode): the rebuild command, the frontend URL, the API URL, and their health endpoints. If `CICD.md` has no such section, stop and ask the user to populate it.

1. **Assert the root `.env` exists BEFORE rebuilding.** The web image bakes `NEXT_PUBLIC_*` at build time from the root `.env`. If it is missing, the rebuild silently bakes an empty Google client id: `/status` still returns 200, so the stack looks healthy, and sign-in is broken — which you only discover when the demo records a broken auth screen. If it is missing, stop and say so.
2. `command -v ffmpeg || echo MISSING` — **optional.** Vidmatic transcodes your recording server-side, so a publish never needs local ffmpeg. If ffmpeg IS present it is used for a nicer local MP4 deliverable and a pre-flight size check; if absent, the raw `.webm` is uploaded and the server does the MP4 work. Do not stop when it is missing.
3. Run the rebuild command exactly as written. Wait for BOTH health endpoints to return 200. Fix any build error before recording — never record a stale or broken stack.
4. Ensure the Playwright harness is ready. Three separate things, each of which has failed on a fresh checkout:
   - **Files:** `web/playwright.config.ts` and `web/e2e/demo/` are committed (config, `_overlay.ts`, `auth.ts`, `timings.ts`).
   - **Package:** `cd web && node -e "require.resolve('@playwright/test')"`. If it throws, run `npm install` in `web/` — the dependency is already pinned in `package-lock.json`, so this installs it without lockfile churn. Do not skip this check because the dependency is listed in `package.json`; a `node_modules` predating the harness satisfies neither.
   - **Browser binary:** `npx playwright install chromium` once if missing (`npm ci` deliberately does not download it).
5. Check the publish path: is the `vidmatic` MCP server connected AND is `VIDMATIC_API_KEY` set? Record the answer — Step 6 branches on it. Do not fail here if it is absent.

Print the resolved values and a **runtime budget**: rebuild ~2 min, record ~2 min, ffmpeg ~1 min, narration render ~2–4 min. Total ≈ 8–10 min.

---

## Step 1: Learn what was just built

Gather, in the main context:
- The newest planning/verification artifacts by mtime (`PRD*.md`, `VERIFY_*.md`, …).
- `git log --oneline -20` plus the diffstat of recent commits.
- The route map (`web/src/app/**/page.tsx`) for real URLs.
- If `$ARGUMENTS` names a focus, bias toward it.

Pick the **5–9 headline things a viewer should be wowed by** — journeys and signature UX, not edge cases. Prefer a narrative arc: discover → create → use → deliver.

---

## Step 2: Storyboard — with narration

Write `demos/<project>_storyboard_<YYYYMMDD_HHMM>.json`:

```json
{
  "project": "<name>", "frontendUrl": "<url>", "viewport": {"width":1280,"height":800},
  "voiceId": "af_heart",
  "title": { "headline": "<Product>", "subhead": "<one-line value prop>", "durationMs": 3000 },
  "scenes": [
    {
      "id": "landing",
      "route": "/",
      "caption": "Scan a QR — no app, no login.",
      "narration": "Every guest joins with a single scan. No app to install, no account to create.",
      "actions": ["scrollTo:#how-it-works", "wait:1500", "highlight:text=Create your album"],
      "dwellMs": 6000,
      "auth": "none"
    }
  ],
  "outro": { "headline": "Ready in ~90 seconds", "cta": "<url>", "durationMs": 3000 }
}
```

`narration` is **distinct from `caption`** and both are required. A caption is read; narration is heard. Writing rules:

- **1–2 sentences, written to be heard.** No parentheses, no slashes, no "e.g.", no UI jargon a listener cannot see.
- **Benefit-led**, not mechanical: "Every guest joins with a single scan", not "The user clicks the Create button".
- **Continuous across scenes** — it is one spoken paragraph, not six captions. Avoid restating the product name every line.
- **Budget ≈ 2.5 words per second.** A 6-second scene holds about 15 words. Going over does not fail loudly, it just makes the narrator rush.
- **`""` is a legitimate line** — a deliberate pause. Use it for a scene that should breathe (a slideshow, an animation).

Pacing: one idea per scene, 4–8s typical, longer for motion. Group signed-in scenes so you authenticate once. Total ~45–90s.

---

## Step 2b: Preflight BEFORE recording — then widen the video to fit

Build a provisional `NarrationScript` from the storyboard's intended `dwellMs` (cumulative windows, in **integer milliseconds**), and call `preflight_narration` (MCP) or `POST /api/demos/preflight`.

For every segment reported `over`:
1. **Widen that scene's `dwellMs`** toward `suggested_window_ms`, and/or
2. **Cut words** — the response says roughly how many.

Re-preflight until nothing is `over`. **Then update the storyboard and record.** This is the whole point of doing it here: the video is recorded to fit the script. After recording, a too-long line can only be fixed by cutting words — you cannot lengthen a scene that is already on tape.

Skip this step only if there is no API key; note in the summary that fit was not checked.

---

## Step 3: Generate the Playwright demo spec

Write `web/e2e/demo/demo.spec.ts` (gitignored — regenerated per run), importing the committed helpers:

- `_overlay.ts` → `installOverlay`, `showCaption`, `cursorClick`, `highlight`, `smoothScrollTo`
- `auth.ts` → `injectAuth` (localStorage + cookie via `addInitScript`; real OAuth cannot complete on localhost)
- `timings.ts` → `timings.start()`, `markScene(id)`, `flushTimings(path)`

Structure:
- One `test`, config-driven video (`video: 'on'`), storyboard viewport.
- `timings.start()` on the **first frame** — every window is relative to it, so a late origin shifts the entire script.
- Title card, then each scene: `markScene(scene.id)` FIRST, then navigate, `waitForLoadState`, run `actions`, `showCaption`, dwell `dwellMs`. Re-install the overlay after every navigation (a page load wipes it).
- Outro card, then `flushTimings('demos/<project>_timings_<stamp>.json')`.
- Deterministic: explicit waits, fixed dwell.

Drive the **real UI**. The only allowed shortcut is attaching a generated file to a real `<input type=file>` so the app's own upload code runs — never call mutation APIs to fabricate a scene.

---

## Step 4: Record (Sonnet subagent)

Dispatch ONE `general-purpose` subagent, `model: "sonnet"`, foreground:
- Run only the demo spec: `cd web && npx playwright test e2e/demo/demo.spec.ts --project=chromium`.
- **Return contract:** the recorded `.webm` path(s), the observed run duration, AND the path to the emitted `_timings.json`. The timings file is not optional — Step 5b depends on it.
- On failure, return the exact Playwright error and failing selector. Do NOT fix — main context owns the spec.

Up to 3 attempts, fixing the spec in main context between them.

---

## Step 5: Prepare the upload artifact

**The artifact you upload is the raw `.webm`.** Vidmatic transcodes it to MP4 (H.264, faststart, viewport-sized), validates the dimensions, and renders the narration — all server-side. There is no required local ffmpeg step.

Concat first only if the run produced multiple `.webm` segments (a rare multi-context recording); a single-context run yields one `.webm` and needs nothing.

**If ffmpeg IS present** (from Step 0), a Sonnet subagent may ALSO, purely for a nicer local deliverable:
- `ffprobe` the `.webm` and assert it matches the storyboard viewport (1280×800). If it is smaller, report it — an undersized capture means `video.size` is missing or disagrees with `viewport` in `playwright.config.ts`. (Server-side validation is the backstop when ffmpeg is absent.)
- Transcode a local MP4 (`libx264 -crf 20 -preset medium -pix_fmt yuv420p -r 30 -movflags +faststart`) + a poster PNG to `demos/<project>_demo_<stamp>.mp4` for Step 7 delivery.

**If ffmpeg is absent**, skip all of the above — the `.webm` is both the upload and the local deliverable.

**Do the upload (Step 6) BEFORE deleting the intermediate `.webm`/`.video` directory.** Cleaning up first means a failed upload has nothing left to retry from.

---

## Step 5b: Re-preflight on the OBSERVED timings

Read the `_timings.json` from Step 4 and build the final `NarrationScript`:
- `segments[i].start_ms` / `end_ms` = the **measured** scene window (integer ms).
- `scene_id` = the storyboard scene id.
- `text` = that scene's `narration`.
- `timing_source: "observed"`, `source: "demo_skill"`, `voice_id` = the storyboard's voice.

Call `preflight_narration` again with `duration_ms` = the MP4's real duration. **This verdict is what you report.** The pre-record check used intentions; this one uses reality, and the two always differ.

The only remedy available now is `suggested_words_delta` — **cut words**. Do not re-record to fix a line.

---

## Step 6: Publish

**If the MCP server + `VIDMATIC_API_KEY` are available:**

1. `create_demo(file_path, script, title, publish: true)` → **202** with `entity_id`, `status_url`, `public_url`. `file_path` is the raw **`.webm`** (or the local MP4 if ffmpeg produced one — the server accepts either and transcodes webm as needed).
2. Poll `get_demo_status(entity_id)` until `ai_twin_status` is `ready` or `error` (every ~10s, give up after ~10 min and print the status URL).
3. On `ready`, print the public URL. **Do not call `set_default_video`** — a demo created with `publish: true` is promoted and made public automatically when its render succeeds, and never before (it is never publicly reachable as a silent video).
4. On `error`, print the message and the recording URL. The video is uploaded and safe; the script can be fixed in the app's narration editor or re-submitted.

**Never blind-retry `create_demo`** — it would upload a second copy of the same video. If it fails *after* the upload, the 422 body carries the `entity_id`: fix the script and call `submit_narration(entity_id, script)` instead.

Persist the last `entity_id` per storyboard slug under `demos/`. On a re-run, print: *"Previous demo: <url> — that link keeps playing the old video."*

**Publishing must never fail the recording.** Any error here still ends with the local artifact (MP4 if ffmpeg produced one, else the `.webm`) delivered.

**If there is no key or no MCP:** deliver the local artifact (MP4 or `.webm`) and print where to get one:

> To publish narrated demos, create an API key at `<frontend>/settings/developer`, then connect the vidmatic MCP server — full setup at https://www.vidmatic.ai/use-cases/generate-demo-video.

---

## Step 7: Deliver

- `SendUserFile` the local artifact — the MP4 if ffmpeg produced one, otherwise the `.webm` (caption: what the demo shows) + its path and the storyboard path.
- Print the public URL when published.
- Summary: scenes shown, runtime, resolution, output files, the **observed vs intended** timing delta, and any narration line that had to be trimmed.
- Confirm `demos/` is gitignored — these are large binaries and must never be committed.

---

## Conventions & guardrails

- **Read everything from `CICD.md` / the repo at runtime.** No hardcoded ports, paths, project names or selectors.
- **Milliseconds, always.** `start_ms` / `end_ms` are integers. A seconds value in a `_ms` field fails *silently*: every window lands past the end of the video, gets truncated, and the demo renders "successfully" with no audible narration.
- **Real flows, not fakes.** Drive the actual UI.
- **Auth on localhost:** inject the stored token via `auth.ts` — real OAuth cannot complete on `localhost`, and no dev-login bypass ships in production for this.
- **Determinism > speed.** Generous dwell and explicit waits make a watchable video; this is the opposite of a test run.
- **Heavy work in Sonnet subagents** (record, and the optional local ffmpeg); authoring stays in the main context. The publish path itself needs no local ffmpeg — the server transcodes.
- **Non-destructive:** clean up demo content you created, never touch originals, never commit `demos/`.
