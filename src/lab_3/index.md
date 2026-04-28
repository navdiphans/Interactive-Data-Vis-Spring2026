---
title: "Lab 3: Mayoral Mystery"
toc: false
theme: slate
---

## Lab 3: Mayoral Mystery

<br>

In the 2024 New York City mayoral election, the candidate entered the race strong but fell short in the end, losing by around 31,000 votes citywide. The campaign collected data across all five boroughs to understand what worked, what did not, and where opportunities exist for any future mayoral run.

The analysis below examines results by district and income level, the effectiveness of the Get Out the Vote (GOTV) operation, and what survey respondents said about the candidate's policy positions.

```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
```

```js
const districts = topojson.feature(nyc, nyc.objects.districts);
```

```js
const voteShareByDistrict = new Map(
  results.map(d => [d.boro_cd, d.votes_candidate / (d.votes_candidate + d.votes_opponent)])
);

const districtsWithData = {
  type: districts.type,
  features: districts.features.map(f => ({
    ...f,
    properties: {
      ...f.properties,
      vote_share: voteShareByDistrict.get(f.properties.BoroCD) ?? null
    }
  }))
};

const resultsWithShare = results.map(d => ({
  ...d,
  vote_share: d.votes_candidate / (d.votes_candidate + d.votes_opponent)
}));

const issueLabels = {
  affordable_housing_alignment: "Affordable Housing",
  public_transit_alignment: "Public Transit",
  childcare_support_alignment: "Childcare",
  small_business_tax_alignment: "Small Business Tax",
  police_reform_alignment: "Police Reform"
};

const issues = Object.keys(issueLabels);

const votedFor = survey.filter(d => d.voted === "Yes" && (d.voted_for === "Candidate" || d.voted_for === "Opponent"));

const alignmentRows = [];
for (const issue of issues) {
  for (const group of ["Candidate", "Opponent"]) {
    const vals = votedFor.filter(d => d.voted_for === group).map(d => d[issue]);
    const mean = vals.reduce((a, b) => a + b, 0) / vals.length;
    alignmentRows.push({ issue: issueLabels[issue], group, mean });
  }
}
```




```js
const selectedDistrict = Inputs.select(
  ["None", ...results.map(d => d.boro_cd).sort((a, b) => a - b)],
  { label: "Highlight District", value: "None" }
);
const selectedDistrictChoice = Generators.input(selectedDistrict);
```



```js
const selectedRow = results.find(d => d.boro_cd === selectedDistrictChoice);
```

<br>

## Where Did the Candidate Win and Lose?

The candidate received 662,280 votes to their opponent's 693,403 votes, losing the overall vote 48.9% to 51.1% by a margin of 31,123 votes. 

Despite winning 36 of 59 community districts, the loss was driven by decisive defeats in high-turnout, high-income districts that outweighed the candidate's strength in the Bronx and Brooklyn. High income districts outweighed the Bronx and Brooklyn strength by nearly 50,000 votes. Breaking even in just those 9 districts would have flipped the entire race.

The map below shows vote share by district: blue districts went to the candidate, red districts to the opponent. The Bronx and Brooklyn were reliable bases of support, while Manhattan, Queens, and Staten Island went largely against the candidate. 

**Every high-income district in the dataset went to the opponent**, and those districts averaged turnout above 81%, the highest of any income tier, making them decisive.

<br>

```js
display(selectedDistrict);
```

