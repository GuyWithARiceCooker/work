# Yibu work log

## 2026-11-28 — Hungarian speech input for the Yibu tutor

**Problem:** The Yibu tutor's speech-to-text only understands English. When the learner speaks Hungarian, the browser's SpeechRecognition engine rejects or mishears it because `r.lang` is set from the practice language tag.

**What we found:**

- The main app (`yibu/index.html`) does **not** upload audio to the Cloudflare STT worker. It uses the browser's built-in `SpeechRecognition` / `webkitSpeechRecognition` directly.
- In `startRec()`, the recognizer is created with `r.lang = recLang`, where `recLang` comes from `speechTag(practiceLang())`. If the practice language is English, `speechTag` returns something like `en-US`, so Hungarian speech is ignored.
- The separate `yibu-stt-test` worker (Whisper large-v3-turbo) is only used by `dictation-test.html` — a test page, not the live tutor. That worker has no `language` parameter wired up yet.
- The `maria-tutor` worker's system prompt (in `tutor-reply.js`) already allows replying in Hungarian when the learner writes in Hungarian, so the chat side is partly ready.

**Plan (not yet implemented — Cloudflare tool was unavailable):**

1. **Frontend (`index.html`):** make `recLang` bilingual — e.g. `r.lang = "hu-HU,en-US"` or detect the learner's actual spoken language and switch. Simplest robust option: set `r.lang` to a comma-separated list of the interface language and the practice language so both are accepted.
2. **STT worker (`yibu-stt-test`):** add an optional `language` query/header param passed through to the Whisper `@cf/openai/whisper-large-v3-turbo` call.
3. **maria-tutor prompt:** already handles Hungarian replies; no change strictly needed, but could be tightened.

**Next step:** retry the Cloudflare connector and apply the two worker/frontend edits, then test with Hungarian speech on the live site.
