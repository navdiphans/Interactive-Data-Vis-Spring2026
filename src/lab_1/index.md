---
title: "Lab 1: Passing Pollinators"
toc: true
theme: stark
---


# Lab 1: Prolific Pollinators
```js
const data = await FileAttachment("data/pollinator_activity_data.csv").csv({typed: true});
```

## Question 1: Body Mass and Wing Span by Pollinator Species

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.dot(data, {x: "pollinator_species", y: "avg_body_mass_g", fill: "pollinator_species", r: 3, opacity: 0.5}),
    Plot.tip(data, Plot.pointer({x:"pollinator_species", y:"avg_body_mass_g", title: d => `${d.pollinator_species}: ${d.avg_body_mass_g}g`})),
    Plot.frame()
    ],
  y: {label: "Body Mass (g)", domain: [0, 0.75]},
  x: {label: "Species", tickRotate: 0, labelOffset: 50}, 
  title: "Body Mass Distribution by Pollinator Species",
  color: {legend: true},
  marginBottom: 60
  })
```
Carpenter Bees are the heaviest pollinator species observed, with body masses clustered 
around 0.38 to 0.51g. Bumblebees fall in the middle range at approximately 0.22 to 0.30g. 
Honeybees are the lightest species, with body masses tightly grouped around 0.08 to 0.11g.

<br>

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.dot(data, {x: "pollinator_species", y: "avg_wing_span_mm", fill: "pollinator_species", r: 3, opacity: 0.5}),
    Plot.tip(data, Plot.pointer({x: "pollinator_species", y: "avg_wing_span_mm", title: d => `${d.pollinator_species}: ${d.avg_wing_span_mm}mm`})),
    Plot.frame()
  ],
  y: {label: "Wing Span (mm)", domain: [0, 75]},
  x: {label: "Species", labelOffset: 50},
  title: "Wing Span Distribution by Pollinator Species",
  color: {legend: true},
  marginBottom: 80
  })
```
Carpenter Bees have the longest wing spans, ranging from approximately 33 to 49mm. 
Bumblebees have a mid-range wing span of 26 to 41mm. Honeybees have the shortest wings, 
tightly grouped between 16 and 22mm. Wing span closely mirrors the body mass pattern 
across all three species.
<br>
<br>
<br>


## Question 2: Ideal Weather Conditions for Pollinating
<br>
Cloudy conditions produce the highest average pollinator visit counts, followed closely 
by Partly Cloudy and Sunny conditions. Visits increase steadily with temperature, peaking 
at warmer temperatures above 27°C. Lower humidity levels around 60 to 70% are associated 
with slightly more visits. Wind speed has the strongest negative effect, with visit counts 
dropping sharply as wind increases above 3km/h.


```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.barY(data, Plot.groupX({y: "mean"}, {x: "weather_condition", y: "visit_count", fill: "weather_condition"})),
    Plot.ruleY([0]),
    Plot.frame()
  ],
  y: {label: "Avg Visit Count", domain: [0, 10]},
  x: {label: "Weather Condition"},
  title: "Average Pollinator Visits by Weather Condition",
  color: {legend: true},
  style: {fontSize: 14, fontWeight: "bold"}
 
})
```

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.lineY(data, Plot.binX({y: "mean"}, {x: "temperature", y: "visit_count", stroke: "steelblue"})),
    Plot.dot(data, Plot.binX({y: "mean"}, {x: "temperature", y: "visit_count", fill: "steelblue"})),
    Plot.frame()
  ],
  y: {label: "Avg Visit Count", domain: [0, 15]},
  x: {label: "Temperature (°C)"},
  title: "Average Pollinator Visits by Temperature",
  style: {fontSize: 14, fontWeight: "bold"}
})

```

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.lineY(data, Plot.binX({y: "mean"}, {x: "humidity", y: "visit_count", stroke: "steelblue"})),
    Plot.dot(data, Plot.binX({y: "mean"}, {x: "humidity", y: "visit_count", fill: "steelblue"})),
    Plot.frame()
  ],
  y: {label: "Avg Visit Count", domain: [0, 10]},
  x: {label: "Humidity (%)"},
  title: "Average Pollinator Visits by Humidity",
  style: {fontSize: 14, fontWeight: "bold"}
})

```

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.lineY(data, Plot.binX({y: "mean"}, {x: "wind_speed", y: "visit_count", stroke: "steelblue"})),
    Plot.dot(data, Plot.binX({y: "mean"}, {x: "wind_speed", y: "visit_count", fill: "steelblue"})),
    Plot.frame()
  ],
  y: {label: "Avg Visit Count", domain: [0,20]},
  x: {label: "Wind Speed (km/h)"},
  title: "Average Pollinator Visits by Wind Speed",
  style: {fontSize: 14, fontWeight: "bold"}
})

```
<br>
<br>
<br>

## Question 3: Nectar Production by Flower Species

```js
Plot.plot({
  marks: [
    Plot.gridY({stroke: "grey", strokeOpacity: 0.5}),
    Plot.barY(data, Plot.groupX({y: "mean"}, {x: "flower_species", y: "nectar_production", fill: "flower_species"})),
  Plot.frame()
  ],
  y: {label: "Avg Nectar Production (μL)", domain: [0, 1.5]},
  x: {label: "Species", labelOffset: 50},
  title: "Average Nectar Production by Flower Species",
  color: {legend: true},
  marginBottom: 60
  })
```
Sunflower produces the most nectar on average at approximately 0.95 μL per flower, 
making it the most productive species in this dataset. Coneflower produces a moderate 
amount at around 0.63 μL, while Lavender produces the least at approximately 0.54 μL.
