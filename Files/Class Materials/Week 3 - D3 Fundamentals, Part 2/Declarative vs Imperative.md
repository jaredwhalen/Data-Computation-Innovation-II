# **Declarative vs. Imperative** Code Reference

## **Setup**

Start from this component. The data, dimensions, scales, and generator are identical in both approaches — copy them once, then build declaratively or compare against the imperative version below:

```
<script>
  import * as d3 from 'd3'

  const data = [
    { month: "Jan", temp: 40 },
    { month: "Feb", temp: 44 },
    { month: "Mar", temp: 54 },
    { month: "Apr", temp: 65 },
    { month: "May", temp: 75 },
    { month: "Jun", temp: 83 },
    { month: "Jul", temp: 88 },
    { month: "Aug", temp: 86 },
    { month: "Sep", temp: 78 },
    { month: "Oct", temp: 67 },
    { month: "Nov", temp: 55 },
    { month: "Dec", temp: 44 }
  ]

  const width = 600
  const height = 400
  const margin = { top: 20, right: 20, bottom: 40, left: 50 }

  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([margin.left, width - margin.right])

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.temp)])
    .range([height - margin.bottom, margin.top])

  const lineGenerator = d3.line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.temp))
</script>

<svg viewBox="0 0 {width} {height}" {width} {height}>

</svg>
```

---

## **The two approaches**

Everything you've built so far has been declarative — you describe what the output should look like and Svelte renders it. Most D3 examples you'll find in the wild are imperative — D3 selects DOM elements and builds the chart directly, step by step.

You'll write declaratively. You'll read imperatively. D3's math is portable between both — scales and generators copy directly.

---

## **Running imperative D3 in Svelte**

In a plain HTML file, imperative D3 runs immediately because the DOM already exists when the script tag runs. In Svelte, the DOM doesn't exist until the component renders. Any imperative D3 code needs to go inside `onMount`, which runs after the first render.

You also need a reference to the container element. In Svelte you do this with `bind:this`:

```
<script>
  import * as d3 from 'd3'
  import { onMount } from 'svelte'

  let container

  onMount(() => {
    // container is available here
    // safe to select and write into it
  })
</script>

<div bind:this={container}></div>
```

`bind:this={container}` connects the `container` variable to the actual DOM element. Inside `onMount`, use `container` instead of `"#chart"`.

---

## **Creating the SVG**

**Imperative** — inside `onMount`:

```javascript
const svg = d3.select(container)
  .append("svg")
  .attr("viewBox", `0 0 ${width} ${height}`)
  .attr("width", width)
  .attr("height", height)
  .style('border', '1px solid #ccc')
```

**Declarative**

```
<svg viewBox="0 0 {width} {height}" {width} {height}>
</svg>
```

---

## **Adding the line**

**Imperative** — inside `onMount`, after the svg is created:

```javascript
svg.append("path")
  .datum(data)
  .attr("d", lineGenerator)
  .attr("fill", "none")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
```

**Declarative** — inside `<svg>`:

```
<path
  d={lineGenerator(data)}
  fill="none"
  stroke="steelblue"
  stroke-width="2"
/>
```

`.datum(data)` binds the entire dataset to a single element — different from `.data()`, which binds one item per element. In the declarative version you call the generator yourself: `d={lineGenerator(data)}`.

---

## **Adding dots**

**Imperative** — inside `onMount`, after the path:

```javascript
svg.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", d => xScale(d.month))
  .attr("cy", d => yScale(d.temp))
  .attr("r", 4)
  .attr("fill", "white")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
```

**Declarative** — inside `<svg>`, after the path:

```
{#each data as d}
  <circle
    cx={xScale(d.month)}
    cy={yScale(d.temp)}
    r="4"
    fill="white"
    stroke="steelblue"
    stroke-width="2"
  />
{/each}
```

`selectAll("circle").data(data).join("circle")` — the data join — is the imperative equivalent of `{#each}`. Select all circles (even if none exist yet), bind data, create one per row.

---

## **The complete imperative version**

