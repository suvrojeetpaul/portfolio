> [!WARNING]
> Example websites that use CursorSwarm are for demonstration purposes only. They may contain copyrighted content and are not affiliated with the original creators.
> Content of example websites are generated with AI tools based on the website https://zll.me style and color palette.

# CursorSwarm

A swarm of "agent" cursors that **builds your web page in front of the visitor** as they scroll: they draw the boxes, type the headings, write the paragraphs and scan the images into place. Scroll back up and they take it apart again.

Inspired by [zll.me](https://zll.me) and [Swarm](https://califa.github.io/swarm-site/). Zero dependencies, ~1 file, works on any existing site.

```html
<script src="cursor-swarm.js"></script>
<script>
  new CursorSwarm({ cursors: 3 }).init();
</script>
```

That's it. CursorSwarm scans the page, splits it into buildable pieces and hands them out to the agents.

## Zero-JS setup

Put the script in `<head>` with `data-auto`. Each `data-*` attribute is an option (kebab-case to camelCase). The page stays hidden until the agents are ready, so nothing flashes.

```html
<script src="cursor-swarm.js" data-auto
        data-cursors="4"
        data-start-position="corners"
        data-exit-after-build="home"
        data-colors="#00ff9c,#ff2e88,#ffe600,#2ee6ff"></script>
```

The instance is exposed as `window.cursorSwarm`.

## How it works

Every element in the page gets one of five **build types**, detected automatically:

| Type    | Detected on                                                     | Build animation                                   | Agent role |
|---------|-----------------------------------------------------------------|---------------------------------------------------|------------|
| `frame` | boxes with a background, border, shadow or gradient (cards…)    | marquee + border lines drawn, then the box fades in | layout     |
| `pop`   | buttons, inputs, pills, small icons                             | the cursor clicks it into existence               | layout     |
| `type`  | headings and big text (≤140 chars)                              | typed letter by letter with a caret               | type       |
| `words` | paragraphs, list items, captions, links…                        | written word by word                              | type       |
| `media` | `img`, `video`, `svg`, `canvas`, `iframe`, background images    | marquee + scan line wipe                          | media      |

Each agent prefers jobs of its role. If an agent falls behind, others help. When the visitor scrolls fast, the whole swarm speeds up (`adaptiveSpeed`). Anything still queued that is already off screen is built instantly. Text inside a box waits until its box is drawn.

## Options

```js
new CursorSwarm({
  cursors: 3,                 // number of agents
  agents: null,               // per-agent overrides: [{ name, color, role, x, y }]
  names: null,                // ['EDITOR', 'LAYOUT'] or i => `BOT ${i}`
  palette: 'default',         // see "Palettes" below. This is so COOL!
  colors: null,               // explicit colors, override the palette's (cycled across agents)
  startPosition: 'center',    // 'center' | 'random' | 'corners' | 'top' | 'bottom' | 'left' | 'right'
                              // | 'outside' (fly in) | { x, y } | [{ x, y }, …] | (i, n) => ({ x, y })
                              // x / y accept px numbers or '50%' strings
  root: null,                 // element or selector to build (default: document.body)
  targets: null,              // selector: only build these elements
  ignore: null,               // selector: never touch these
  frames: true,               // draw boxes
  deconstruct: true,          // un-build when scrolling back up
  exitAfterBuild: false,      // true / 'edge': leave the screen when idle · 'home': go back to start position
  idle: 'dim',                // idle agent look: 'dim' | 'stay' | 'hide'
  spawn: true,                // spawn animation on init
  speed: 1,                   // global speed multiplier
  adaptiveSpeed: true,        // speed up when the backlog grows
  maxSpeed: 4,
  trigger: 0.88,              // build when the element's top crosses 88% of the viewport
  typeDuration: 1.6,          // max seconds spent typing one block
  hud: true,                  // status pill: current job + build progress
  hudPosition: 'bottom-left', // 'bottom-left' | 'bottom-right' | 'top-left' | 'top-right'
  labels: true,               // name tag next to each cursor
  marquee: true,              // selection box while measuring
  guides: true,               // crosshair guide lines
  sparks: true,               // particles when an element is done
  caret: true,                // blinking caret while typing
  copyPaste: false,           // true (50%) | 0..1: copy repeated elements from a built sibling (see below)
  relocation: false,          // true (30%) | 0..1: build some elements out of place, then drag them home
  zIndex: 2147483000,
  respectReducedMotion: true, // do nothing for prefers-reduced-motion users

  onBuild(el, type, agentName) {},
  onUnbuild(el, type, agentName) {},
  onIdle(agent) {},
  onDone() {}                 // every element is built
});
```

## Methods

| Method             | What it does                                                               |
|--------------------|----------------------------------------------------------------------------|
| `init()`           | Scan the page, spawn the agents and start building. Returns the instance.  |
| `pause()`          | Freeze every cursor and animation where it is.                             |
| `resume()`         | Continue after `pause()`.                                                  |
| `toggle()`         | Pause or resume.                                                           |
| `stop({ reveal })` | Stop the swarm. By default (`reveal: true`) everything is shown instantly. |
| `destroy()`        | Stop and restore the DOM exactly as it was before `init()`.                |
| `buildAll()`       | Instantly build everything, keep the swarm running.                        |
| `refresh()`        | Pick up elements added to the page after `init()`.                         |
| `add(el, type?)`   | Register one element, optionally forcing its type.                         |
| `setSpeed(n)`      | Change the speed multiplier live.                                          |
| `setPalette(p)`    | Recolor the agents and the HUD live (same values as the `palette` option). |
| `on(ev, fn)` / `off(ev, fn)` | Events: `start`, `build`, `unbuild`, `idle`, `done`, `pause`, `resume`, `stop`, `destroy`, `palette`. |

Properties: `state` (`idle`, `running`, `paused`, `stopped`, `destroyed`), `progress` (0 to 1), `colors` (current agent colors), `agents`, `records`.

## Copy-paste and relocation

Two optional behaviors that make the agents look like they are really editing the page:

```js
new CursorSwarm({
  copyPaste: true,   // or a probability: 0.7
  relocation: true   // or a probability: 0.5
}).init();
```

**`copyPaste`**: when an agent reaches a repeated element (a card in a grid, a pricing plan, a chip, a photo next to others), it may skip building it from scratch. Instead it selects an identical sibling that is already built (`⌘C`/`Ctrl+C`), drags a copy over to the new spot while resizing it, pastes it (`⌘V`), and then edits it into the real content. Siblings count as repeated when they have the same parent, tag and classes. `true` means a 50% chance per element.

**`relocation`**: some elements are created slightly out of place (shifted and tilted, with a dashed outline marking where they belong). Once built, the agent grabs them and moves them into position. `true` means a 30% chance. Very large, `fixed` and `sticky` elements are never moved.

Force either behavior per element:

```html
<div class="card" data-cs-copy="true">…</div>      <!-- always copy-pasted when a built twin is on screen -->
<div class="card" data-cs-copy="false">…</div>     <!-- never -->
<img data-cs-relocate="true" …>                     <!-- always built out of place -->
<h2 data-cs-relocate="false">…</h2>                 <!-- never -->
```

## Palettes

`palette` sets the agents' colors (and optionally the HUD's look). It accepts:

