<script setup lang="ts">
import { onMounted, ref } from 'vue';
import * as d3 from 'd3';
import PieChart from './PieChart.vue';

interface e6PostInterface {
  '_id': number,
  'created_at': number,
  'rating': string,
  'artist': Array<string>,
  'character': Array<string>
}

const selected = ref([])

var margin = {top: 10, right: 30, bottom: 30, left: 40},
    width = 600 - margin.left - margin.right,
    height = 300 - margin.top - margin.bottom;

onMounted(() => {
  // append the svg object to the body of the page
  const svg = d3.select(".histgram")
    .append("svg")
      .attr("viewBox", `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
      // .attr("width", width + margin.left + margin.right)
      // .attr("height", height + margin.top + margin.bottom)
    .append("g")
      .attr("transform",
            `translate(${margin.left}, ${margin.top})`);

  // get the data
  d3.json("/data/beastars.json").then( function(data:Array<e6PostInterface>) {

    // X axis: scale and draw:
    const x = d3.scaleTime()
        .domain([d3.min(data, function(d) { return +d.created_at }), d3.max(data, function(d) { return +d.created_at })])
        .range([0, width]);
    svg.append("g")
        .attr("transform", `translate(0, ${height})`)
        .call(d3.axisBottom(x));

    // set the parameters for the histogram
    const histogram = d3.bin()
        .value(function(d) { return d.created_at; })   // I need to give the vector of value
        .domain(x.domain())  // then the domain of the graphic
        .thresholds(x.ticks(50)); // then the numbers of bins

    // And apply this function to data to get the bins
    const bins = histogram(data);

    // Y axis: scale and draw:
    const y = d3.scaleLinear()
        .range([height, 0]);
        y.domain([0, d3.max(bins, function(d) { return d.length; })]);   // d3.hist has to be called before the Y axis obviously
    svg.append("g")
        .call(d3.axisLeft(y));

    // Add a tooltip div. Here I define the general feature of the tooltip: stuff that do not depend on the data point.
    // Its opacity is set to 0: we don't see it by default.
    const tooltip = d3.select(".histgram")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      .style("background-color", "black")
      .style("color", "white")
      .style("border-radius", "5px")
      .style("padding", "10px")

    // A function that change this tooltip when the user hover a point.
    // Its opacity is set to 1: we can now see it. Plus it set the text and position of tooltip depending on the datapoint (d)
    const showTooltip = function(event,d:Array<e6PostInterface>) {
      tooltip
        .transition()
        .duration(100)
        .style("opacity", 1)
      tooltip
        .html("Explicit: " + d.filter(x => x.rating == "e").length + 
              "\nQuestionable: " + d.filter(x => x.rating == "q").length + 
              "\nSafe: " + d.filter(x => x.rating == "s").length)
        .style("left", (event.x)/2-100 + "px")
        .style("top", (event.y)/2 + "px")
    }
    const moveTooltip = function(event,d) {
      tooltip
      .style("left", (event.x)/2-100 + "px")
      .style("top", (event.y)/2 + "px")
    }
    // A function that change this tooltip when the leaves a point: just need to set opacity to 0 again
    const hideTooltip = function(event,d) {
      tooltip
        .transition()
        .duration(100)
        .style("opacity", 0)
    }

    const showDetail = function(event,d) {
      selected.value = d;
    }

    // append the bar rectangles to the svg element
    svg.selectAll("rect")
        .data(bins)
        .join("rect")
          .attr("x", 1)
          .attr("transform", function(d) { return `translate(${x(d.x0)}, ${y(d.length)})`})
          .attr("width", function(d) { return x(d.x1) - x(d.x0) -1 ; })
          .attr("height", function(d) { return height - y(d.length); })
          .style("fill", "#69b3a2")
          // Show tooltip on hover
          .on("mouseover", showTooltip )
          .on("mousemove", moveTooltip )
          .on("mouseleave", hideTooltip )
          .on("click", showDetail)
  });
});

</script>

<template>
  <div>
    <div class="histgram"></div>
  </div>
  <div>
    <div v-if="selected.length > 0">
      <PieChart :chart-data="selected" />
    </div>
    <div v-else>
      Select a bar for more info
    </div>
  </div>
</template>

<style>

</style>
