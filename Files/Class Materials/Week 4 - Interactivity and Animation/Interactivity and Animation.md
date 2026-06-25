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

---

## **Putting it together**

The sections above cover individual concepts. The examples below walk through two complete charts — a column chart and a line chart — from a static starter to a finished interactive version.

**Column chart** — adds `$state` for selection and tooltips, event handlers on each bar, and a CSS opacity transition.

**Line chart** — adds a metric toggle, `$derived` scales and generators, `bind:clientWidth` for responsive width, and CSS/Svelte transitions when the data view changes.

---

### **Column chart starter**

```
<script>
  import * as d3 from 'd3'

  const data = [
    { city: "New York",  avg: 55 },
    { city: "Miami",     avg: 77 },
    { city: "Chicago",   avg: 50 },
    { city: "Houston",   avg: 68 },
    { city: "Seattle",   avg: 52 }
  ]

  const width = 500
  const height = 320
  const margin = { top: 20, right: 20, bottom: 40, left: 45 }

  const xScale = d3.scaleBand()
    .domain(data.map(d => d.city))
    .range([margin.left, width - margin.right])
    .padding(0.2)

  const yScale = d3.scaleLinear()
    .domain([0, 100])
    .range([height - margin.bottom, margin.top])

  const colorScale = d3.scaleOrdinal()
    .domain(data.map(d => d.city))
    .range(d3.schemeTableau10)
</script>

<svg {width} {height}>
  {#each yScale.ticks(5) as tick}
    <line
      x1={margin.left}
      x2={width - margin.right}
      y1={yScale(tick)}
      y2={yScale(tick)}
      stroke="#e5e5e5"
    />
    <text
      x={margin.left - 8}
      y={yScale(tick)}
      text-anchor="end"
      dominant-baseline="middle"
    >{tick}°</text>
  {/each}

  <line
    x1={margin.left}
    x2={width - margin.right}
    y1={height - margin.bottom}
    y2={height - margin.bottom}
    stroke="#999"
  />

  {#each data as d}
    <rect
      x={xScale(d.city)}
      y={yScale(d.avg)}
      width={xScale.bandwidth()}
      height={height - margin.bottom - yScale(d.avg)}
      fill={colorScale(d.city)}
    />
    <text
      x={xScale(d.city) + xScale.bandwidth() / 2}
      y={height - margin.bottom + 16}
      text-anchor="middle"
    >{d.city}</text>
  {/each}
</svg>

<style>
  text {
    font-family: sans-serif;
    font-size: 12px;
    fill: #555;
  }
</style>
```

---

### **Column chart finished**

Adds selection state, tooltips, and opacity transitions on hover and click.

```
<script>
  import * as d3 from 'd3'

  const data = [
    { city: "New York", avg: 55 },
    { city: "Miami", avg: 77 },
    { city: "Chicago", avg: 50 },
    { city: "Houston", avg: 68 },
    { city: "Seattle", avg: 52 }
  ]

  const width = 500
  const height = 320
  const margin = { top: 20, right: 20, bottom: 40, left: 45 }

  const xScale = d3.scaleBand()
    .domain(data.map(d => d.city))
    .range([margin.left, width - margin.right])
    .padding(0.2)

  const yScale = d3.scaleLinear()
    .domain([0, 100])
    .range([height - margin.bottom, margin.top])

  const colorScale = d3.scaleOrdinal()
    .domain(data.map(d => d.city))
    .range(d3.schemeTableau10)

  let selected = $state(null)
  let tooltip = $state(null)
</script>

<svg {width} {height}>
  {#each yScale.ticks(5) as tick}
    <line
      x1={margin.left}
      x2={width - margin.right}
      y1={yScale(tick)}
      y2={yScale(tick)}
      stroke="#e5e5e5"
    />
    <text
      x={margin.left - 8}
      y={yScale(tick)}
      text-anchor="end"
      dominant-baseline="middle"
    >{tick}°</text>
  {/each}

  <line
    x1={margin.left}
    x2={width - margin.right}
    y1={height - margin.bottom}
    y2={height - margin.bottom}
    stroke="#999"
  />

  {#each data as d}
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <rect
      x={xScale(d.city)}
      y={yScale(d.avg)}
      width={xScale.bandwidth()}
      height={height - margin.bottom - yScale(d.avg)}
      fill={colorScale(d.city)}
      opacity={selected == null || selected == d.city ? 1 : 0.2}
      onclick={() => selected = selected === d.city ? null : d.city}
      onmouseenter={() => {
        tooltip = {
          x: xScale(d.city) + xScale.bandwidth() / 2,
          y: yScale(d.avg) - 8,
          text: `${d.city}: ${d.avg}`
        }
      }}
      onmouseleave={() => tooltip = null}
    />
    <text
      x={xScale(d.city) + xScale.bandwidth() / 2}
      y={height - margin.bottom + 16}
      text-anchor="middle"
    >{d.city}</text>
  {/each}

  {#if tooltip}
    <text
      x={tooltip.x}
      y={tooltip.y}
      text-anchor="middle"
      font-size="13"
      font-weight="bold"
      fill="#222"
    >{tooltip.text}</text>
  {/if}
</svg>

<style>
  text {
    font-family: sans-serif;
    font-size: 12px;
    fill: #555;
  }

  rect {
    cursor: pointer;
    transition: opacity 200ms ease;
  }
</style>
```

---

### **Line chart starter**

