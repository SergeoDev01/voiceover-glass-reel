# Asset sourcing — free pipeline (Pixabay, Wikimedia Commons, thesvg, local synthesis)

> This file keeps its historical name because `reel.py chunks` prints it. It now documents the
> **free** sourcing pipeline: no credits, no paid generation, no spend approval. Paid AI
> generation (Higgsfield / Seedance) survives only as an **optional appendix** at the very end,
> for when the user explicitly asks for it.

Everything visual is sourced free of charge:
- the **Pixabay MCP** (`search_pixabay_images`, `search_pixabay_videos`) and the **thesvg MCP**
  (`search_icons`, `list_variants`, `get_icon_url`, `get_icon`) when running inside OpenCode;
- **Wikimedia Commons** over plain HTTP (no key, no account) anywhere else;
- **local synthesis** (ffmpeg Ken-Burns moves, `reel.py sfx`) where libraries cannot help.

Before you start, add up nothing: there is no cost model. Ask the user only for
rights-related decisions (logos, their own face) — never for money.

## Tested retrieval methods (verified recipes - use them as written)

| source | tool / endpoint | what to verify after |
|---|---|---|
| Pixabay images | `pixabay.search_pixabay_images {query, image_type: "photo", orientation, per_page}` | direct URL `https://pixabay.com/get/..._640.jpg` — HEAD gives `image/jpeg` + exact size |
| Pixabay videos | `pixabay.search_pixabay_videos {query, video_type: "film", orientation, per_page, min/max_duration}` | direct URL `https://cdn.pixabay.com/video/...medium.mp4` — HEAD gives `video/mp4` + size (e.g. ~16 MB for 40 s) |
| Wikimedia video | `GET https://commons.wikimedia.org/w/api.php?action=query&list=search&srsearch=filetype:video <words>&srnamespace=6&srlimit=5&format=json` | `prop=imageinfo&iiprop=url\|size\|mime\|duration\|sha1` → download URL + `sha1` |
| Wikimedia images | same API with `filetype:image` (or `filetype:bitmap`) | `imageinfo` gives `image/jpeg`, size, `sha1` |
| Wikimedia audio | same API with `filetype:audio` | usually `.ogg`/`.flac` — convert to MP3/WAV for After Effects |
| thesvg icons | `thesvg.search_icons {query}` → `list_variants {slug}` → `get_icon_url {slug, variant}` | CDN URL `https://cdn.jsdelivr.net/gh/glincker/thesvg@v0.6.0/...svg` — HEAD gives `image/svg+xml` |

Networking rules that were tested today on a real download:
- Always **HEAD first**, keep the `content-type`, and compare `content-length` with the file you
  land on disk. A buffer-download of a full video can time out; HEAD does not.
- For Wikimedia uploads, a bare tool default User-Agent got **HTTP 429** (rate limited). Use
  `curl -L` with a descriptive UA **and** retries - it succeeds where `Invoke-WebRequest` fails.
- Compare the **SHA-1** from `imageinfo` with the local file to prove integrity (tested: exact
  match on a 15.4 MB `.ogv`).
- Pixabay / thesvg / Commons licensing: Pixabay content licence, Commons per-file licence (many
  public-domain archival assets - perfect for the default look), thesvg SVGs via jsDelivr. Record
  every file, source page and licence in two ledger files: `work/SOURCES.md` and
  `assets/logos/SOURCES.md`.

Universal verification snippet (bash; macOS has curl and shasum):

```bash
# HEAD check
curl -sIL "$URL" | grep -iE "HTTP/|content-type|content-length"
# download (the UA line is what kept Wikimedia from 429-ing us)
curl -L -o "$out" -A "AZReel-sourcing/1.0 (contact: project user)" --retry 4 --retry-delay 8 --retry-all-errors "$URL"
# size + integrity
ls -l "$out"
shasum -a 1 "$out"   # compare with imageinfo sha1 (Wikimedia only)
```

## Shared look (instead of the old "style line")

The default is black-and-white 1950s-60s archival documentary. Sources that match rubric:

