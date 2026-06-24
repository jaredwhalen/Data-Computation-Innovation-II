# **Interactivity and Animation** — Code Reference

---

## **`$state`**

Reactive state in Svelte 5 is declared with `$state`. When the value changes, anything that depends on it re-renders automatically. You don't call a setter or trigger an update — you just reassign.

```svelte
<script>
  let count = $state(0)
</script>

<p>{count}</p>
<button onclick={() => count++}>Increment</button>
```

---

## **Events**

Svelte uses standard DOM event attributes on HTML and SVG elements alike. The handler is a function — reassign state inside it and the component updates.

```svelte
<script>
  let active = $state(false)
</script>

<rect
  width="100" height="60" x="50" y="50"
  fill={active ? "tomato" : "steelblue"}
  onclick={() => active = !active}
/>
```

Common events in chart work: `onclick`, `onmouseenter`, `onmouseleave`, `onmousemove`, `oninput`.

---

## **`$derived`**

When a value is computed from state, use `$derived`. It recalculates automatically when its dependencies change.

```svelte
<script>
  let threshold = $state(50)

  const data = [
    { label: "A", value: 30 },
    { label: "B", value: 70 },
    { label: "C", value: 45 },
    { label: "D", value: 90 }
  ]

  let filtered = $derived(data.filter(d => d.value >= threshold))
</script>

<label>
  Min value: {threshold}
  <input type="range" min="0" max="100" bind:value={threshold} />
</label>

{#each filtered as d}
  <p>{d.label}: {d.value}</p>
{/each}
```

Use `$derived` for filtered datasets and for scales or generators that depend on state. Scales that don't depend on state stay `const`.

---

## **`bind:value`**

Two-way binding for form inputs. Wires an input's value directly to a state variable without writing an event handler.

```svelte
<script>
  let threshold = $state(50)
</script>

<!-- with bind -->
<input type="range" min="0" max="100" bind:value={threshold} />

<!-- equivalent without bind -->
<input
  type="range" min="0" max="100"
  value={threshold}
  oninput={(e) => threshold = Number(e.target.value)}
/>
```

---

## **`bind:clientWidth`**

Svelte can bind element dimensions. `bind:clientWidth` reads the rendered pixel width of any element into a state variable. When the browser resizes, the binding updates and anything that depends on it recalculates.

```svelte
<script>
  import * as d3 from 'd3'

  let containerWidth = $state(0)

  const height = 300
  const margin = { top: 20, right: 20, bottom: 40, left: 50 }

  let xScale = $derived(
    d3.scalePoint()
      .domain(data.map(d => d.month))
      .range([margin.left, containerWidth - margin.right])
  )
</script>

<div bind:clientWidth={containerWidth}>
  {#if containerWidth > 0}
    <svg width={containerWidth} {height}>
      <!-- chart -->
    </svg>
  {/if}
</div>
```

The `{#if containerWidth > 0}` guard prevents a zero-width render on the initial mount before the binding has fired.

---

## **Reactive scales**

When a scale depends on state, declare it with `$derived`. Scales that don't depend on state stay `const`.

```svelte
<script>
  import * as d3 from 'd3'

  let metric = $state("temp")

  const data = [
    { month: "Jan", temp: 34, precip: 3.2 },
    { month: "Feb", temp: 37, precip: 2.8 },
    // ...
  ]

  // x doesn't depend on state — stays const
  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([40, 560])
    .padding(0.5)

  // y depends on metric — needs $derived
  let yScale = $derived(
    d3.scaleLinear()
      .domain([0, d3.max(data, d => d[metric])])
      .range([280, 20])
      .nice()
  )

  let lineGenerator = $derived(
    d3.line()
      .x(d => xScale(d.month))
      .y(d => yScale(d[metric]))
  )
</script>
```

`d[metric]` is JavaScript bracket notation — when `metric` is `"temp"`, `d[metric]` is the same as `d.temp`. This lets one scale and one generator serve multiple metrics without duplication.

---

## **Tooltips**

