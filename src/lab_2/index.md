---
title: "Lab 2: Subway Staffing"
toc: true
---

```js
const incidents = await FileAttachment("./data/incidents.csv").csv();
const local_events = await FileAttachment("./data/local_events.csv").csv();
const upcoming_events = await FileAttachment("./data/upcoming_events.csv").csv();
const ridership = await FileAttachment("./data/ridership.csv").csv();
```
```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
};
```

Question 1. How did local events impact ridership in summer 2025? What effect did the July 15th fare increase have?
<br>
<br>


```js
// Daily totals across all stations
const dailyMap = new Map();
for (const d of ridership) {
  const total = +d.entrances + +d.exits;
  dailyMap.set(d.date, (dailyMap.get(d.date) ?? 0) + total);
}
const daily = [...dailyMap.entries()]
  .map(([ds, total]) => ({ date: new Date(ds), total }))
  .sort((a, b) => a.date - b.date);

// Per-station season average
const stationSum   = new Map();
const stationCount = new Map();
for (const d of ridership) {
  const t = +d.entrances + +d.exits;
  stationSum.set(d.station,   (stationSum.get(d.station)   ?? 0) + t);
  stationCount.set(d.station, (stationCount.get(d.station) ?? 0) + 1);
}
const stationAvgMap = new Map(
  [...stationSum.entries()].map(([s, sum]) => [s, sum / stationCount.get(s)])
);

// Nested map: date string -> station -> total that day
const stationDayMap = new Map();
for (const d of ridership) {
  if (!stationDayMap.has(d.date)) stationDayMap.set(d.date, new Map());
  const inner = stationDayMap.get(d.date);
  inner.set(d.station, (inner.get(d.station) ?? 0) + (+d.entrances + +d.exits));
}

// Lift for each event vs. that station's seasonal average
const eventLift = local_events.map(e => {
  const dk       = new Date(e.date).toISOString().slice(0, 10);
  const observed = stationDayMap.get(dk)?.get(e.nearby_station);
  const avg      = stationAvgMap.get(e.nearby_station);
  if (observed == null || avg == null) return null;
  return { ...e, date: new Date(e.date), observed, lift_pct: (observed - avg) / avg * 100 };
}).filter(Boolean);
```
```js
const fareDate  = new Date("2025-07-15");
const preDays   = daily.filter(d => d.date <  fareDate);
const postDays  = daily.filter(d => d.date >= fareDate);
const preAvg    = preDays.reduce((s, d)  => s + d.total, 0) / preDays.length;
const postAvg   = postDays.reduce((s, d) => s + d.total, 0) / postDays.length;
const pctChange = (postAvg - preAvg) / preAvg * 100;
const meanLift  = eventLift.reduce((s, d) => s + d.lift_pct, 0) / eventLift.length;
const minDate   = daily[0].date;
const maxDate   = daily[daily.length - 1].date;
```




```js
html`
    ${Plot.plot({
    width: 860,
    height: 350,
    marginRight: 30,
    x: { type: "utc", label: "Date" },
    y: {
      label: "Total riders",
      tickFormat: d => (d / 1000).toFixed(0) + "k",
      domain: [490000, 715000]
    },
    marks: [
      Plot.rect(
  [
    { x1: new Date(minDate),  x2: new Date(fareDate), fill: "rgba(34,197,94,0.08)" },
    { x1: new Date(fareDate), x2: new Date(maxDate),  fill: "rgba(220,38,38,0.07)" }
  ],
  { x1: "x1", x2: "x2", fill: "fill", stroke: null }
),
      Plot.lineY(daily, {
        x: "date", y: "total",
        stroke: "#bfdbfe", strokeWidth: 1.5
      }),
      Plot.lineY(daily, Plot.windowY({ k: 7, reduce: "mean" }, {
        x: "date", y: "total",
        stroke: "#2563eb", strokeWidth: 2.5
      })),
      Plot.ruleY([preAvg], {
        x1: minDate, x2: fareDate,
        stroke: "#2563eb", strokeDasharray: "4,3", strokeOpacity: 0.4
      }),
      Plot.ruleY([postAvg], {
        x1: fareDate, x2: maxDate,
        stroke: "#dc2626", strokeDasharray: "4,3", strokeOpacity: 0.45
      }),
      Plot.ruleX([fareDate], {
        stroke: "#dc2626", strokeWidth: 2, strokeDasharray: "5,4"
      }),
      Plot.text(
        [{ date: fareDate, total: 704000, label: "Fare increase: Jul 15" }],
        {
          x: "date", y: "total", text: "label",
          textAnchor: "start", dx: 6,
          fill: "#dc2626", fontSize: 11, fontWeight: "600"
        }
      )
    ]
  })}
