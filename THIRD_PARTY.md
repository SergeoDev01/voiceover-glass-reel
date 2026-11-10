# Third-party components

This repository is MIT-licensed, but it depends on components and software that
keep their own licenses. This file lists them. When in doubt, the license file
shipped with the component wins.

## MCP servers and tools (not bundled)

| Component | License | Source |
|---|---|---|
| `pixabay-mcp` | MIT | https://github.com/zym9863/pixabay-mcp |
| `@thesvg/mcp-server` | MIT (code) | https://github.com/glincker/thesvg |
| thesvg icon registry | Brand logos are trademarks of their owners | https://thesvg.org |
| ffmpeg | LGPL 2.1+ / GPL 2+ depending on the build | https://ffmpeg.org/ |
| curl | curl license (MIT-like) | https://curl.se/ |
| Python | PSF License | https://www.python.org/ |
| After Effects | Commercial Adobe license — **required** for build/render | https://www.adobe.com/products/aftereffects.html |
| Xcode Command Line Tools / Swift | Apple license | https://developer.apple.com/ |

## Bundled code

| Component | License |
|---|---|
| `scripts/ae/build_reel.jsx`, `scripts/ae/reel_lib.jsx` | MIT, as part of this repository |
| `scripts/reel.py`, shell scripts | MIT, as part of this repository |
| `scripts/tools/*.swift` | MIT, as part of this repository |

## SFX and graphics

| Asset | Origin | License |
|---|---|---|
| Sound effects | Synthesised locally by `reel.py sfx` | No third-party license |
| Glass frames, grid, captions, pages | Drawn by the After Effects build | MIT, as part of this repository |

## Media sourced at runtime (not in this repository)

| Source | License | Attribution |
|---|---|---|
| Pixabay | Pixabay Content License | Not required |
| Wikimedia Commons | Per-file license (public domain, CC BY, CC BY-SA, …) | Per file — record it in `work/SOURCES.md` and `work/CREDITS.md` |
| thesvg | Brand logos / trademarks | Identification use; check the brand's rules |

The skill writes `work/SOURCES.md` and `work/CREDITS.md` inside each project.
Keep them with the rendered video, especially for Wikimedia Commons material.
