<template>
  <div>
    <h1>NYC Shootings (2006–Present)</h1>

    <h2>By Borough</h2>
    <svg ref="barChart"></svg>

    <h2>Map of Incidents</h2>
    <svg ref="mapChart"></svg>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as d3 from 'd3'

const data = ref([])
const barChart = ref(null)
const mapChart = ref(null)

async function getData() {
    const response = await fetch("https://data.cityofnewyork.us/resource/5ucz-vwe8.json?$limit=23000")
    data.value = await response.json()
}

function createChart() {
    const svg = d3.select(barChart.value)

    const width = 800
    const height = 400
    const margin = { top: 20, right: 20, bottom: 50, left: 60 }

    svg.attr("width", width).attr("height", height)

    const grouped = d3.rollup(
        data.value,
        v => v.length,
        d => d.boro || "Unknown"
    )

    const dataset = Array.from(grouped, ([key, value]) => ({
        name: key,
        value: value
    }))

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

async function createMap() {
    const svg = d3.select(mapChart.value)

    const width = 800
    const height = 600

    svg.attr("width", width).attr("height", height)

    const geoData = await d3.json("https://raw.githubusercontent.com/dwillis/nyc-maps/master/boroughs.geojson")

    const projection = d3.geoMercator()
        .fitSize([width, height], geoData)

    const path = d3.geoPath().projection(projection)

    svg.append("g")
        .selectAll("path")
        .data(geoData.features)
        .enter()
        .append("path")
        .attr("d", path)
        .attr("fill", "#f0f0f0")
        .attr("stroke", "#999")

    const filtered = data.value.filter(d => d.latitude && d.longitude)

    svg.append("g")
        .selectAll("circle")
        .data(filtered)
        .enter()
        .append("circle")
        .attr("cx", d => projection([+d.longitude, +d.latitude])[0])
        .attr("cy", d => projection([+d.longitude, +d.latitude])[1])
        .attr("r", 1)
        .attr("fill", "red")
        .attr("opacity", 0.3)
}

onMounted(async () => {
    await getData()
    createChart()
    await createMap()
})
</script>

<style scoped>
svg {
  display: block;
  margin-bottom: 40px;
}
</style>