- Wikimedia Commons gives real **period-accurate archival photography** (public domain in large
  part) - prefer it for characters, locations and cards in the default world;
- Pixabay gives clean modern footage/stills - prefer it for a colour/modern subject;
- either way the pick must survive B&W: strong shapes and contrast, a clear subject, recognisable
  in half a second, **no readable text, logos or watermarks in frame**;
- keep one look for the whole reel. For a modern or colour subject, swap era keywords in the
  queries, not the pipeline.

## 1. Host portraits (stills, for cards and references) — 1-3 images
- Source from Pixabay (query: `presenter portrait studio medium close-up facing camera`,
  `photo`, per_page ≥ 5) or, better, ask the user for a real photo of themselves (free, no
  licence worry). A Commons portrait works too (`filetype:image`).
- The portrait is only a card item image or a reference never generated into lip-sync - so no
  facebox step is needed for portraits at all (that was for the paid video pipeline).

## 2. TV plate (only IF the story keeps `host` shots in `tv` mode)
Skip the plate entirely if there are no host-on-TV shots: no `tv` mode → no plate → no green
screen step. Choose one:
1. **No host shots at all (default when the user has no footage of themselves)** - restructure
   those beats into `type`, `giant` or `cards` pages (§6 below). Nothing else changes.
2. **Source + paint a plate**: pick a straight-on vintage CRT photo (Pixabay query
   `vintage television front view` or Commons `filetype:image vintage television 1950s`), then
   paint the screen flat chroma green with ffmpeg and validate with the engine:
   ```bash
   ffmpeg -y -i tv_photo.jpg -vf "drawbox=x=X:y=Y:w=W:h=H:color=0x00B140:t=fill" assets/tv_plate.png
   PY S/scripts/reel.py screen <proj>/assets/tv_plate.png <proj>
   ```
   Adjust X/Y/W/H until `screen` reports a sane box (TV should fill roughly a quarter of the
   frame width). Two failed tries → go with option 3.
3. **`full` mode host shots** - no plate, no keying needed.

## 3. Character portraits (one per recurring character)
- Default look: Wikimedia Commons period photographs - query the era and role
  (`filetype:image 1950s office clerk portrait`…), check the licence field, record it.
- Modern look: Pixabay portraits. Keep the same rule as the old sheets: one calm neutral
  face, unretouched skin, no real public figures, no text/logos.
- These stills feed `cards` items directly (`file` accepts an image) - no faceboxing.

## 4. Locations (3-6)
- Still photo per location from Commons (archival) or Pixabay (modern); query place + era +
  light (`1950s factory floor photograph`, `subway interior`...).
- For motion inside `clip` shots, Ken-Burns it (§5). Locations may also stay stills used as
  `cards` items.

## 5. Story clips (the bulk: ~1 clip per 3-5 shots)
Two free motions, both `.mp4`. One 8-12 s clip feeds several shots through different `in`
times - same grammar as the generated pipeline, so `story.json` math is unchanged.

1. **Real stock video**: Pixabay `search_pixabay_videos` (`video_type: "film"`, prefer
   `orientation` matching the frame, `min_duration: 8`, `max_duration: 15`), or Commons
   `filetype:video` (transcode before use - see below). Verify with HEAD, download with the
   verified curl recipe.
   ```bash
   ffmpeg -y -i in.ogv -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p -an out.mp4
   ```
2. **Ken-Burns from a sourced still** (archival photos often have no video twin):
   ```bash
   ffmpeg -y -loop 1 -i still.jpg \
     -vf "scale=-2:1080,zoompan=z='min(1.0+0.0009*on,1.15)':d=<seconds*25>:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':s=1920x1080:fps=25" \
     -t <seconds> -r 25 -c:v libx264 -pix_fmt yuv420p -an out.mp4
   ```
Point each shot at its sub-second: `in` = sub-shot start + 0.3 s exactly as before. Never show
the same second twice. If a clip is shorter than its shot, choose an earlier `in` or re-source.

