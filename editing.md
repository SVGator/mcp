# Editing projects

The server edits projects through a **surgical, validated loop** rather than shipping whole
project files back and forth. This keeps token usage low, makes every change verifiable, and
means a bad edit is rejected server-side with a descriptive error instead of corrupting the
project.

## The editing loop

```
1. get_skeleton   → map the structure, find the element by id, see what's animated
2. read_part      → read the current value at an exact path
3. edit_part      → apply ONE validated edit
4. verify         → check the returned `element`, or request preview="image"
```

For **building a whole document from scratch** (or a bulk transformation), the coarse-write tools
`create_project` / `replace_project` take a full project JSON instead — see
[When to use which write](#when-to-use-which-write).

## Path addressing

Every read and edit targets an **id-anchored path**. Element ids come from `get_skeleton` —
always address by `id`, never by guessing array indices.

| Path | Targets |
| --- | --- |
| `#rect-abc123` | The whole element |
| `#rect-abc123/properties/fill` | The element's fill property group |
| `#rect-abc123/properties/fill/paint` | A leaf value (here: the static fill paint) |
| `#rect-abc123/animators/fill/paint` | A whole animator channel (group `fill`, channel `paint`) |
| `#rect-abc123/animators/fill/paint/keys/2/value` | The 3rd keyframe's value |
| `#rect-abc123/animators/transform/origin/keys/0/time` | The first origin keyframe's time |

The animator segment is always **`animators/<group>/<channel>`** with group one of `transform`,
`fill`, `stroke`, `compositing`, `filter`, `shape`. Keyframe indices are zero-based and
time-ordered; `get_skeleton`'s per-channel keyframe counts tell you the valid index range (a
count of 3 means indices `0..2`).

After a `read_part`, copy the returned **`resolvedItem`** verbatim as the edit target — it is the
canonical form of the path you asked for.

## The one rule that prevents invisible edits: `itemAnimated`

A static property that is **animated is shadowed by its animation** — writing the static value
changes nothing visible. Both `read_part` and `edit_part` surface `itemAnimated` so this is
always detectable:

- `itemAnimated: false` → edit the static property (`…/properties/…`).
- `itemAnimated: true` → edit the **keyframe** instead:
  `…/animators/<group>/<channel>/keys/<i>/value`.

One convenience: after editing an animator's **keyframe 0**, the server re-syncs the matching
static property automatically — don't also write it.

## Inserting, moving, deleting

- **Insert** a new element with `action=before|after|append|prepend` and the element subtree as
  `value`. `append`/`prepend` are the only way to populate an empty container.
- **The server assigns/remaps ids on insert.** Address the new node by the returned
  `insertedItem` (and `idMap` for the whole subtree) — never by the id you sent.
- **Move** with `action=move`, `target=<element id>`, `position=before|after|append|prepend`.
  The element keeps its id and animators.
- **Delete** an element, a whole animator channel, or a single keyframe (`…/keys/<i>`) with
  `action=delete` — no `value` needed.

To add keyframes, gradient stops, or path points, `update` the surrounding value (e.g. rewrite
the channel's `keys` array) — there is no per-keyframe insert action.

## Authoring elements: get the shapes right first

Before authoring any element or animation JSON (for an insert, or for
`create_project`/`replace_project`), call **`describe_shape`** for each element type and concept
involved (`rect`, `path`, `keyframe`, `easing`, `paint`, …). It returns a copy-ready `shape`
example plus the element's actual property groups and animatable channels, so field names are
copied, not guessed.

Non-negotiable authoring rules:

- **Animators are nested** — `animators.<group>.<channel>`, e.g.
  `{"fill": {"opacity": {"keys": […]}}}`. Never a dotted key like `{"fill.opacity": …}`.
- **Property groups are per-element, not universal** — e.g. `image`/`g`/`a` carry no
  fill/stroke; `svg`/`defs` carry none. `describe_shape` lists the groups each element has.
- **Give every element a `title`** (a short human-readable name like `"Ball"`, `"Shadow"`) so it
  is identifiable in the editor and in `get_skeleton`.
- Easing is per-keyframe (bezier); keyframe `keys` arrays are time-ordered.

## Verifying an edit

Every successful `edit_part` returns `element` — the affected node **as finally stored**
(server-assigned ids, keyframe-0-synced baseline, preserved children), so most edits need no
follow-up read. For visual verification, pass `preview="image"` to get a rendered static SVG
string; `preview="module"` additionally returns the animation payload (opt-in — it can be
~400 KB).

## When to use which write

| Situation | Tool |
| --- | --- |
| Change a color, a value, a keyframe, one element | `edit_part` (one edit per call) |
| Add/remove/move elements | `edit_part` with an insert/`delete`/`move` action |
| Build a new document you've assembled | `create_project` |
| Clone or template a project | `create_project` |
| Bulk-transform / redo a whole document | `replace_project` |

The coarse writes validate the **whole project as one unit** — a single wrong field fails the
entire call (nothing is persisted), which is why `describe_shape`-first matters. Also note the
`options` caution: a **stored** project nests every format under
`options.export.settings.<format>`; this is *not* the flattened parameter shape
`export_project` accepts, so never copy export-tool parameters into a project's `options`.

## Error handling

The server is the validation authority. Invalid edits return a descriptive `422` naming exactly
what's wrong (unknown field, bad enum value, malformed keyframe, …) — the intended workflow is:
read the message, fix the payload, retry. Nothing is persisted on a rejected edit or replace.
