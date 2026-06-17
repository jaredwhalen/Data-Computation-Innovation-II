# **D3 Generators** Code Reference

## **Setup**


Start from this component. Copy it once, then add to it as you work through each section below:

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
</script>

<svg viewBox="0 0 {width} {height}" {width} {height}>

</svg>
```

---

## **What a generator is**

A shape generator is a function that takes your data array and returns an SVG path string. You pass it your data, it gives you back something like:

```
"M 60,278 L 160,265 L 260,232 L 360,195..."
```

That string goes directly into a `<path>` element's `d` attribute. D3 handles the math, Svelte handles the rendering — same pattern as scales.

`.x()` and `.y()` are accessor functions. Same pattern as inline SVG attributes in `{#each}` loops — except instead of Svelte calling them once per element, the generator calls them once per row and assembles a path string from the results.

---

## **d3.line()**

Add a line generator below your scales:

```javascript
const lineGenerator = d3.line()
  .x(d => xScale(d.month))
  .y(d => yScale(d.temp))
```

Inside `<svg>`:

```
<path
  d={lineGenerator(data)}
  fill="none"
  stroke="steelblue"
  stroke-width="2"
/>
```

`fill="none"` because a `<path>` fills its interior by default — without this you'd get a filled shape rather than a line. `stroke` and `stroke-width` control the line color and thickness.

---

## **Adding dots**

The path handles the line. Add a `{#each}` loop inside `<svg>` for circles at each data point:

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

White circles with a blue stroke sit on top of the line. The `fill="white"` punches out the line at each data point, which gives it a cleaner look.

Order matters in SVG: the path comes before the circles in the markup, so the circles render on top. Swap them and the line draws over the dots.

---

## **d3.area()**

Add an area generator below your line generator:

```javascript
const areaGenerator = d3.area()
  .x(d => xScale(d.month))
  .y0(height - margin.bottom)
  .y1(d => yScale(d.temp))
```

Inside `<svg>`, add the area path before the line so the line renders on top:

```
<path
  d={areaGenerator(data)}
  fill="steelblue"
  opacity="0.15"
/>
```

`.y0()` is the baseline — the bottom of the chart area. `.y1()` is the data line. The area fills everything between them.

Try adjusting the opacity — 0.1 is subtle, 0.3 is more prominent.

---

## **Y axis**

Inside `<svg>`, add this before your chart marks so it renders behind them:

```
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
    font-size="12"
    fill="#666"
  >{tick}°</text>
{/each}
```

`yScale.ticks(5)` asks the scale for approximately five evenly-spaced values within the domain — it picks round numbers automatically. For each tick, draw a horizontal line spanning the full chart width and a text label to the left.

`text-anchor="end"` right-aligns the label so it sits flush against the left edge of the chart. `dominant-baseline="middle"` vertically centers it on the tick position.

---

## **X axis**

Inside `<svg>`, add month labels below the chart:

```
{#each data as d}
  <text
    x={xScale(d.month)}
    y={height - margin.bottom + 20}
    text-anchor="middle"
    font-size="12"
    fill="#666"
  >{d.month}</text>
{/each}
```

Loop over `data` directly rather than `xScale.ticks()` because `scalePoint` gives you exactly one position per category. The label sits 20 pixels below the bottom of the chart area.

---

## **Axis lines**

The gridlines label the chart, but the axis lines frame it. Add these inside `<svg>`, before your tick loops:

```
<line
  x1={margin.left}
  x2={width - margin.right}
  y1={height - margin.bottom}
  y2={height - margin.bottom}
  stroke="#ccc"
/>
<line
  x1={margin.left}
  x2={margin.left}
  y1={margin.top}
  y2={height - margin.bottom}
  stroke="#ccc"
/>
```

Place axis lines before the chart marks so gridlines and labels render behind the data.

---

## **The full pattern**

```
<script>
  import * as d3 from 'd3'

  // 1. data
  const data = [...]

  // 2. dimensions
  const width = 600
  const height = 400
  const margin = { top: 20, right: 20, bottom: 40, left: 50 }

  // 3. scales
  const xScale = d3.scalePoint()
    .domain(data.map(d => d.month))
    .range([margin.left, width - margin.right])

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .range([height - margin.bottom, margin.top])

  // 4. generators
  const lineGenerator = d3.line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.value))

  const areaGenerator = d3.area()
    .x(d => xScale(d.month))
    .y0(height - margin.bottom)
    .y1(d => yScale(d.value))
</script>

<!-- 5. rendering -->
<svg viewBox="0 0 {width} {height}" {width} {height}>
  <!-- axes -->
  <!-- area, line, dots -->
</svg>
```

D3 handles steps 3 and 4. Svelte handles step 5.