A tooltip is a conditional SVG element positioned with reactive state.

```svelte
<script>
  let tooltip = $state(null)
</script>

{#each data as d}
  <rect
    ...
    onmouseenter={() => {
      tooltip = {
        x: xScale(d.label) + xScale.bandwidth() / 2,
        y: yScale(d.value) - 8,
        text: `${d.label}: ${d.value}`
      }
    }}
    onmouseleave={() => tooltip = null}
  />
{/each}

{#if tooltip}
  <text
    x={tooltip.x}
    y={tooltip.y}
    text-anchor="middle"
    dominant-baseline="auto"
  >{tooltip.text}</text>
{/if}
```

`tooltip` starts as `null`. On `onmouseenter` it becomes an object with position and text. The `{#if}` renders only when it has a value. On `onmouseleave` it clears.

---

## **CSS transitions on SVG**

CSS transitions animate property changes on persistent elements. Declare them in `<style>` — no JavaScript required.

```svelte
<style>
  rect {
    transition: fill 200ms ease, opacity 200ms ease;
  }
</style>
```

Any time `fill` or `opacity` changes on a `rect`, the browser interpolates between the old and new values over 200ms.

**Works:** `fill`, `stroke`, `opacity`, `transform`, `r` (circle radius, most browsers), `d` (path string, modern browsers)

**Doesn't work directly:** `x`, `y`, `width`, `height` as SVG presentation attributes. Use `transform: translateY()` on a wrapping `<g>` instead, which does transition.

---

## **Animating a line path**

The `d` attribute of a `<path>` can be transitioned in modern browsers. When the line generator recalculates, the browser interpolates between the old and new shapes.

```svelte
<path
  d={lineGenerator(data)}
  fill="none"
  stroke="steelblue"
  stroke-width="2.5"
/>
```

```svelte
<style>
  path {
    transition: d 300ms ease;
  }
</style>
```

---

## **Animating axis ticks**

SVG `y` attributes can't be CSS-transitioned. The workaround: position each tick group with `transform: translateY()` on a wrapping `<g>` and put children at `y="0"`.

```svelte
<script>
  import { fade } from 'svelte/transition'
</script>

{#each yScale.ticks(5) as tick (tick)}
  <g
    style="transform: translateY({yScale(tick)}px)"
    transition:fade={{ duration: 150 }}
  >
    <line x1={margin.left} x2={width - margin.right} y1="0" y2="0" stroke="#e5e5e5" />
    <text x={margin.left - 8} y="0" text-anchor="end" dominant-baseline="middle">{tick}</text>
  </g>
{/each}
```

```svelte
<style>
  g {
    transition: transform 300ms ease;
  }
</style>
```

The `(tick)` key lets Svelte track each gridline individually. Ticks that disappear fade out via `transition:fade`. Ticks that survive slide to their new position via the CSS transform transition.

---

## **Svelte transitions**

Svelte's built-in transitions animate elements entering and leaving the DOM. Import from `svelte/transition`.

```svelte
<script>
  import { fade, fly } from 'svelte/transition'

  let show = $state(false)
</script>

<button onclick={() => show = !show}>Toggle</button>

{#if show}
  <div transition:fade={{ duration: 300 }}>
    Fades in and out.
  </div>

  <div
    in:fly={{ y: 20, duration: 400 }}
    out:fade={{ duration: 200 }}
  >
    Flies in from below, fades out.
  </div>
{/if}
```

`transition:` applies the same animation on enter and exit. `in:` and `out:` let you specify different behavior for each direction.

---

## **Keyed `{#each}` for enter/exit transitions**

By default Svelte reuses DOM nodes when a list changes. Adding a key forces Svelte to track items individually, enabling enter and exit transitions when items are added or removed.

```svelte
{#each data as d (d.label)}
  <g transition:fade={{ duration: 150 }}>
    <rect ... />
  </g>
{/each}
```

The `(d.label)` is the key. When an item leaves the array, Svelte runs its `out:` transition before removing the element from the DOM.