## 6. Host lip-sync clips — the one thing money used to buy
Free paths, in order:
1. **The user's own footage of themselves.** A webcam clip, any content, is honest inside the
   TV collage look because the lips move while the real voice-over plays. Make `align` exact by
   giving the clip the chunk's audio as its own track:
   ```bash
   PY S/scripts/reel.py chunks <proj>                 # writes work/host/host_NN.wav
   ffmpeg -y -i webcam.mp4 -i <proj>/work/host/host_NN.wav \
          -map 0:v:0 -map 1:a:0 -shortest <proj>/assets/host/host_NN.mp4
   PY S/scripts/reel.py align <proj>
   ```
   The clip audio now IS the chunk audio, so `align` matches the words perfectly; the lips move
   loosely, which reads as collage, not as error.
2. **No host footage** → no host shots: restructure hook, turns and CTA into `type` / `giant` /
   `cards` pages. Keep the caption cadence; the reel still lands.

## 7. Music (1 track, optional)
- Cheapest and cleanest: **omit** `music` from `story.json` - the build simply skips the bed
  (`build_reel.jsx` only adds it when present). The voice and the SFX carry fine.
- The user's own track: drop it at `assets/music.mp3`, set `music_db`.
- Wikimedia Commons audio: `filetype:audio` with the standard API recipe, then convert
  (After Effects does not import OGG):
  ```bash
  ffmpeg -y -i in.ogg -c:a libmp3lame -b:a 192k assets/music.mp3
  ```
- Pixabay has a music site but no MCP tool for it: if the user wants it, they download the track
  manually in the browser and hand you the file.

## 8. Logos (only for names that are spoken, user's OK required first)
- **thesvg MCP (tested)**: `search_icons {query}` → `list_variants {slug}` → prefer the
  `light` / `dark` / `mono` variant for the badge, `get_icon_url` for the CDN link. Verify with
  HEAD (`image/svg+xml`).
- Rasterise for the badge (PNG needed by the build): macOS has QuickLook
  (`qlmanage -t -s 512 -o out file.svg` → `file.svg.png`); or `rsvg-convert`/`inkscape` when
  installed. Fall back to Wikimedia's render when the logo is on Commons:
  `https://commons.wikimedia.org/wiki/Special:FilePath/<File name>.svg?width=512` gives a
  512 px PNG.
- Keep `assets/logos/SOURCES.md` (file, source page, licence) and get the user's OK on the list
  first. Invented characters: when no logo exists for a spoken AI-model name, ask the user or
  use the caption without the badge.

## Download hygiene (applies to every asset)
- Save into `<proj>/assets/...` under the names `story.json` uses; keep `work/SOURCES.md`
  updated with file → source URL → licence → author, date fetched.
- Never overwrite a sourced file silently: `_v2`, `_v3`.
- Every verified unit must appear in your final report to the user with its source and licence.

---

# Appendix — paid generation (Higgsfield / Seedance) — ONLY on explicit request

Do not open this section unless the user explicitly asks for AI-generated footage and accepts
the credit cost. If they do, the historical procedure below still holds.

Work through the Higgsfield tools: `generate_image`, `generate_video`, `generate_audio` (or
their `_batch` versions); `media_upload` + `media_confirm` for local files; `jobs_wait` until
jobs finish; download results into `<proj>/assets/`.

Before you start: check the balance, add up the whole list's cost and ask the user **once**.

Test one item of each kind first (one sheet, one clip), look at it, then batch the rest.

## Shared style line (put at the start of every image and video prompt)
> Black-and-white 16mm archival documentary footage from a 1950s social-experiment film: heavy natural film grain, slight gate
> weave, soft contrast, faint scratches and dust, real practical light, observational camera.

For a modern or colour subject, swap in a matching documentary look. Keep one look for the reel.

People: realistic adults, unretouched skin; modest period clothes (long sleeves, closed
necklines, long skirts); one calm expression per sheet; no real public figures; no text, logos
or watermarks in any picture.