```js
palette: 'neon'                          // a preset
palette: 'mono:#ff5e00'                  // shades of one color (one per agent)
palette: '#ff5e00'                       // same as 'mono:#ff5e00'
palette: ['#111', '#555', '#999']        // a list of colors
palette: '#111,#555,#999'                // same, handy in data-palette="…"
palette: {                               // full control
  colors: ['#5b5bf7', '#ff5e8a'],        // or mono: '#5b5bf7'
  hud: { background: '#fff', color: '#111', mute: '#666', border: 'rgba(0,0,0,.1)' }
}
```

Presets: `default`, `neon`, `pastel`, `sunset`, `ocean`, `forest`, `candy`, `mono` (grays) and `ink` (dark cursors and a light HUD, for white pages). Mono palettes also tint the HUD with the same hue. Name tags and cursor outlines switch between black and white automatically so they stay readable on any color.

Switch it at any time, with no restart:

```js
swarm.setPalette('ocean');
swarm.setPalette('mono:' + picker.value);
```

Register your own preset, or use the helpers:

```js
CursorSwarm.palettes.brand = { colors: ['#5b5bf7', '#ff5e8a', '#00c2a8'] };
new CursorSwarm({ palette: 'brand' }).init();

CursorSwarm.mono('#1e90ff', 4);  // the 4 shades a mono palette would use
CursorSwarm.inkOn('#1e90ff');    // '#000' or '#fff', whichever reads better on it
```

An agent with its own `color` in `agents: [...]` keeps it when the palette changes.

## HTML attributes

Fine-tune detection directly in your markup:

```html
<div data-cs-ignore>…</div>          <!-- never touched (including its children) -->
<div data-cs="skip">…</div>          <!-- same as data-cs-ignore -->
<div data-cs="none">…</div>          <!-- not built itself, children still are -->
<h2 data-cs="words">…</h2>           <!-- force a type: frame | pop | type | words | media -->
<img data-cs-dir="t" …>              <!-- media scan direction: l | r | t -->
<p data-cs-label="intro">…</p>       <!-- name shown in the HUD -->
<div data-cs-copy="true">…</div>     <!-- force / prevent copy-paste: true | false -->
<img data-cs-relocate="true" …>      <!-- force / prevent relocation: true | false -->
```

## Avoiding a flash of content

With `data-auto`, the page is hidden until the swarm is ready. In manual mode, call `init()` as early as possible (a script at the end of `<body>` is fine). If you load the script in `<head>`, call `CursorSwarm.cloak()` right away and `init()` on `DOMContentLoaded`:

```html
<head>
  <script src="cursor-swarm.js"></script>
  <script>
    CursorSwarm.cloak();
    addEventListener('DOMContentLoaded', () => new CursorSwarm().init());
  </script>
</head>
```

## Examples

Open [`examples/index.html`](examples/index.html), or serve the folder:

```bash
python3 -m http.server 8787
```

Then visit http://localhost:8787/examples/.

- **saas-landing**: 3 agents, brand colors, deconstruct, copy-paste cards
- **blog-article**: 2 named agents that fly in from outside and leave after building
- **photo-portfolio**: 6 agents, random start, hidden when idle, relocation + copy-paste
- **restaurant**: 1 agent with a custom start position, no HUD, builds once
- **zero-js**: only `data-*` attributes
- **monochrome**: a site built from one base color; the toolbar recolors the site and the swarm live with `setPalette()`
- **playground**: every option live (palettes included), plus pause / resume / stop / destroy / refresh

**BUILD BY AI**: The content of the example websites is generated with AI tools based on the website https://zll.me style and color palette. They are for demonstration purposes only and may contain copyrighted content. They are not affiliated with the original creators.

## Notes

- Text is split into `<span>`s so it can be typed. `destroy()` puts the original text nodes back.
- Visitors with `prefers-reduced-motion` get the plain page (disable with `respectReducedMotion: false`).
- Content is never removed: an element that is off screen when its turn comes is simply shown.
- Works in every modern browser (uses `IntersectionObserver`, `clip-path` and the `scale` CSS property).

## License

MIT #OpenSource
