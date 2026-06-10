### **Reading**

[**Official Svelte Tutorial:**](https://svelte.dev/tutorial/svelte/welcome-to-svelte) Complete the following modules: Introduction, Reactivity, Props, and Logic

### **Deliverables**

#### **Part 1 — Hand-drawn chart**

Pick a small dataset — real data, but small enough to work with by hand (most likely a slice of a larger dataset).

Draw a chart on a grid. Graph paper, Figma, Illustrator, Google Sheets with gridlines, anything that gives you a coordinate system to work with. Label your axes and scale. Then write pseudocode describing how the data is being encoded.

Your pseudocode should specify:

* Canvas dimensions  
* What the mark is  
* What each channel encodes and which data column drives it  
* How your scale works (what is the input range, what is the output range, what is the formula)

Write your pseudocode in a way that makes sense to you. This is just to get you thinking through the steps that would be "under the hood" of a charting tool. Here's an example using average monthly temperatures:

```
dataset: average high temp by month, Philadelphia (6 rows)
columns: month (categorical), temp (quantitative)

canvas: 600 wide, 400 tall
margin: 50px on all sides
usable area: 500 wide, 300 tall

mark: BAR

channels:
  x position → month (categorical, evenly spaced)
  height     → temp (quantitative)
  color      → steelblue (constant, not encoding data)

scales:
  x: 6 months across 500px
     bar width: 60px, gap: 20px
     formula: x = margin + i * (barWidth + gap)

  y: temp ranges 0–100 (rounded for headroom)
     100 units maps to 300px
     1 unit = 3px
     bar height = temp * 3
     bar y = 350 - (temp * 3)

calculated positions:
  Jan → x=50,  y=230, height=120
  Feb → x=130, y=218, height=132
  Mar → x=210, y=188, height=162
  Apr → x=290, y=155, height=195
  May → x=370, y=125, height=225
  Jun → x=450, y=101, height=249
```

**Submit:** photos or screenshots of the hand-drawn chart with pseudocode (either in the image or in the text entry is fine).

#### **Part 2 — Try to replicate in Svelte**

Clone the class starter template and rename it for the week.

Replicate your hand-drawn chart as closely as you can using only Svelte and SVG — no libraries yet. Your data should live in the \<script\> block as an array of objects. Try rendering your marks with {\#each}. If you're feeling up for it, try modifying one of the scales in the week 1 code reference to to fit your data.

The goal isn't a perfect match. The goal is to translate the scale logic you already worked out by hand into code. If something behaves differently than you expected, note it. Focusing in on the logic needed to visualize an element will help make the transition to D3 easier.

**Submit:** a link to your GitHub repository and a screenshot of the rendered chart.

