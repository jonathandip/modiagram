# modiagram
Draw molecular orbital and energy pathway diagrams: a package inspired by the LaTeX [`modiagram`](https://ctan.org/pkg/modiagram) package by Clemens Niederberger plus additional features for plotting energy pathway diagrams.

Requires [`@preview/cetz:0.4.2`](https://typst.app/universe/package/cetz).

---

## Quick start

```typst
#import "@preview/modiagram:0.1.0" as mo

#figure({
  import mo: *
  modiagram(
    ao(name: "1s-L", x: 0,    energy: 0,     electrons: "pair", label: $1s$),
    ao(name: "1s-R", x: 3.5,  energy: 0,     electrons: "pair", label: $1s$),
    ao(name: "S",    x: 1.75, energy: -0.65, electrons: "pair", label: $sigma$),
    ao(name: "S*",   x: 1.75, energy:  0.65, electrons: "",     label: $sigma^*$),
    connect("1s-L & S", "1s-R & S", "1s-L & S*", "1s-R & S*"),
    energy-axis(title: [Energy]),
  )
})
```

> **Tip — avoid name conflicts.** `modiagram` exports wrappers named `line`, `circle`, `rect`, `arc`, `grid`, and `content` that shadow Typst built-ins. Import inside a scoped block to keep names local:
> 
> ```typst
> #import "@preview/modiagram:0.1.0" as mo
> #figure({ import mo: *; modiagram(...) })
> ```
> 
> Alternatively use the `mo-line`, `mo-circle`, … aliases, or prefix Typst built-ins with `std.`.

---

## Import styles

```typst
// Recommended — scoped, no leakage
#import "@preview/modiagram:0.1.0" as mo
#figure({ import mo: *; modiagram(...) })

// Named module — explicit prefix
#import "@preview/modiagram:0.1.0" as mo
mo.modiagram(mo.ao(...), mo.connect(...))

// Star import — use mo-* aliases if needed
#import "@preview/modiagram:0.1.0": *
modiagram(ao(...), mo-line(...))   // mo-line avoids shadowing std line
```

---

## Reference

### `modiagram(..items, config: (:))`

Renders the complete diagram. All elements are passed as positional arguments.

|Parameter|Type|Description|
|---|---|---|
|`..items`|any|`ao()`, `connect()`, `config()`, `energy-axis()`, `en-pathway()`, `raw()`, cetz wrappers|
|`config`|dict|Diagram-level overrides — same keys as `modiagram-setup()`|

---

### `ao(...)` — atomic / molecular orbital

Draws one orbital bar with optional electrons and label.

|Parameter|Default|Description|
|---|---|---|
|`name`|auto|String identifier used by `connect()`|
|`x`|`0`|Horizontal position (float = cm, or any Typst length)|
|`energy`|`0`|Vertical position (energy level)|
|`electrons`|`""`|Space-separated spin tokens: `"up"`, `"down"`, `"pair"`|
|`label`|`none`|Content placed below the bar, e.g. `$1s$`|
|`color`|`black`|Base color for bar, electrons, and label|
|`bar-color`|`auto`|Bar color only (overrides `color`)|
|`el-color`|`auto`|Electron arrow color (overrides `color`)|
|`label-color`|`auto`|Label color (overrides `color`)|
|`style`|`auto`|Per-orbital style (see **Orbital styles** below)|
|`label-size`|`auto`|Font size for the label|
|`label-gap`|`auto`|Gap between bar and label (cm)|
|`el-stroke-w`|`auto`|Electron arrow stroke width|
|`bar-stroke-w`|`auto`|Bar stroke width|
|`ao-width`|`auto`|Bar width (cm)|
|`up-el-pos`|`auto`|X offset from bar center for ↑ electron|
|`down-el-pos`|`auto`|X offset from bar center for ↓ electron|

**Electron tokens** (space-separated in the `electrons` string):

|Token|Effect|
|---|---|
|`"up"`|↑ one spin-up|
|`"down"`|↓ one spin-down|
|`"pair"`|↑↓ paired electrons|


---

### `connect(..pairs, ...)` — connection lines

Draws lines between pairs of orbitals.

```typst
connect("1s-L & S", "1s-R & S", style: "dashed", color: blue)
```

|Parameter|Default|Description|
|---|---|---|
|`..pairs`|—|One or more `"nameA & nameB"` strings|
|`style`|`auto`|Line style (see **Connection styles**)|
|`color`|`auto`|Line color|
|`thickness`|`auto`|Stroke width|
|`gap`|`auto`|Dot/dash spacing|
|`dash-length`|`auto`|Dash segment length|

---

### `connect-label(a, b, body, ...)` — label along a connection

Places content along the line between two orbitals.

```typst
connect-label("a", "σ", $Delta E$, ratio: 50%, pad: 0.15, anchor: "south")
```

|Parameter|Default|Description|
|---|---|---|
|`a`, `b`|—|Orbital names (must match a `connect()` pair)|
|`body`|—|Any Typst content|
|`ratio`|`50%`|Position along the line|
|`pad`|`0`|Perpendicular offset (positive = above)|
|`size`|`auto`|Font size (scales text and math)|

---

### `energy-axis(...)` — energy arrow

Draws a vertical (or horizontal) energy arrow at the left of the diagram.

```typst
energy-axis(title: [Energy], style: "vertical")
energy-axis(title: "E", padding: 0.7cm, style: "horizontal")
```

|Parameter|Default|Description|
|---|---|---|
|`title`|`none`|Content near the arrowhead|
|`padding`|`0.5`|Gap from leftmost orbital edge (cm)|
|`style`|`"vertical"`|`"vertical"` or `"horizontal"`|

---

### `config(...)` — inline diagram override

Applies settings to all subsequent elements within the same `modiagram()` call. Pass `auto` to reset a key to the diagram default.

```typst
modiagram(
  config(color: red, style: "square"),
  ao(...),                           // red, square
  config(color: auto),               // reset color
  ao(...),                           // black, square
)
```

Accepted keys: `color`, `bar-color`, `el-color`, `label-color`, `style`, `label-size`, `label-gap`, `el-stroke-w`, `bar-stroke-w`, `ao-width`, `conn-style`, `up-el-pos`, `down-el-pos`, `x-scale`, `energy-scale`.

---

### `en-pathway(..energies, ...)` — evenly-spaced orbital sequence

Generates a sequence of orbitals at uniform horizontal spacing, connected by lines. Useful for energy level diagrams (e.g. aufbau principle, reaction pathways).

```typst
en-pathway(0, 0.5, 1.0, skip, 1.5,
  color: blue, labels: ($1s$, $2s$, $2p$, $3s$),
  conn-style: "dashed", show-energies: true)
```

Use `skip` as an energy value to advance `x` without drawing an orbital.

|Parameter|Default|Description|
|---|---|---|
|`..energies`|—|Energy values (float, int, length, or numeric string), or `skip`|
|`color`|`black`|Color for bars, electrons, labels, connections|
|`x-step`|`1.2`|Horizontal distance between orbitals (cm)|
|`electrons`|`""`|Electron string applied to all orbitals|
|`style`|`auto`|Orbital style|
|`conn-style`|`"dashed"`|Connection style between adjacent orbitals (`none` to disable)|
|`labels`|`none`|Array of content, one per orbital|
|`name-prefix`|`"ep"`|Prefix for auto-generated orbital names|
|`x-start`|`0`|Starting x position|
|`show-energies`|`false`|Show energy value above each orbital|
|`energy-format`|`auto`|Function `v => content` for custom energy display|
|`energy-size`|`auto`|Font size for energy labels|
|`bar-stroke-w`|`1.5pt`|Stroke width for bars and electrons|
|`ao-width`|`0.75`|Bar width (cm)|

---

### `raw(closure)` — arbitrary cetz drawing

Executes raw cetz `draw.*` calls inside the diagram canvas.

```typst
raw((xs, ys, anchors) => {
  let p = at("σ", anchors, edge: "right")
  draw.content((p.at(0) + 0.2, p.at(1)), [bond], anchor: "west")
})
```

Closure signature: `(xs, ys, anchors) => { ... }`

- `xs`, `ys` — active x-scale and energy-scale
- `anchors` — dict of all orbital anchors; query with `at()`

---

## Position forms

All cetz wrappers accept these position forms:

|Form|Example|Description|
|---|---|---|
|`(x, y)`|`(1.5, 0.3)`|Numeric tuple, scaled by x-scale / energy-scale|
|`"name"`|`"σ"`|Center of the named orbital|
|`"name.edge"`|`"σ.right"`|Named edge: `left`, `right`, `top`, `bottom`, `center`|
|`("a", ratio%, "b")`|`("a", 50%, "b")`|Linear interpolation between two positions|
|`rel(dx, dy)`|`rel(1, 0)`|Relative offset from the previous point|

Extra keyword arguments available on all wrappers: `pad`, `dx`, `dy`.

---

## Orbital styles

|Style|Description|Default connection|
|---|---|---|
|`"plain"`|Horizontal line only (default)|dotted|
|`"square"`|Square box|solid|
|`"round"`|Square box with rounded corners|solid|
|`"circle"`|Circle with side extensions|solid|
|`"fancy"`|Rounded box with longer side extensions|dashed|

---

## Connection styles

|Style|Appearance|
|---|---|
|`"dotted"`|Dotted line|
|`"dashed"`|Dashed line|
|`"solid"`|Solid line|
|`"gray"`|Solid gray line|

---

## Global configuration

**`modiagram-setup(...)`** — set document-level defaults for all `modiagram()` calls:

```typst
#modiagram-setup(style: "square", scale: 1.2cm, label-size: 6pt)
```

**`en-pathway-setup(...)`** — set document-level defaults for all `en-pathway()` calls:

```typst
#en-pathway-setup(conn-style: "solid", show-energies: true, color: blue)
```

---

## Helper functions

|Function|Description|
|---|---|
|`at(name, anchors, edge: "center")`|Resolve a named orbital to `(x, y)` — use inside `raw()`|
|`sp(pt, xs, ys)`|Manually scale a `(x, y)` pair — use inside `raw()`|
|`rel(dx, dy)`|Relative offset sentinel for multi-point wrappers|
|`skip`|Skip sentinel for `en-pathway()`|

---

## Cetz wrappers

All standard cetz primitives are available as first-class diagram elements with automatic position resolution and `rel()` support:

`line` · `content` · `circle` · `rect` · `arc` · `grid` · `bezier` · `catmull` · `hobby` · `mark` · `merge-path` · `set-style` · `on-layer` · `group` · `hide` · `intersections` · `copy-anchors`

Collision-free aliases (for star imports): `mo-line`, `mo-circle`, `mo-rect`, `mo-arc`, `mo-grid`, `mo-content`.

---

## Priority chain

Settings are resolved in this order (first match wins):

```
per-element parameter
  → config() inline override
    → modiagram(config:) dict
      → modiagram-setup() global
        → built-in defaults
```

---

## License

MIT — see LICENSE for details.

Inspired by the LaTeX `modiagram` package by Clemens Niederberger (v0.3a).