```js
if (selectedRow) {
  const voteShare = selectedRow.votes_candidate / (selectedRow.votes_candidate + selectedRow.votes_opponent);
  const won = selectedRow.votes_candidate > selectedRow.votes_opponent;
  display(html`
    <div style="margin: 8px 0 16px 0; padding: 12px 16px; background: #1a1a2e; border-left: 4px solid ${won ? "#2980b9" : "#c0392b"}; border-radius: 4px; font-size: 14px; line-height: 1.8;">
      <strong>District ${selectedRow.boro_cd}</strong> &nbsp;|&nbsp;
      ${selectedRow.income_category} income &nbsp;|&nbsp;
      ${won ? "✓ Candidate won" : "✗ Candidate lost"}<br>
      Vote share: <strong>${(voteShare * 100).toFixed(1)}%</strong> &nbsp;|&nbsp;
      Turnout: <strong>${selectedRow.turnout_rate.toFixed(1)}%</strong> &nbsp;|&nbsp;
      Doors knocked: <strong>${selectedRow.gotv_doors_knocked.toLocaleString()}</strong> &nbsp;|&nbsp;
      Candidate hours: <strong>${selectedRow.candidate_hours_spent}</strong>
    </div>
  `);
}
```

```js
const projection = d3.geoMercator().fitSize([950, 750], districts);

const mapPlot = Plot.plot({
  title: "Candidate Vote Share by NYC Community District",
  width: 950,
  height: 750,
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    type: "linear",
    domain: [0.25, 0.5, 0.75],
    range: ["#c0392b", "#f5f5f5", "#2980b9"],
    label: "Candidate Vote Share",
    legend: true
  },
  marks: [
    Plot.geo(districtsWithData, {
      fill: d => d.properties.vote_share,
      stroke: d => d.properties.BoroCD === selectedDistrictChoice ? "yellow" : "white",
      strokeWidth: d => d.properties.BoroCD === selectedDistrictChoice ? 3 : 0.5,
      title: d =>
        d.properties.vote_share != null
          ? `District ${d.properties.BoroCD}\nVote share: ${(d.properties.vote_share * 100).toFixed(1)}%`
          : `District ${d.properties.BoroCD}\nNo data`
    })
  ]
});

mapPlot.addEventListener("click", (event) => {
  const rect = mapPlot.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  for (const feature of districtsWithData.features) {
    if (d3.geoContains(feature, projection.invert([x, y]))) {
      selectedDistrict.value = feature.properties.BoroCD;
      selectedDistrict.dispatchEvent(new Event("input", { bubbles: true }));
      break;
    }
  }
});

display(mapPlot);
```

<br>
<br>

## Did the GOTV Campaign Make a Difference?

The campaign knocked on doors across all 59 districts, but effort was distributed unevenly. Districts that received the most GOTV attention showed higher candidate vote share. 

The top 10 districts by doors knocked averaged 58.4% vote share for the candidate; the bottom 10 averaged just 36.4%. The correlation between doors knocked and vote share is 0.70, one of the strongest relationships in the data. Middle-income districts received minimal GOTV effort yet still returned some competitive results suggesting potential support that a stronger ground game could have converted.

<br>

```js
Plot.plot({
  title: "GOTV Doors Knocked vs. Candidate Vote Share by Income Level",
  width: 900,
  height: 700,
  marginLeft: 60,
  marginBottom: 50,
  x: {
    label: "Doors Knocked (GOTV)",
    grid: true
  },
  y: {
    label: "Candidate Vote Share",
    tickFormat: "%",
    domain: [0.2, 0.8],
    grid: true
  },
  color: {
    legend: true,
    domain: ["Low", "Middle", "High"],
    range: ["#CD7F32", "#C0C0C0", "#4FC3F7"],
    label: "Income Category"
  },
  marks: [
    Plot.ruleY([0.5], { stroke: "white", strokeDasharray: "4,3", strokeOpacity: 0.4 }),
    Plot.dot(resultsWithShare, {
      x: "gotv_doors_knocked",
      y: "vote_share",
      fill: "income_category",
      r: 5,
      opacity: 0.85,
      tip: true,
      title: d => `District ${d.boro_cd}\nIncome: ${d.income_category}\nDoors knocked: ${d.gotv_doors_knocked.toLocaleString()}\nVote share: ${(d.vote_share * 100).toFixed(1)}%`
    })
  ]
})
```

<br>

## What Did Voters Think of the Issues?

Post-election survey responses reveal a candidate whose policy platform was broadly popular except on one issue. 

