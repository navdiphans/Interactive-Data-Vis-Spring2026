---
toc: true
theme: coffee
---

# Lab 1: Prolific Pollinators

<br>

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

Carpenter Bees are the heaviest species, with body masses ranging from 0.301g to 0.518g. Bumblebees are mid-range with a range of 0.206g to 0.311g. Honeybees are the lightest with a range of 0.076g to 0.116g.

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
  marginBottom: 60
  })
```

Carpenter Bees have the longest wings, ranging from 33.2mm to 49.1mm. Bumblebees range from 25.4mm to 41.6mm. Honeybees have the shortest wings, ranging from 15.7mm to 22.4mm.

<br>

## Question 2: Ideal Weather Conditions for Pollinating

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
Cloudy conditions produce the highest average pollinator visit counts, followed closely by Partly Cloudy and Sunny conditions. Visits increase steadily with temperature, increasing at temperatures above 27°C. Lower humidity levels around 60 to 70% percent are associated with slightly more visits. Wind speed has a significant negative effect, with visit counts dropping markedly as wind increases above 3km/h.

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

Sunflower produces the most nectar on average at 0.94 μL per flower. Coneflower produces 0.64 μL on average. Lavender produces the least at 0.54 μL on average.