```
<script>
  import * as d3 from 'd3'

  const data = [
    { month: "Jan", temp: 34, precip: 3.2 },
    { month: "Feb", temp: 37, precip: 2.8 },
    { month: "Mar", temp: 46, precip: 4.0 },
    { month: "Apr", temp: 57, precip: 3.7 },
    { month: "May", temp: 67, precip: 3.5 },
    { month: "Jun", temp: 76, precip: 3.4 },
    { month: "Jul", temp: 82, precip: 4.3 },
    { month: "Aug", temp: 80, precip: 3.8 },
    { month: "Sep", temp: 72, precip: 3.2 },
    { month: "Oct", temp: 61, precip: 2.7 },
    { month: "Nov", temp: 50, precip: 3.1 },
    { month: "Dec", temp: 39, precip: 3.6 }
  ]

  const width = 560
  const height = 320
  const margin = { top: 20, right: 20, bottom: 30, left: 45 }

  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([margin.left, width - margin.right])
    .padding(0.5)

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.temp)])
    .range([height - margin.bottom, margin.top])
    .nice()

  const lineGenerator = d3.line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.temp))
    .curve(d3.curveCatmullRom)
</script>

<svg {width} {height}>
  {#each yScale.ticks(5) as tick}
    <line
      x1={margin.left}
      x2={width - margin.right}
      y1={yScale(tick)}
      y2={yScale(tick)}
      stroke="#e5e5e5"
    />
    <text
      x={margin.left - 8}
      y={yScale(tick)}
      text-anchor="end"
      dominant-baseline="middle"
    >{tick}</text>
  {/each}

  {#each data as d}
    <text
      x={xScale(d.month)}
      y={height - margin.bottom + 16}
      text-anchor="middle"
    >{d.month}</text>
  {/each}

  <line
    x1={margin.left}
    x2={margin.left}
    y1={margin.top}
    y2={height - margin.bottom}
    stroke="#999"
  />
  <line
    x1={margin.left}
    x2={width - margin.right}
    y1={height - margin.bottom}
    y2={height - margin.bottom}
    stroke="#999"
  />

  <path
    d={lineGenerator(data)}
    fill="none"
    stroke="steelblue"
    stroke-width="2.5"
  />
</svg>

<style>
  text {
    font-family: sans-serif;
    font-size: 12px;
    fill: #555;
  }
</style>
```

---

### **Line chart finished**

Adds a metric toggle, responsive width via `bind:clientWidth`, reactive scales, and transitions between states.

```
<script>
  import * as d3 from 'd3'
  import { fade } from 'svelte/transition'

  const data = [
    { month: "Jan", temp: 34, precip: 3.2 },
    { month: "Feb", temp: 37, precip: 2.8 },
    { month: "Mar", temp: 46, precip: 4.0 },
    { month: "Apr", temp: 57, precip: 3.7 },
    { month: "May", temp: 67, precip: 3.5 },
    { month: "Jun", temp: 76, precip: 3.4 },
    { month: "Jul", temp: 82, precip: 4.3 },
    { month: "Aug", temp: 80, precip: 3.8 },
    { month: "Sep", temp: 72, precip: 3.2 },
    { month: "Oct", temp: 61, precip: 2.7 },
    { month: "Nov", temp: 50, precip: 3.1 },
    { month: "Dec", temp: 39, precip: 3.6 }
  ]

  const height = 320
  const margin = { top: 20, right: 20, bottom: 30, left: 45 }

  let metric = $state("temp")
  let containerWidth = $state(0)

  let xScale = $derived(
    d3.scalePoint()
      .domain(data.map(d => d.month))
      .range([margin.left, containerWidth - margin.right])
      .padding(0.5)
  )

  let yScale = $derived(
    d3.scaleLinear()
      .domain([0, d3.max(data, d => d[metric])])
      .range([height - margin.bottom, margin.top])
      .nice()
  )

  let lineGenerator = $derived(
    d3.line()
      .x(d => xScale(d.month))
      .y(d => yScale(d[metric]))
      .curve(d3.curveCatmullRom)
  )
</script>

<div bind:clientWidth={containerWidth}>
  <div class="controls">
    <button onclick={() => metric = "temp"}>Temperature</button>
    <button onclick={() => metric = "precip"}>Precipitation</button>
  </div>

  {#if containerWidth > 0}
    <svg width={containerWidth} {height}>
      {#each yScale.ticks(5) as tick (tick)}
        <g
          style="transform: translateY({yScale(tick)}px)"
          transition:fade={{ duration: 150 }}
        >
          <line
            x1={margin.left}
            x2={containerWidth - margin.right}
            y1="0"
            y2="0"
            stroke="#e5e5e5"
          />
          <text
            x={margin.left - 8}
            y="0"
            text-anchor="end"
            dominant-baseline="middle"
          >{tick}</text>
        </g>
      {/each}

      {#each data as d}
        <text
          x={xScale(d.month)}
          y={height - margin.bottom + 16}
          text-anchor="middle"
        >{d.month}</text>
      {/each}

      <line
        x1={margin.left}
        x2={margin.left}
        y1={margin.top}
        y2={height - margin.bottom}
        stroke="#999"
      />
      <line
        x1={margin.left}
        x2={containerWidth - margin.right}
        y1={height - margin.bottom}
        y2={height - margin.bottom}
        stroke="#999"
      />

      <path
        d={lineGenerator(data)}
        fill="none"
        stroke="steelblue"
        stroke-width="2.5"
      />
    </svg>
  {/if}
</div>

<style>
  g {
    transition: transform 300ms ease;
  }

  path {
    transition: d 300ms ease;
  }

  text {
    font-family: sans-serif;
    font-size: 12px;
    fill: #555;
  }

  .controls {
    display: flex;
    gap: 8px;
    margin-bottom: 8px;
  }
</style>
```

