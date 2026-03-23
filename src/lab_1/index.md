---
title: "Lab 1: Passing Pollinators"
toc: true
theme: stark
---

</style>

<style>
  figure svg text {
    font-size: 14px !important;
  }
  figure svg .tick text {
    transform: translateY(8px) !important;
  }
</style>


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
  marginBottom: 80
  })
```

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



## Question 2: Ideal Weather Conditions for Pollinating
<br>
In this set of data, the ideal weather conditions for pollinating are during cloudy weather, 
temps greater than 27 degrees celsius, with humidity less than 75% percent, and windspeed under 3km/h.


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
  marginBottom: 80
  })
```
