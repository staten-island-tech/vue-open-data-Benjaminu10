<template>
    <div>
<h1>NYC Shootings (2006-Present)</h1>
<svg ref="chart"></svg>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as d3 from 'd3'

const data = ref(null)
const chart = ref(null)

async function getData() {
    try {
        const response = await fetch("https://data.cityofnewyork.us/resource/5ucz-vwe8.json?$limit=23000")
        const jsonData = await response.json()
        data.value = jsonData
        console.log(data.value)
    } catch (error) {
        console.log(error)
    }
}

function createChart() {
    const svg = d3.select(chart.value)

    const width = 800
    const height = 400
    const margin = { top: 20, right: 20, bottom: 50, left: 60 }

    svg
        .attr("width", width)
        .attr("height", height)

    const grouped = d3.rollup(
        data.value,
        v => v.length,
        d => d.boro || "Unknown"
    )

    const dataset = Array.from(grouped, ([key, value]) => ({
        name: key,
        value: value
    }))

/* [
  {
    "incident_key": "297623042",
    "occur_date": "2024-12-06T00:00:00.000",
    "occur_time": "13:06:00",
    "boro": "MANHATTAN",
    "loc_of_occur_desc": "OUTSIDE",
    "precinct": "1",
    "jurisdiction_code": "0",
    "loc_classfctn_desc": "STREET",
    "x_coord_cd": "983437",
    "y_coord_cd": "201643",
    "latitude": "40.720149",
    "longitude": "-74.002931"
  }
]
 */

    const x = d3.scaleBand()
        .domain(dataset.map(d => d.name))
        .range([margin.left, width - margin.right])
        .padding(0.2)

    const y = d3.scaleLinear()
        .domain([0, d3.max(dataset, d => d.value)])
        .nice()
        .range([height - margin.bottom, margin.top])



    svg.append("g")
        .attr("transform", `translate(0, ${height - margin.bottom})`)
        .call(d3.axisBottom(x))

    svg.append("g")
        .attr("transform", `translate(${margin.left}, 0)`)
        .call(d3.axisLeft(y))



    svg.selectAll("rect")
        .data(dataset)
        .enter()
        .append("rect")
        .attr("x", d => x(d.name))
        .attr("y", d => y(d.value))
        .attr("height", d => y(0) - y(d.value))
        .attr("width", x.bandwidth())
        .attr("fill", "steelblue")
}

function createPie() {
    const height = Math.min(width, 500);
  const radius = Math.min(width, height) / 2;

  const arc = d3.arc()
      .innerRadius(radius * 0.67)
      .outerRadius(radius - 1);

  const pie = d3.pie()
      .padAngle(1 / radius)
      .sort(null)
      .value(d => d.value);

  const color = d3.scaleOrdinal()
      .domain(data.map(d => d.name))
      .range(d3.quantize(t => d3.interpolateSpectral(t * 0.8 + 0.1), data.length).reverse());

  const svg = d3.create("svg")
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [-width / 2, -height / 2, width, height])
      .attr("style", "max-width: 100%; height: auto;");

  svg.append("g")
    .selectAll()
    .data(pie(data))
    .join("path")
      .attr("fill", d => color(d.data.name))
      .attr("d", arc)
    .append("title")
      .text(d => `${d.data.name}: ${d.data.value.toLocaleString()}`);

  svg.append("g")
      .attr("font-family", "sans-serif")
      .attr("font-size", 12)
      .attr("text-anchor", "middle")
    .selectAll()
    .data(pie(data))
    .join("text")
      .attr("transform", d => `translate(${arc.centroid(d)})`)
      .call(text => text.append("tspan")
          .attr("y", "-0.4em")
          .attr("font-weight", "bold")
          .text(d => d.data.name))
      .call(text => text.filter(d => (d.endAngle - d.startAngle) > 0.25).append("tspan")
          .attr("x", 0)
          .attr("y", "0.7em")
          .attr("fill-opacity", 0.7)
          .text(d => d.data.value.toLocaleString("en-US")));

  return svg.node();
}
onMounted(async () => {
    await getData()
    createChart()

    
})


</script>

<style scoped>

</style>