`
```
The July 15th fare increase had an impact on ridership. 
Before the increase, daily ridership across all 25 stations averaged around 632,000 riders per day.

On July 15th, ridership fell to 564,000 and never came back to pre-increase levels for the rest of the summer. 

The 7-day rolling average, clearing up daily noise, makes this noticeable. It held above 620,000 through the first half of July and fell very quickly after the fare increase.
<br>

---







```js
const selectedStation = view(Inputs.select(
  ["All stations", ...[...new Set(eventLift.map(d => d.nearby_station))].sort()],
  { label: "Station", value: "All stations" }
));
```



```js
const filteredEventLift = selectedStation === "All stations"
  ? eventLift
  : eventLift.filter(d => d.nearby_station === selectedStation);
```






```js
html`
  ${Plot.plot({
    width: 1024,
    height: 768,
    marginLeft: 185,
    marginRight: 30,
    style: { fontSize: "13px" },
    x: { type: "utc", label: "Event date", grid: true, domain: [new Date("2025-06-01"), new Date("2025-08-14")] },
    y: { label: "Station", grid: true },
    r: d => Math.pow(+d.lift_pct, 1.8) / 800,
    r: { range: [3, 35] },
    color: {
      domain: ["Before Jul 15", "After Jul 15"],
      range: ["#2563eb", "#dc2626"],
      legend: true
    },
    marks: [
      Plot.dot(
        filteredEventLift,
        {
          x: "date",
          y: "nearby_station",
          r: d => Math.pow(+d.estimated_attendance, 0.7) / 80,
          fill: d => d.date < fareDate ? "Before Jul 15" : "After Jul 15",
          fillOpacity: 0.75,
          stroke: "white",
          strokeWidth: 0.5,
          tip: true,
          title: d => [
            d.event_name,
            `Station: ${d.nearby_station}`,
            `Lift: +${d.lift_pct.toFixed(1)}%`,
            `Est. attendance: ${(+d.estimated_attendance).toLocaleString()}`
          ].join("\n")
        }
      ),
      Plot.ruleX([fareDate], {
        stroke: "#dc2626",
        strokeWidth: 2,
        strokeDasharray: "5,4"
      }),
      Plot.text(
        [{ date: fareDate }],
        {
          x: "date",
          frameAnchor: "top",
          text: () => "Fare increase: Jul 15",
          textAnchor: "start",
          dx: 6,
          dy: 6,
          fill: "#dc2626",
          fontSize: 11,
          fontWeight: "600"
        }
      )
    ]
  })}
`
```
*Bubble size represents ridership lift above station seasonal average. Larger bubbles indicate a greater percentage increase in ridership on the event day.*

The July 15 fare increase lessened the effect somewhat, pulling the average event lift from 53% before the fare increase to 39% after, though events continued to drive noticeable increases through the end of the summer.

---