## A1. Host portrait — `soul_cinematic` (or `seedream_v5_pro`), 16:9, 2k
User photo as `image_references`: "the same person as in the reference photo, keep the exact face."
> Medium close-up portrait of a presenter looking straight into the lens, calm and confident, plain softly lit studio wall
> behind, shoulders and head centred with room above the head, 50mm lens, natural skin texture, <wardrobe>. Single person, no
> text, no logos.

## A2. TV plate — `soul_cinematic`, 16:9, 2k (screen = flat #00B140, no reflections)
> Cinematic film still: a single vintage 1950s CRT television set standing on a low concrete plinth in a dim, minimal room with a
> seamless grey wall and one soft top light, straight-on eye-level view, the TV in the centre of the frame taking about a quarter
> of the frame width. The TV screen is a perfectly flat, uniform, solid chroma-green (#00B140) rectangle with no reflections, no
> glare, no image and no gradient. Black-and-white grade except the green screen. No people, no text, no logos.

## A3. Character sheets — `seedream_v5_pro` (host reference) / `cinematic_studio_2_5`, 16:9, 2k
Facebox recipe unchanged: black-box the small faces on the full-body panels before video uses a
sheet: `S/scripts/bin/facebox sheet.png sheet_facebox.png`.
> Three-panel character sheet on a seamless light grey studio background: left panel full-body front view standing upright, head
> to toe with both feet visible; middle panel full-body back view standing; right panel head-and-shoulders close-up. <The same
> person as in the reference photo in all three panels: keep the exact face.> Identical character in all three panels, one calm
> neutral expression. Look: <role in one line>. Wearing <full modest period wardrobe, colours and materials>. Visible fine skin
> texture, no beauty filter. Soft diffused studio lighting, unretouched photography, sharp focus. Single subject only, exactly
> one person, no props, no furniture, no text, no watermark, no logos, no frame borders.

## A4. Locations — `soul_cinematic`, 16:9, 2k
One line of place, time of day, light and lens, plus "no people, no text, no logos".

## A5. Story clips — `seedance_2_5`, `omni_reference`, 1080p, 16:9, 8-12 s, `generate_audio: false`
`image_references`: faceboxed sheets + locations, as @Image1, @Image2... 4-6 sub-shots of 2 s
separated by `HARD CUT.`
> <style line> @Image1 is <character>, @Image2 is <character>, fully covered. Location: <place> from @Image3. Script line:
> '<the lines this clip illustrates>'.
> Shot 1 (2 s): <framing, action>.
> HARD CUT. Shot 2 (2 s): <framing, action>.
> HARD CUT. Shot 3 (2 s): ...
> Realistic, grounded, no blood. Only these people. No text, no logos, no music.

## A6. Host lip-sync clips — `seedance_2_5`, `omni_reference`, 1080p, 16:9, duration = chunk `gen_duration`, `generate_audio: true`; `image_references`: host portrait; `audio_references`: uploaded `work/host/host_NN.wav`. Upload each chunk via `media_upload` (`Content-Type: video/mp4` upload rule stays IE header-sensitive: PUT with `If-None-Match: *`), wait, download, save `assets/host/host_NN.mp4`, then `reel.py align <proj>`. CHECK → regenerate.
> The same person as in @Image1, exactly the same face and clothes, medium close-up, looking straight into the lens and talking
> to the viewer, small natural head movement and hand gestures below frame, plain softly lit wall behind. Her/his lips are
> perfectly synced to the speech in @Audio1: she/he says exactly those words in that voice, starting right at the beginning.
> Static camera. Photoreal. No text, no subtitles, no music.

## A7. Music — `elevenlabs_music_v2_5`, `force_instrumental: true`, duration = reel length + 2 s
> Instrumental cinematic documentary underscore, <N> seconds, sits under a voice-over. <mood>. Structure: 0:00 <hook: a deep
> sub boom and a short glitch, then a hushed build>; <m:ss> <drop>; <m:ss> <build>; <m:ss> <peak with low hits>; <m:ss> <calm
> resolving outro that fades out>. No vocals, no lead melody fighting the voice, no pop drum kit, clean mix.