Across affordable housing, public transit, and childcare, respondents who voted for the candidate scored alignment notably higher than those who voted for the opponent. 

However, **police reform** alignment was low across the board, averaging just 1.43 out of 5 among all respondents, with no one scoring it above 2. Open-ended responses repeatedly named police reform as a dealbreaker even among voters who liked everything else about the candidate. 

<br>

```js
Plot.plot({
  title: "Average Issue Alignment Score by Vote Choice",
  width: 800,
  height: 600,
  marginLeft: 140,
  marginRight: 160,
  marginBottom: 50,
  x: {
    label: "Average Alignment Score (1=Strongly Disagree, 5=Strongly Agree)",
    domain: [0, 5],
    grid: true
  },
  y: {
    label: null
  },
  fy: {
    label: null
  },
  color: {
    legend: true,
    domain: ["Candidate", "Opponent"],
    range: ["#2980b9", "#c0392b"],
    label: "Voted For"
  },
  marks: [
    Plot.ruleX([0]),
    Plot.barX(alignmentRows, {
      x: "mean",
      y: "group",
      fy: "issue",
      fill: "group",
      tip: true,
      title: d => `${d.issue}\nVoted for: ${d.group}\nAvg score: ${d.mean.toFixed(2)}`
    })
  ]
})
```

<br>

## Where Did the Candidate Spend Time / Was It Helpful?

The candidate's personal time was focused very significantly in high-income districts, which averaged 22.8 hours of candidate presence per district. Those same districts returned an average vote share of just 28.9% which was the worst performance of any income tier. 

Middle-income districts averaged fewer than 2 hours of candidate time **but still produced nearly 49.2% vote share.** 

Low-income districts, the candidate's strongest base at 58.4% vote share, received a modest 12.3 hours on average. The data suggests the candidate invested the most time where support was structurally unavailable, while underleveraging districts where the race was competitive.

<br>

```js
Plot.plot({
  title: "Candidate Hours Spent vs. Vote Share by Income Level",
  width: 900,
  height: 700,
  marginLeft: 60,
  marginBottom: 50,
  x: {
    label: "Candidate Hours Spent in District",
    grid: true
  },
  y: {
    label: "Candidate Vote Share",
    tickFormat: "%",
    domain: [0.2, 0.8],
    grid: true
  },
  color: {
    legend: true,
    domain: ["Low", "Middle", "High"],
    range: ["#CD7F32", "#C0C0C0", "#4FC3F7"],
    label: "Income Category"
  },
  marks: [
    Plot.ruleY([0.5], { stroke: "white", strokeDasharray: "4,3", strokeOpacity: 0.4 }),
    Plot.dot(resultsWithShare, {
      x: "candidate_hours_spent",
      y: "vote_share",
      fill: "income_category",
      r: 5,
      opacity: 0.85,
      tip: true,
      title: d => `District ${d.boro_cd}\nIncome: ${d.income_category}\nHours spent: ${d.candidate_hours_spent}\nVote share: ${(d.vote_share * 100).toFixed(1)}%`
    })
  ]
})
```

<br>
<br>

---

<div style="margin-top: 3rem; padding: 2rem; background: #0d1117; border: 1px solid #2a3a4a; border-radius: 6px; font-family: Arial, sans-serif; color: #cdd9e5;">

<div style="text-align: center; margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 2px solid #2e75b6;">
  <div style="font-size: 22px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.4rem;">2024 New York City Mayoral Campaign</div>
  <div style="font-size: 18px; color: #8ba7bf; margin-bottom: 0.8rem;">Post-Election Analysis and Recommendations</div>
  <div style="font-size: 16px; color: #8ba7bf; font-style: italic;">Prepared for the Campaign Committee &nbsp;|&nbsp; April 2026</div>
</div>

