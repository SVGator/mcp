# Exporting projects

`export_project` turns a project into a deliverable. Six formats are supported, in two families:

| Format | Family | What you get |
| --- | --- | --- |
| `svg` | sync | Animated (or static) SVG for the web — JS- or CSS-driven |
| `lottie` | sync | Lottie JSON |
| `react-native` | sync | Mobile-wrapped source (React Native) |
| `flutter` | sync | Mobile-wrapped source (Flutter) |
| `video` | async raster | MP4 / MOV / AVI / MKV / WebM render |
| `gif` | async raster | GIF / WebP / PNG (sequence) / ZIP render |

Exports use the project's **saved export settings** by default — every parameter below is an
optional override. Exports count against your SVGator plan's limits, exactly as in the editor.

## Delivery: the three `kind`s

The result always carries a `kind` field:

- **`kind: "link"`** — the default for sync formats. The exported file is uploaded to a
  temporary **`content_url`** (valid ~1–2 days). This is token-light: the assistant hands you
  the link instead of pasting file contents into the conversation.
- **`kind: "content"`** — sync formats with `inline: true`. The file comes back inline in the
  `content` field (SVG markup, Lottie JSON, or framework source). Use when the assistant
  actually needs to read or embed the file.
- **`kind: "render"`** — always, for `video` and `gif`. Rendering is asynchronous: the response
  carries a `render_id`, and the assistant polls **`get_render`** until `status` is `"done"`,
  then fetches the **short-lived (~5 min) signed `download` URL**. Raster export requires the
  project to be animated ("Animate it first" otherwise).

```
video/gif:  export_project ──► render_id ──► get_render (poll) ──► status:"done" ──► download URL
```

## Export options

All optional. An option that does not apply to the chosen format/combination is rejected with a
validation error naming the exact constraint (exceptions: `player` and `font` are silently
ignored where inapplicable). "web/mobile" below means `svg`, `react-native`, and `flutter`.

### Animation playback

| Parameter | Values | Applies to | Notes |
| --- | --- | --- | --- |
| `iterations` | integer 0–999 | all except `lottie` | Loop count; `0` = infinite |
| `direction` | `normal` \| `reverse` | all except `lottie` | Pair with `alternate` for alternate-reverse |
| `alternate` | boolean | all except `lottie` | Ping-pong: reverse on every other iteration |
| `fill` | `forwards` \| `backwards` | all except `lottie` | `forwards` holds the last frame |
| `speed` | 0.01–100 | all | Playback rate multiplier (1 = normal) |
| `fps` | integer ≥ 1 | all | Max 100 for web/mobile/lottie, 60 for video, 30 for gif |

(`lottie` loop/direction settings are fixed server-side.)

### Output shape & interactivity (web/mobile)

| Parameter | Values | Applies to | Notes |
| --- | --- | --- | --- |
| `svgFormat` | `animated` \| `static` | web/mobile/lottie | `static` = single frame |
| `type` | `js` \| `css` | `svg` + `svgFormat=animated` | How the animation is driven |
| `start` | `load` \| `hover` \| `click` \| `scroll` \| `programmatic` | web/mobile + animated | Start trigger. `type=css` supports only `load`/`hover` |
| `click` | `freeze` \| `restart` \| `reverse` \| `none` | + `type=js` + `start=click` | Action on subsequent clicks |
| `hover` | `freeze` \| `reset` \| `reverse` \| `none` | + `start=hover` | `reverse`/`none` need `type=js` |
| `scroll` | integer 1–100 | + `type=js` + `start=scroll` | Scroll trigger offset (%) |
| `player` | `embed` \| `external` | web/mobile | Player script bundling (default external); ignored elsewhere |
| `font` | `embed` \| `external` \| `no-export` | web/mobile | Font bundling (default external); ignored elsewhere |
| `hyperlink` | URL string | `svg` only | Wraps the SVG in a link |

### Raster (video/gif)

| Parameter | Values | Applies to | Notes |
| --- | --- | --- | --- |
| `extension` | video: `mp4` \| `mov` \| `avi` \| `mkv` \| `webm`; gif: `gif` \| `webp` \| `png` \| `zip` | video/gif | Output container (defaults: mp4 / gif) |
| `quality` | `lowest` \| `lower` \| `low` \| `medium` \| `high` \| `higher` \| `highest` | video/gif | Encode quality |
| `resolution` | `{ width: 10–3840, height: 10–2160 }` | video/gif | Output pixel size |
| `keepTransparency` | boolean | gif, or video with `mov`/`webm` | Transparency-capable containers only |
| `dither` | boolean | gif + `extension=gif` | |
| `matte` | boolean | gif + `extension=gif` | Matte against the canvas colour |
| `colorPaletteSize` | integer 1–256 | gif + `extension=gif` | GIF palette size |

### Canvas & sizing

| Parameter | Values | Applies to | Notes |
| --- | --- | --- | --- |
| `responsive` | boolean | all | Responsive sizing; omit to keep the project's saved value |
| `bgColor` | `null` \| `{type:"none"}` \| `{type:"color", value:{r,g,b (0–255), a (0–1)}}` | all | Canvas background colour override; `null` clears it |
| `exportCanvasColor` | boolean | all | Bake the canvas background into the output — set it together with `bgColor` |

## Recipes

**Animated SVG that plays on hover and reverses when the pointer leaves:**

```json
{ "project_id": "pi_…", "format": "svg", "type": "js", "start": "hover", "hover": "reverse" }
```

**Infinite-loop hero animation:**

```json
{ "project_id": "pi_…", "format": "svg", "iterations": 0 }
```

**Transparent GIF at a fixed size:**

```json
{ "project_id": "pi_…", "format": "gif", "keepTransparency": true,
  "resolution": { "width": 800, "height": 600 } }
```

**Full-HD MP4 at 60 fps, highest quality:**

```json
{ "project_id": "pi_…", "format": "video", "extension": "mp4", "quality": "highest",
  "fps": 60, "resolution": { "width": 1920, "height": 1080 } }
```

## What export cannot do

`export_project` only overrides **export settings** (options, animation playback, `responsive`,
`bgColor`) — never the project's structure. There is no parameter for changing shapes, text,
colors, or animation content at export time: make content changes first with `edit_part` (or
`replace_project`), then export. See [Editing projects](editing.md).