```js
const top15 = [...eventLift]
  .sort((a, b) => b.lift_pct - a.lift_pct)
  .slice(0, 15)
  .map(d => {
    const t  = Math.min(d.lift_pct / 130, 1);
    const r  = Math.round(29  + t * (30  - 29));
    const g  = Math.round(158 + t * (64  - 158));
    const bl = Math.round(117 + t * (175 - 117));
    return {
      ...d,
      label:    `${d.event_name} (${d.nearby_station})`,
      barColor: `rgb(${r},${g},${bl})`
    };
  });
```
```js
html`
    ${Plot.plot({
    width: 720,
    height: 660,
    marginLeft: 260,
    marginRight: 70,
    x: {
      label: "Ridership lift vs. station summer average (%)",
      tickFormat: d => "+" + d + "%"
    },
    y: { label: null },
    marks: [
      Plot.barX(top15, {
        x: "lift_pct",
        y: "label",
        sort: { y: "x", reverse: true },
        fill: "barColor",
        fillOpacity: 0.82,
        stroke: "barColor",
        strokeWidth: 0.5
      }),
      Plot.ruleX([0]),
      Plot.text(top15, {
        x: "lift_pct",
        y: "label",
        text: d => `+${d.lift_pct.toFixed(0)}%`,
        textAnchor: "start",
        dx: 4,
        fontSize: 11,
        fill: "#1e40af"
      }),
      Plot.tip(top15, Plot.pointer({
        x: "lift_pct",
        y: "label",
        title: d => [
          d.event_name,
          `Station: ${d.nearby_station}`,
          `Date: ${d.date.toISOString().slice(0, 10)}`,
          `Lift: +${d.lift_pct.toFixed(1)}%`,
          `Station ridership: ${d.observed.toLocaleString()}`,
          `Est. attendance: ${(+d.estimated_attendance).toLocaleString()}`
        ].join("\n")
      }))
    ]
  })}
`
```
Local events drove ridership above seasonal baselines, with an average lift of 48% across all 59 events. 

The highest impact events were over 120% above a station's typical daily ridership. 

Lower volume stations saw larger percentage spikes than major hubs for comparable events. 


<br>

---


```js
// Per-station mean response time
const stationMeanMap = new Map();
for (const d of incidents) {
  if (!stationMeanMap.has(d.station)) stationMeanMap.set(d.station, []);
  stationMeanMap.get(d.station).push(+d.response_time_minutes);
}
const stationResponseMeans = [...stationMeanMap.entries()]
  .map(([station, times]) => ({
    station,
    mean: times.reduce((s, t) => s + t, 0) / times.length
  }))
  .sort((a, b) => a.mean - b.mean);

const overallMeanResponse =
  incidents.reduce((s, d) => s + +d.response_time_minutes, 0) / incidents.length;
```
```js
html`
  ${Plot.plot({
    width: 720,
    height: 560,
    marginLeft: 185,
    marginRight: 60,
    x: {
      label: "Mean response time (minutes)",
      grid: true
    },
    y: { label: null },
    color: {
      domain: ["low", "medium", "high"],
      range: ["#1d9a6c", "#f59e0b", "#dc2626"],
      legend: true
    },
    marks: [
      Plot.barX(stationResponseMeans, {
        x: "mean",
        y: "station",
        sort: { y: "x" },
        fill: "#e2e8f0",
        stroke: "#cbd5e1",
        strokeWidth: 0.5
      }),
      Plot.dot(
        incidents,
        Plot.groupY(
          { x: "mean" },
          {
            y: "station",
            x: "response_time_minutes",
            fill: "severity",
            r: 5,
            sort: { y: "x", reduce: "mean" },
            tip: true
          }
        )
      ),
      Plot.ruleX([overallMeanResponse], {
        stroke: "#1e3a5f",
        strokeDasharray: "4,3",
        strokeWidth: 1.8
      }),
      Plot.text(
        [{ x: overallMeanResponse }],
        {
          x: "x",
          frameAnchor: "top",
          text: () => `Mean: ${overallMeanResponse.toFixed(1)} min`,
          textAnchor: "start",
          dx: 6,
          dy: 6,
          fill: "#1e3a5f",
          fontSize: 11,
          fontWeight: "600"
        }
      )
    ]
  })}
`
```

Station response times vary, splitting into two tiers. 

The fastest stations are Fulton St, Houston St, Times Sq-42 St, 34 St-Penn Station, and 86 St. They averaged around 5 minutes per incident, well below the mean of 10.2 minutes. 

The slowest stations are 59 St-Columbus Circle, West 4 St-Wash Sq, Canal St, Bowling Green, and 125 St. They averaged between 18 and 19 minutes, nearly four times slower. 

The gap is consistent across all severity levels, pointing to structural differences rather than incident type.

---

