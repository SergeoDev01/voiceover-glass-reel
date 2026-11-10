# The one prompt

Install the skill once. Unzip it into `~/.claude/skills/` so that the result is `~/.claude/skills/voiceover-glass-reel/SKILL.md`.
Then open After Effects, start Claude Code in any folder, and send:

```text
Use the voiceover-glass-reel skill. Here is my voiceover: /path/to/my_voiceover.wav
Turn it into the finished reel: cut out all pauses and breaths, write the visual story from what I say, source every picture
free of charge (Wikimedia Commons archival photos, Pixabay clips and portraits, logo SVGs via the thesvg connector - nothing
paid), and build it in After Effects. Every clip goes in a floating rounded glass frame with a thin glowing border and a smooth
push-in. When I list points, put a moving perspective grid behind them and flip each keyword's colour right as I say it. The main
captions are big and spread across the frame, and the glass cards land exactly on the cut. Check it, fix what looks off, and
render the final 1080p MP4.
```

Optional additions to the same message:
- `Script: <paste the text>`: names and spelling in the captions will then match the script.
- `Host: /path/to/my_webcam.mp4` (or a photo: `/path/to/my_photo.jpg`): lets the reel show a real person on the TV.
- `Look: colour, modern tech documentary`. The default is a black-and-white 1950s archival documentary.
- `Accent colour: #FF3B30`

There is no credit budget to approve - everything is free (Pixabay MCP, Wikimedia Commons,
thesvg MCP, local synthesis). Claude asks once along the way:
1. to approve the logo downloads (only names you actually say).
Everything else runs on its own. A Higgsfield connector is used only if you explicitly ask for
paid AI generation (see the appendix of `references/higgsfield.md`).
