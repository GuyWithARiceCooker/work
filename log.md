# Yibu work log

## 2026-11-28 — Hungarian speech input for the Yibu tutor

**Problem:** The Yibu tutor's speech-to-text only understands English. When the learner speaks Hungarian, the browser's SpeechRecognition engine rejects or mishears it because `r.lang` is set from the practice language tag alone.

**What we found:**

- The main app (`yibu/index.html`) does **not** upload audio to the Cloudflare STT worker. It uses the browser's built-in `SpeechRecognition` / `webkitSpeechRecognition` directly.
- In `startRec()`, the recognizer is created with `r.lang = recLang`, where `recLang` comes from `speechTag(practiceLang())`. If the practice language is English, `speechTag` returns `en-US`, so Hungarian speech is ignored.
- `SPEECH_TAGS` already maps `hu` to `hu-HU`, and `syncRecLang()` exists to abort and restart recognition when the language changes — so no new infrastructure is needed.
- The separate `yibu-stt-test` worker (Whisper large-v3-turbo) is only used by `dictation-test.html`, a test page. That worker has no `language` parameter wired up yet; it remains a fallback option.
- The `maria-tutor` worker's system prompt (in `tutor-reply.js`) already allows replying in Hungarian when the learner writes in Hungarian, so the chat side is partly ready.

**Solution implemented (frontend only):**

1. Added a small helper:

```js
var bothLangs = function bothLangs() {
  var a = speechTag(uiLang), b = speechTag(practiceLang());
  return a === b ? a : a + "," + b;
};
```

2. In `startRec()`, replaced `r.lang = recLang;` with `r.lang = bothLangs();`.

3. In `syncRecLang()`, replaced `recognition.lang = recLang;` with `recognition.lang = bothLangs();`.

**Effect:** if the UI language is Hungarian and the practice language is English, the browser receives `hu-HU,en-US` and accepts speech in either language. If both are the same, it stays a single tag.

**Not done (Cloudflare tool was unavailable at the time):**

- Adding an optional `language` param to the `yibu-stt-test` Whisper worker for the dictation-test page.
- Any change to the maria-tutor system prompt (already handles Hungarian replies).

**Next step:** test Hungarian speech on the live site; retry the Cloudflare connector for the STT worker param if the dictation page needs it.