```js
const currentStaffing = {
  "Times Sq-42 St": 19, "Grand Central-42 St": 18, "34 St-Penn Station": 15,
  "14 St-Union Sq": 4, "Fulton St": 17, "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15, "Canal St": 4, "59 St-Columbus Circle": 6,
  "125 St": 7, "96 St": 19, "86 St": 19, "72 St": 10,
  "66 St-Lincoln Center": 15, "50 St": 20, "28 St": 13, "23 St": 8,
  "Christopher St": 15, "Houston St": 18, "Spring St": 12,
  "Chambers St": 18, "Wall St": 9, "Bowling Green": 6,
  "West 4 St-Wash Sq": 4, "Astor Pl": 7
};

// Aggregate upcoming events per station
const upcomingMap = new Map();
for (const e of upcoming_events) {
  if (!upcomingMap.has(e.nearby_station)) {
    upcomingMap.set(e.nearby_station, { event_count: 0, total_expected: 0 });
  }
  const s = upcomingMap.get(e.nearby_station);
  s.event_count += 1;
  s.total_expected += +e.expected_attendance;
}

// Mean response time per station from incidents
const incidentMap = new Map();
for (const d of incidents) {
  if (!incidentMap.has(d.station)) incidentMap.set(d.station, []);
  incidentMap.get(d.station).push(+d.response_time_minutes);
}
const responseAvgMap = new Map(
  [...incidentMap.entries()].map(([s, times]) => [
    s, times.reduce((a, b) => a + b, 0) / times.length
  ])
);

// Build risk table
const riskData = [...upcomingMap.entries()].map(([station, vals]) => {
  const staffing      = currentStaffing[station] ?? null;
  const response_time = responseAvgMap.get(station) ?? null;
  if (staffing == null || response_time == null) return null;
  const risk_score = (vals.total_expected / staffing) * response_time;
  return { station, ...vals, staffing, response_time, risk_score };
}).filter(Boolean);

const maxRisk = Math.max(...riskData.map(d => d.risk_score));
const riskNorm = riskData
  .map(d => ({ ...d, risk_norm: d.risk_score / maxRisk * 100 }))
  .sort((a, b) => b.risk_norm - a.risk_norm);

const top3 = new Set(riskNorm.slice(0, 3).map(d => d.station));
```
```js
html`
  ${Plot.plot({
    width: 720,
    height: 520,
    marginLeft: 185,
    marginRight: 60,
    x: {
      label: "Composite risk score (normalized)",
      grid: true,
      tickFormat: d => d + "%"
    },
    y: { label: null },
    marks: [
      Plot.barX(riskNorm, {
        x: "risk_norm",
        y: "station",
        sort: { y: "x", reverse: true },
        fill: d => top3.has(d.station) ? "#dc2626" : "#94a3b8",
        fillOpacity: 0.82
      }),
      Plot.text(riskNorm, {
        x: "risk_norm",
        y: "station",
        text: d => `${d.risk_norm.toFixed(0)}%`,
        textAnchor: "start",
        dx: 4,
        fontSize: 11,
        fill: d => top3.has(d.station) ? "#7f1d1d" : "#334155"
      }),
      Plot.tip(riskNorm, Plot.pointer({
        x: "risk_norm",
        y: "station",
        title: d => [
          d.station,
          `Events scheduled: ${d.event_count}`,
          `Total expected attendance: ${d.total_expected.toLocaleString()}`,
          `Current staffing: ${d.staffing}`,
          `Mean response time: ${d.response_time.toFixed(1)} min`,
          `Risk score: ${d.risk_norm.toFixed(1)}%`
        ].join("\n")
      }))
    ]
  })}
`
```

The risk score is (total expected attendance ÷ current staffing) × mean response time, normalized so the highest-scoring station reads as 100. Hover any bar for the full breakdown.

Canal St leads by a wide margin: 8 events, 70,000 expected attendees, only 4 staff, and an 18.4 minute mean response time. 

West 4 St-Wash Sq and 23 St follow, both carrying low staffing and slow response times against a meaningful 2026 event load. 

These three stations represent the clearest mismatch between expected demand and current operational capacity.

---



