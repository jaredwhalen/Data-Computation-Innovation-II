### **Reading**

[**Using D3 with Svelte:**](https://datavisualizationwithsvelte.com/) Read through the Essentials section.

### **Final project update**

**Data in hand:** Have your dataset acquired and at least partially cleaned. Prepare a rough sketch (on paper or in code) of the visual structure you're considering.


### **Deliverables**

**Week 3 Homework — Observable Translation**

Find two charts from the [official D3 gallery](https://observablehq.com/@d3/gallery?utm_source=d3js-org&utm_medium=hero&utm_campaign=try-observable) or elsewhere on [Observable](https://observablehq.com) and recreate them in Svelte.

This is a translation exercise. The goal is to get comfortable reading imperative D3 code in the wild and converting it into the declarative style we've been using.

---

**Chart 1 — Recreate**

Find a chart from the official D3 gallery or Observable that interests you. Recreate it as closely as possible in your Svelte starter template — same data, same chart type, same visual style.

A few things to keep in mind:

* Copy the scales and generators directly. They don't change between approaches.
* Translate `selectAll` + `data` + `join` into `{#each}` loops.
* Translate `.attr()` chains into inline SVG attributes.
* Watch for the D3 version the notebook uses. If something isn't working, that's often why — check the code reference for the translation pattern.

You don't need to match it pixel-for-pixel, but try to get it as close as possible. The goal is a working chart that produces the same output from the same data.

---

**Chart 2 — Adapt**

Find a second chart — it can be a different type or a different approach. This time, adapt it for a dataset of your own choosing. Change the colors, adjust the layout, make it yours.

What to change:

* Swap in your own data and update column references accordingly
* Adjust the domain and range to fit your data
* Change colors, stroke widths, font sizes — any visual properties you want

What to keep:

* The overall chart structure and mark type
* The scale and generator pattern

---

**Submitting**

Post both charts to Courseworks before next class. For each one include:

* A link to your GitHub repository
* A screenshot of the original (D3 gallery or Observable)
* A screenshot of your Svelte version
* The link to the example you started from
* One or two sentences on the biggest translation challenge you ran into