<div style="margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #2a3a4a;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">Executive Summary</div>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 0.75rem;">The candidate entered the 2024 New York City mayoral race with great potential and a policy platform that resonated broadly with working-class and middle-income New Yorkers. 
  
  The final result, a 31,123-vote defeat on 48.9% of the overall vote, does not reflect a failed campaign. Instead, it reflects a winnable race with **three correctable structural problems**: a misallocated field schedule, an underperforming Get Out the Vote operation in competitive districts, and a single policy position that functioned as a dealbreaker for an otherwise persuadable city electorate.</p>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0;">This report presents findings from election results across 59 community districts, a post-election survey of 100 registered voters, and campaign event data covering May through November 2024. All findings are data-driven. Recommendations are specific and actionable.</p>
</div>

<div style="margin-bottom: 1rem;">
  <table style="width: 100%; border-collapse: collapse; font-size: 15px;">
    <thead>
      <tr style="background: #1f3864;">
        <th style="padding: 8px 10px; text-align: left; color: #ffffff; border: 1px solid #2a3a4a;">Metric</th>
        <th style="padding: 8px 10px; text-align: center; color: #ffffff; border: 1px solid #2a3a4a;">Result</th>
        <th style="padding: 8px 10px; text-align: left; color: #ffffff; border: 1px solid #2a3a4a;">Context</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: #111820;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Total votes — Candidate</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">662,280</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">48.9% of total votes cast</td>
      </tr>
      <tr style="background: #0d1117;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Total votes — Opponent</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">693,403</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">51.1% of total votes cast</td>
      </tr>
      <tr style="background: #111820;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Margin of defeat</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">31,123</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">Closeable with targeted changes</td>
      </tr>
      <tr style="background: #0d1117;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Community districts won</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">36 of 59</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">Won majority of districts, lost citywide</td>
      </tr>
      <tr style="background: #111820;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Low-income districts won</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">26 of 26</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">Dominant performance in base</td>
      </tr>
      <tr style="background: #0d1117;">
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">High-income districts won</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">0 of 9</td>
        <td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; font-style: italic;">Structural loss, not messaging failure</td>
      </tr>
    </tbody>
  </table>
</div>

<div style="margin-top: 2rem; padding-top: 1.5rem; border-top: 1px solid #2a3a4a; margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #2a3a4a;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">1. Geographic Performance</div>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 1rem;">The candidate's coalition was geographically concentrated but not broad. The Bronx and Brooklyn delivered strong results, with 11 of 12 and 15 of 18 districts won respectively. Manhattan, Queens, and Staten Island went substantially against the candidate.</p>
  <table style="width: 100%; border-collapse: collapse; font-size: 15px; margin-bottom: 1rem;">
    <thead>
      <tr style="background: #1f3864;">
        <th style="padding: 8px 10px; text-align: left; color: #fff; border: 1px solid #2a3a4a;">Borough</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Districts</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Won</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Vote Share</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Avg Turnout</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Assessment</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Bronx</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">12</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">11</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">57.6%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">59.1%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; text-align: center;">Strong base</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Brooklyn</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">18</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">15</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">54.8%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">55.4%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; text-align: center;">Strong base</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Queens</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">14</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">6</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #e6a020; font-weight: bold; text-align: center;">45.0%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">51.4%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #e6a020; text-align: center;">Competitive</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Manhattan</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">12</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">4</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">40.0%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">65.2%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">Difficult terrain</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Staten Island</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">0</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">38.0%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">60.4%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">Non-competitive</td></tr>
    </tbody>
  </table>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0;"><strong style="color: #4fc3f7;">Key Finding:</strong> Manhattan's high turnout rate of 65.2% combined with a low vote share of 40.0% means the candidate was losing high-participation voters at a significant rate. Efforts to effectively convert Manhattan are unlikely to yield returns proportional to campaign investment.</p>
</div>

