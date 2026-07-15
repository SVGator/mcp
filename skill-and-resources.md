# The motion-design skill, resources & prompt

Beyond the tools, the server ships a **motion-design craft skill** — design, animation, and
interaction direction layered on top of the raw authoring tools — plus a structural reference
resource. The goal: any MCP client gets an assistant that art-directs an animation (personality,
timing, layering) instead of shipping generic drawings and raw, un-eased tweens.

The skill is delivered through three standard MCP mechanisms, so it works in any client:

1. **Server `instructions`** — a compact router injected at `initialize`. It primes the model to
   apply craft when *designing, animating, or making something interactive*, and to ignore the
   guidance for pure export/inspection/listing calls. It also points to the deeper resources
   below, to be read on demand.
2. **Resources** — the full craft library, 24 markdown files under
   `skill://svgator-motion-design/…`. Clients that support MCP resources can list and read them;
   the router tells the model which file to pull for the task at hand.
3. **A prompt** — `svgator-motion-design`, an explicit entry point that returns the full skill
   brief (in Claude Code it appears as the `/svgator-motion-design` slash command). Optional
   argument `brief`: the animation request to art-direct, e.g. `"animate this logo reveal"`.

## The craft library (resources)

All resources are markdown (`text/markdown`) under `skill://svgator-motion-design/<path>`.
Relative links between the files mirror the folder tree.

| Resource path | Covers | Read when |
| --- | --- | --- |
| `SKILL.md` | Router / the full craft brief | Entry point |
| `design/composition.md` | Hierarchy, squint test, negative space, asymmetry, optical alignment | Designing artwork from scratch |
| `design/color.md` | Committed palettes, OKLCH thinking, tinted neutrals, banned AI palettes | Choosing a palette for a from-scratch piece |
| `design/form-and-type.md` | Shape language, materiality, anti-slop tells, display lettering | Finishing marks + lettering before motion |
| `director/core-philosophy.md` | Layers, attention budget, the anti-flat rules | Composing any non-trivial piece |
| `director/decision-framework.md` | Purpose → personality → direction → move | Unsure where to start |
| `director/disney-principles.md` | The twelve principles, mapped to SVGator channels | Squash/stretch, anticipation, follow-through |
| `director/motion-personality.md` | The four archetypes + a brand motion signature | Choosing or holding a personality |
| `director/emotion-mapping.md` | Feeling → curve, pace, path, color | Setting the emotional target |
| `director/choreography.md` | Coordinating a cast of elements | Scenes, staggers, ensembles |
| `director/narrative-structure.md` | Setup → action → resolution | Structuring a beat or sequence |
| `director/context-adaptation.md` | Embed, loop, reduced-motion, export target | Deciding amplitude/format for where the piece ships |
| `director/interactivity.md` | Motion that answers input: frequency, purpose, affordance, feedback, feel | The piece reacts to input (craft) |
| `patterns/entrance-exit.md` | Intro/outro recipes | Reveals and exits |
| `patterns/state-feedback.md` | Success / error / loader moments | Self-playing feedback pieces |
| `patterns/ambient-continuous.md` | Breathe, float, spin, shimmer, particles | Loops and living accents |
| `patterns/multi-element.md` | Stagger + scene choreography | Many elements moving at once |
| `patterns/signature-effects.md` | Standout hooks | A piece needs a signature move |
| `reference/output-types.md` | The piece-type playbook | Right after naming the piece |
| `reference/property-meaning.md` | What each channel connotes | Choosing the carrying channel |
| `reference/timing-easing.md` | Durations, curves, the overshoot trick | Dialing timing + easing |
| `reference/interactivity.md` | SVGator triggers, actions, and the Player API | Wiring a trigger or Player-API control (delivery) |
| `reference/quality-checklist.md` | Approval gate + review ritual | Before declaring an animation done |
| `reference/troubleshooting.md` | "Moves but looks wrong" fixes | Something reads off |

## The structural reference resource

| Resource | Type | Covers |
| --- | --- | --- |
| `svgator://reference/element-shapes` | `application/json` | The full element & animation shape catalog — element types, animator channels, keyframes, easings, paint. The same data the [`describe_shape`](tools.md#describe_shape) tool serves per-topic, published as one document for clients that prefer to load it once |

## For client/integrator authors

- If your client surfaces the server's `instructions` to the model, motion-design craft is
  applied automatically — no setup needed.
- If it supports **resources**, the model can follow the router's `skill://…` pointers and read
  depth on demand. Resource URIs mirror the folder tree, so the files' relative markdown links
  resolve against each resource's own URI; clients that don't resolve relative links can fall
  back to constructing `skill://svgator-motion-design/<path>` directly.
- If it supports **prompts**, `svgator-motion-design` is the explicit "art-direct this" entry
  point — useful when you want the full brief up front instead of relying on the model to
  self-trigger from `instructions`.
