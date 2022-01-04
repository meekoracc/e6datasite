<script setup lang="ts">
import { onMounted, onUpdated } from 'vue';
import * as d3 from 'd3';
import {nest} from 'd3-collection';

const props = defineProps<{ chartData: Array<any> }>()

var margin = {top: 10, right: 30, bottom: 30, left: 40},
    width = 450 - margin.left - margin.right,
    height = 450 - margin.top - margin.bottom;

// The radius of the pieplot is half the width or half the height (smallest one). I subtract a bit of margin.
const radius = Math.min(width, height) / 2;

const arc = d3.arc()
              .innerRadius(radius - 50)
              .outerRadius(radius)

// Count instances of each character
function countUnique(table:Array<string>, entry) {
  entry['character'].forEach((element:string) => {
    table.push(element);
  });
  return table;
}

onUpdated(() => {
  const data = nest().key(d => d).rollup(d => d.length).entries(props.chartData.reduce(countUnique, new Array<string>()));

  // Compute the position of each group on the pie:
  const pie = d3.pie()
    .value(function(d) {return d.value});
  const data_ready = pie(data);

  let path = d3.select(".piechart")
    .selectAll("path")
    .data(data_ready)

  let enterdata = path.enter()
                      .append("path")
                      .merge(path)
                      .attr("d", arc);

  path.transition().attrTween("d", arcTween);

  function arcTween(a) {
   var i = d3.interpolate(this._current, a);
   this._current = i(0);
   return function(t) {
      return arc(i(t));
   };
  }
  
  enterdata.exit().remove();
  path.exit().remove();
});

onMounted(() => {
  // append the svg object to the div called 'my_dataviz'
  const svg = d3.select(".piechart")
    .append("svg")
      .attr("width", width)
      .attr("height", height)
    .append("g")
      .attr("transform", `translate(${width/2}, ${height/2})`);

  // set the color scale
  const color = d3.scaleOrdinal()
    .range(["#98abc5", "#8a89a6", "#7b6888", "#6b486b", "#a05d56"])

  const data = nest().key(d => d).rollup(d => d.length).entries(props.chartData.reduce(countUnique, new Array<string>()));

  // Compute the position of each group on the pie:
  const pie = d3.pie()
    .value(function(d) {return d.value});
  const data_ready = pie(data);

  console.log(data_ready);

  // Build the pie chart: Basically, each part of the pie is a path that we build using the arc function.
  svg
    .selectAll('whatever')
    .data(data_ready)
    .join('path')
    .attr('d', arc)
    .attr('fill', function(d){ return(color(d.data.value)) })
    .attr("stroke", "black")
    .style("stroke-width", "2px")
    .style("opacity", 0.7);

});

</script>

<template>
  <div>
    <div class="piechart">

    </div>
  </div>
</template>

<style>

</style>