<div style="margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #2a3a4a;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">2. Performance by Income Level</div>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 1rem;">Income tier is the single strongest predictor of district-level outcomes. The candidate swept every low-income district and lost every high-income district. Middle-income districts are the strategic battleground.</p>
  <table style="width: 100%; border-collapse: collapse; font-size: 15px; margin-bottom: 1rem;">
    <thead>
      <tr style="background: #1f3864;">
        <th style="padding: 8px 10px; text-align: left; color: #fff; border: 1px solid #2a3a4a;">Income Tier</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Districts</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Won</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Vote Share</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Avg Turnout</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Avg Hours</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Avg Doors</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Low (&lt;$50K)</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">26</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">26</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">58.4%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">60.6%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">12.3 hrs</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">6,938</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Middle ($50K–$100K)</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">24</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #e6a020; font-weight: bold; text-align: center;">10</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #e6a020; font-weight: bold; text-align: center;">49.2%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">45.0%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">1.96 hrs</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">611</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">High (&gt;$100K)</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">9</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">0</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">28.9%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">81.6%</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">22.8 hrs</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">340</td></tr>
    </tbody>
  </table>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0;"><strong style="color: #4fc3f7;">Critical Finding:</strong> The candidate spent an average of 22.8 hours in each high-income district while receiving only 28.9% vote share there. Middle-income districts received fewer than 2 hours of candidate time but produced 49.2% vote share. This is the most actionable finding in the data.</p>
</div>

<div style="margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #2a3a4a;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">3. Get Out the Vote Operation</div>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 1rem;">The GOTV operation produced the strongest quantitative signal in the data. The correlation between doors knocked and candidate vote share is 0.70. The problem is not the effectiveness of doors knocked but the distribution of the effort.</p>
  <table style="width: 100%; border-collapse: collapse; font-size: 15px; margin-bottom: 1rem;">
    <thead>
      <tr style="background: #1f3864;">
        <th style="padding: 8px 10px; text-align: left; color: #fff; border: 1px solid #2a3a4a;">GOTV Metric</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Finding</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Correlation: doors knocked vs. vote share</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">0.70 (strong positive)</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Top 10 districts by doors knocked - avg vote share</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; font-weight: bold; text-align: center;">58.4%</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Bottom 10 districts by doors knocked - avg vote share</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">36.4%</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Difference between top and bottom GOTV districts</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">22 percentage points</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Avg doors knocked - Low income districts</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold; text-align: center;">6,938</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Avg doors knocked - Middle income districts</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #e6a020; font-weight: bold; text-align: center;">611</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Avg doors knocked - High income districts</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold; text-align: center;">340</td></tr>
    </tbody>
  </table>
</div>

<div style="margin-bottom: 2rem; padding-bottom: 1.5rem; border-bottom: 1px solid #2a3a4a;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">4. Voter Survey Analysis</div>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 1rem;">The post-election survey of 100 registered voters reveals a candidate whose policy platform was broadly popular except on one issue.</p>
  <table style="width: 100%; border-collapse: collapse; font-size: 15px; margin-bottom: 1rem;">
    <thead>
      <tr style="background: #1f3864;">
        <th style="padding: 8px 10px; text-align: left; color: #fff; border: 1px solid #2a3a4a;">Policy Issue</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">All Respondents</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Candidate Voters</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Opponent Voters</th>
        <th style="padding: 8px 10px; text-align: center; color: #fff; border: 1px solid #2a3a4a;">Assessment</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Public Transit</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center; font-weight: bold;">3.92 / 5</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">4.00</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.61</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; text-align: center;">Strength</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Affordable Housing</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center; font-weight: bold;">3.89 / 5</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.88</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.61</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; text-align: center;">Strength</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Childcare Support</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center; font-weight: bold;">3.60 / 5</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.85</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">2.94</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #4fc3f7; text-align: center;">Differentiator</td></tr>
      <tr style="background: #0d1117;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5;">Small Business Tax</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center; font-weight: bold;">3.22 / 5</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.31</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; text-align: center;">3.00</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #8ba7bf; text-align: center;">Neutral</td></tr>
      <tr style="background: #111820;"><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #cdd9e5; font-weight: bold;">Police Reform</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center; font-weight: bold;">1.43 / 5</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">1.50</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; text-align: center;">1.42</td><td style="padding: 7px 10px; border: 1px solid #2a3a4a; color: #c0392b; font-weight: bold; text-align: center;">Critical liability</td></tr>
    </tbody>
  </table>
  <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0;"><strong style="color: #c0392b;">Police Reform Finding:</strong> Every single survey respondent scored police reform alignment at 1 or 2 out of 5. No respondent scored it above 2. Open-ended responses repeatedly named it as a dealbreaker even among voters who supported the rest of the platform. This is not a messaging problem. The position itself is the liability.</p>
