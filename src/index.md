---
theme: dashboard
title: Cami's Thesis Graphs
toc: false
footer: false
---

<!-- Imports and data loads -->

```js
import { regressionLinear } from "npm:d3-regression";
const preparedData = FileAttachment("data/prepareddata.csv").csv({
  typed: true,
});
```

<!-- Transform data -->

```js
const dataHeaders = Object.keys(preparedData[0]);
const catScores = preparedData.flatMap((d) =>
  ["Compensation", "Masking", "Assimilation"].map((category) => ({
    x: d[`CATQ_${category}`],
    y: d["NR6_Score"],
    age: d["Age"],
    ethnicity: d["Ethnicity"],
    gender: d["Gender"],
    education: d["Education"],
    Q1: d["Response_Q1_Natural_World_Importance"],
    Q2: d["Response_Q2_Favorite_Things"],
    Q3: d["Response_Q3_Nature_Knowledge"],
    category,
  }))
);
```

<!-- Inputs -->

```js
const categoryInput = Inputs.radio(
  ["Compensation", "Masking", "Assimilation"],
  {
    label: "CAT-Q component",
    value: "Compensation",
  }
);
const genderInput = Inputs.toggle({
  label: "Control for gender",
  value: true,
});
const selectedCategory = Generators.input(categoryInput);
const controlForGender = Generators.input(genderInput);
```

<!-- Calculate linear regression -->

```js
const regression = regressionLinear()
  .x((d) => d.x)
  .y((d) => d.y);
const compRegression = regression(
  catScores.filter((d) => d.category === "Compensation")
);
const maskRegression = regression(
  catScores.filter((d) => d.category === "Masking")
);
const assimRegression = regression(
  catScores.filter((d) => d.category === "Assimilation")
);
console.log(
  compRegression.rSquared,
  maskRegression.rSquared,
  assimRegression.rSquared
);
```

<!-- Plot of CAT-Q and NR6 correlation -->

```js
function catVsNr6(data, gender, { width } = {}) {
  const plot = Plot.plot({
    title: "CAT-Q and NR6 Correlation",
    width,
    height: width * 0.6,
    grid: true,
    x: { label: "CATQ Subscore" },
    y: { label: "NR6 Score" },
    marks: [
      Plot.dot(data, {
        x: "x",
        y: "y",
        fy: gender ? "gender" : null,
        fill: "category",
        symbol: "category",
        filter: (d) => d.category === selectedCategory,
      }),
      Plot.dot(
        data,
        Plot.pointer({
          x: "x",
          y: "y",
          fy: gender ? "gender" : null,
          fill: "category",
          symbol: "category",
          r: 8,
          filter: (d) => d.category === selectedCategory,
        })
      ),
      Plot.linearRegressionY(data, {
        x: "x",
        y: "y",
        fy: gender ? "gender" : null,
        stroke: "category",
        filter: (d) => d.category === selectedCategory,
      }),
      Plot.text(
        data,
        Plot.pointer({
          px: "x",
          py: "y",
          fy: gender ? "gender" : null,
          dx: 25,
          dy: -10,
          fontSize: 12,
          text: (d) =>
            `Age: ${d.age}\nEthnicity: ${d.ethnicity}\nGender: ${d.gender}\nEducation: ${d.education}`,
          frameAnchor: "bottom-left",
          filter: (d) => d.category === selectedCategory,
        })
      ),
    ],
  });
  plot.addEventListener("input", (_) => {
    document.getElementById("Q1").innerText = plot.value.Q1;
    document.getElementById("Q2").innerText = plot.value.Q2;
    document.getElementById("Q3").innerText = plot.value.Q3;
  });
  return plot;
}
```

```html
<div class="grid grid-cols-2" style="grid-auto-rows: auto;">
  <div class="card grid-colspan-2">
    ${resize((width) => catVsNr6(catScores, controlForGender, { width }))}
  </div>
  <div class="card">${categoryInput}${genderInput}</div>
  <div class="card">
    <div id="display_data">
      <p><b>Natural World Importance</b></p>
      <p id="Q1"></p>
      <p><b>Favorite Things</b></p>
      <p id="Q2"></p>
      <p><b>Nature Knowledge</b></p>
      <p id="Q3"></p>
    </div>
  </div>
</div>
```