```
<script>
  import * as d3 from 'd3'
  import { onMount } from 'svelte'

  const data = [
    { month: "Jan", temp: 40 },
    { month: "Feb", temp: 44 },
    { month: "Mar", temp: 54 },
    { month: "Apr", temp: 65 },
    { month: "May", temp: 75 },
    { month: "Jun", temp: 83 },
    { month: "Jul", temp: 88 },
    { month: "Aug", temp: 86 },
    { month: "Sep", temp: 78 },
    { month: "Oct", temp: 67 },
    { month: "Nov", temp: 55 },
    { month: "Dec", temp: 44 }
  ]

  const width = 600
  const height = 400
  const margin = { top: 20, right: 20, bottom: 40, left: 50 }

  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([margin.left, width - margin.right])

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.temp)])
    .range([height - margin.bottom, margin.top])

  const lineGenerator = d3.line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.temp))

  let container

  onMount(() => {
    const svg = d3.select(container)
      .append("svg")
      .attr("viewBox", `0 0 ${width} ${height}`)
      .attr("width", width)
      .attr("height", height)

    svg.append("path")
      .datum(data)
      .attr("d", lineGenerator)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 2)

    svg.selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", d => xScale(d.month))
      .attr("cy", d => yScale(d.temp))
      .attr("r", 4)
      .attr("fill", "white")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 2)
  })
</script>

<div bind:this={container}></div>
```

---

## **The complete declarative version**

```
<script>
  import * as d3 from 'd3'

  const data = [
    { month: "Jan", temp: 40 },
    { month: "Feb", temp: 44 },
    { month: "Mar", temp: 54 },
    { month: "Apr", temp: 65 },
    { month: "May", temp: 75 },
    { month: "Jun", temp: 83 },
    { month: "Jul", temp: 88 },
    { month: "Aug", temp: 86 },
    { month: "Sep", temp: 78 },
    { month: "Oct", temp: 67 },
    { month: "Nov", temp: 55 },
    { month: "Dec", temp: 44 }
  ]

  const width = 600
  const height = 400
  const margin = { top: 20, right: 20, bottom: 40, left: 50 }

  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([margin.left, width - margin.right])

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.temp)])
    .range([height - margin.bottom, margin.top])

  const lineGenerator = d3.line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.temp))
</script>

<svg viewBox="0 0 {width} {height}" {width} {height}>
  <path
    d={lineGenerator(data)}
    fill="none"
    stroke="steelblue"
    stroke-width="2"
  />

  {#each data as d}
    <circle
      cx={xScale(d.month)}
      cy={yScale(d.temp)}
      r="4"
      fill="white"
      stroke="steelblue"
      stroke-width="2"
    />
  {/each}
</svg>
```

---

## **Reading Observable**

Observable uses the same imperative style with a few differences:

```javascript
{
  const svg = d3.create("svg")
    .attr("viewBox", [0, 0, width, height])

  svg.append("path")
    .datum(data)
    .attr("d", lineGenerator)
    .attr("fill", "none")
    .attr("stroke", "steelblue")

  return svg.node()
}
```

`d3.create("svg")` instead of `d3.select(container).append("svg")` — Observable creates the element directly rather than finding a container.

`return svg.node()` hands the element back to Observable to display it.

Everything in between is identical to standard imperative D3. The translation process is the same.

---

## **Quick reference**

| Imperative | Declarative |
|---|---|
| `import { onMount } from 'svelte'` | Not needed |
| `let container` + `bind:this={container}` | Not needed |
| `onMount(() => { ... })` | Not needed |
| `d3.select(container).append("svg")` | `<svg>` in your template |
| `.attr("fill", "steelblue")` | `fill="steelblue"` |
| `.attr("cx", d => xScale(d.month))` | `cx={xScale(d.month)}` |
| `.datum(data).attr("d", lineGenerator)` | `d={lineGenerator(data)}` |
| `selectAll("circle").data(data).join("circle")` | `{#each data as d}<circle />{/each}` |
| Scales and generators | Identical — copy directly |