</div>

<div style="margin-bottom: 2rem;">
  <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 1px;">5. Strategic Recommendations</div>

  <div style="margin-bottom: 1.25rem; padding: 1rem; background: #111820; border-left: 4px solid #4fc3f7; border-radius: 3px;">
    <div style="font-size: 16px; font-weight: bold; color: #4fc3f7; margin-bottom: 0.4rem;">Recommendation 1: Revise the Police Reform Position</div>
    <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 0.5rem;">This is the highest-priority recommendation. The current position is losing voters the candidate cannot afford to lose. The campaign should commission qualitative research to identify a modified position that retains progressive values while reducing the alienation effect documented in the survey data. If the position cannot be substantively modified, it should be deprioritized in public-facing messaging and not featured in advertising, debates, or candidate-led events unless directly challenged.</p>
    <ul style="font-size: 15px; color: #8ba7bf; margin: 0; padding-left: 1.2rem; line-height: 1.8;">
      <li>Police reform is the only issue where candidate and opponent voters are statistically identical, meaning the position provides no electoral benefit while generating documented defections.</li>
      <li>Conduct focus groups in Queens and Brooklyn middle-income districts to test alternative framings before a future campaign launch.</li>
    </ul>
  </div>

  <div style="margin-bottom: 1.25rem; padding: 1rem; background: #111820; border-left: 4px solid #e6a020; border-radius: 3px;">
    <div style="font-size: 16px; font-weight: bold; color: #e6a020; margin-bottom: 0.4rem;">Recommendation 2: Reallocate Candidate Time to Middle-Income Districts</div>
    <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 0.5rem;">The candidate spent an average of 22.8 hours in each high-income district while receiving only 28.9% vote share. Middle-income districts received 1.96 hours of candidate presence but still produced a 49.2% vote share. Shifting even half of planned high-income hours to competitive middle-income districts is the single highest-leverage scheduling change available.</p>
    <ul style="font-size: 15px; color: #8ba7bf; margin: 0; padding-left: 1.2rem; line-height: 1.8;">
      <li>Priority: the 14 middle-income districts that were lost with vote shares between 45% and 50%, where the gap is closeable with candidate presence alone.</li>
      <li>Candidate hours in high-income districts showed a negative correlation with vote share (-0.37), suggesting diminishing or counterproductive returns in that terrain.</li>
    </ul>
  </div>

  <div style="padding: 1rem; background: #111820; border-left: 4px solid #27ae60; border-radius: 3px;">
    <div style="font-size: 16px; font-weight: bold; color: #27ae60; margin-bottom: 0.4rem;">Recommendation 3: Expand and Redistribute the GOTV Operation</div>
    <p style="font-size: 16px; line-height: 1.8; color: #cdd9e5; margin: 0 0 0.5rem;">The 0.70 correlation between doors knocked and vote share is the strongest signal in the dataset. The problem is distribution, not effectiveness. Low-income districts averaged 6,938 doors knocked. Middle-income districts averaged only 611. The 22-point gap in vote share between the top and bottom GOTV districts demonstrates the stakes.</p>
    <ul style="font-size: 15px; color: #8ba7bf; margin: 0; padding-left: 1.2rem; line-height: 1.8;">
      <li>Target a minimum of 2,500 doors per middle-income district, with highest priority given to Bronx and Brooklyn districts where base support exists but turnout is low.</li>
      <li>25% of survey respondents had not heard of the candidate. Early visibility investment before the traditional campaign season could convert non-aware non-voters in middle-income districts.</li>
    </ul>
  </div>
</div>

