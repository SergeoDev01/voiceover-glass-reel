# Troubleshooting

| symptom | fix |
|---|---|
| `ae_run` exit 6 "AE did not start the script" | AE is closed, or a dialog is open in AE (close it). The first time, macOS asks to let the terminal control After Effects: allow it under System Settings → Privacy & Security → Automation. Also turn on AE Settings → Scripting & Expressions → Allow Scripts to Write Files and Access Network. |
| exit 4 "still running" | AE is busy with the script. Do not resend. Wait, then read the log path it printed. |
| "Another After Effects project with unsaved changes is open" | Ask the user to save or close their project. Never close it without saving. Reopen it after the build. |
| transcription fails or finds no words | System Settings → Privacy & Security → Speech Recognition: allow the terminal app. For other languages, set `REEL_LOCALE=de-DE` (etc.) before `reel.py prep`. Transcription needs **macOS 26+** (on-device SpeechAnalyzer). On older macOS, get per-word times another way, save them as `{"words": [{"w", "start", "end"}]}` and run `REEL_WORDS=<file> reel.py prep ...`. |
| `swiftc` not found | `xcode-select --install`, then `scripts/build_tools.sh` |
| voice cut too tight or too loose | The threshold is the speech median − 6 dB (printed by `prep`). For a noisy recording, clean it first, or raise the speech level. |
| host lips out of sync | `reel.py align` again. The host clip's audio track must BE the chunk (rebuilt with `ffmpeg -map 1:a`, §6 of `references/higgsfield.md`); then align is exact. With paid appendix clips: regenerate with the same chunk and make sure `generate_audio` was on. A shot's lips may run up to ±0.1 s off after the median word match. |
| no green screen found | Re-source the plate: a straight-on vintage TV photo plus an `ffmpeg drawbox` green fill (§2 of `references/higgsfield.md`), re-run `screen`. Two failures → switch the host shots to `full` mode (no plate needed). |
| black gap at the end of a shot | The clip is shorter than its shot. The build then holds the last frame and warns. Choose an earlier `in` or a longer clip. |
| fonts look wrong | The build log prints the chosen fonts. Install SF Pro (Apple) for the closest look, or edit `FONT` in `build_reel.jsx`. |
| build is slow (> 3 min) | A normal build takes about 1 min for a 70 s reel. A new text layer costs about 1.7 s in AE 2025–26, so the build makes each text style once (`__text_styles`) and copies it. Keep it that way when you add text. Also close heavy comps in the viewer: the build parks the viewer on a tiny comp. |
| render: "no H.264 output template" | AE before 2023: `render.sh` falls back to "High Quality" (.mov). Convert it afterwards if an MP4 is needed. |
