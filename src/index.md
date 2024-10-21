---
theme: dashboard
title: Cami's Thesis Graphs
toc: false
footer: false
---

<!-- Imports and data loads -->

```js
import { regressionLinear } from "npm:d3-regression";
const data = FileAttachment("data/prepareddata.csv").csv({ typed: true });
```

<!-- Transform data -->

```js
const catScores = data.flatMap((d) => [
  {
    x: d["CATQ_Compensation"],
    y: d["NR6_Score"],
    Q1: d["Response_Q1_Natural_World_Importance"],
    Q2: d["Response_Q2_Favorite_Things"],
    Q3: d["Response_Q3_Nature_Knowledge"],
    label: "Compensation",
  },
  {
    x: d["CATQ_Masking"],
    y: d["NR6_Score"],
    Q1: d["Response_Q1_Natural_World_Importance"],
    Q2: d["Response_Q2_Favorite_Things"],
    Q3: d["Response_Q3_Nature_Knowledge"],
    label: "Masking",
  },
  {
    x: d["CATQ_Assimilation"],
    y: d["NR6_Score"],
    Q1: d["Response_Q1_Natural_World_Importance"],
    Q2: d["Response_Q2_Favorite_Things"],
    Q3: d["Response_Q3_Nature_Knowledge"],
    label: "Assimilation",
  },
]);
```

<!-- Calculate linear regression -->

```js
const regression = regressionLinear()
  .x((d) => d.x)
  .y((d) => d.y);
const compRegression = regression(
  catScores.filter((d) => d.label === "Compensation")
);
const maskRegression = regression(
  catScores.filter((d) => d.label === "Masking")
);
const assimRegression = regression(
  catScores.filter((d) => d.label === "Assimilation")
);
console.log(
  compRegression.rSquared,
  maskRegression.rSquared,
  assimRegression.rSquared
);
```

<!-- Plot of CAT-Q and NR6 correlation -->

```js
function catVsNr6(data, { width } = {}) {
  return Plot.plot({
    title: "CAT-Q and NR6 Correlation",
    width: 800,
    height: 600,
    grid: true,
    color: { legend: true },
    x: { label: "CATQ Subscore" },
    y: { label: "NR6 Score" },
    marks: [
      Plot.dot(catScores, {
        x: "x",
        y: "y",
        fill: "label",
        symbol: "label",
        // channels: {
        //   Q1: "Q1",
        //   Q2: "Q2",
        //   Q3: "Q3"
        // },
        title: (d) => [d.Q1, d.Q2, d.Q3].join("\n\n"),
        tip: true,
      }),
      Plot.linearRegressionY(catScores, { x: "x", y: "y", stroke: "label" }),
    ],
  });
}
```

```html
<div class="grid grid-cols-1">
  <div class="card">${resize((width) => catVsNr6(catScores, { width }))}</div>
</div>
```
