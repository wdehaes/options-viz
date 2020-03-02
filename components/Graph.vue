<template>
  <div class="svg-container">
    <svg v-bind:id="svgId" class="graph"></svg>
  </div>
</template>
<script>
import * as d3 from 'd3'
export default {
  props: {
    input: {
      type: Array,
      default: () => []
    },
    svgId: {
      type: String,
      default: () => 'graph'
    },
    putStrike: {
      type: Number,
      default: () => 32
    },
    callStrike: {
      type: Number,
      default: () => 32
    },
    field: {
      type: String,
      default: () => 'Close'
    }
  },
  mounted() {
    this.renderChart()
  },
  methods: {
    transformDate(ary) {
      const parser = d3.utcParse('%m/%d/%Y')
      return this.input.map((element) => {
        return { price: element.Close, date: parser(element.Date) }
      })
    },
    renderChart() {
      const self = this
      const margin = { top: 30, right: 30, bottom: 30, left: 50 }
      const width = 600 - margin.left - margin.right
      const height = 300 - margin.top - margin.bottom

      const mindate = new Date(2019, 2, 1)
      const maxdate = new Date(2020, 2, 1)

      const data = self.transformDate(self.input)
      // append the svg object to the body of the page
      const svg = d3
        .select('#' + this.svgId)
        .attr('preserveAspectRatio', 'xMinYMin meet')
        .attr('viewBox', '0 0 460 400')
        // .attr("width", width + margin.left + margin.right)
        // .attr("height", height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')')
      const x = d3
        .scaleTime()
        .domain([mindate, maxdate])
        .range([0, width])
      svg
        .append('g')
        .attr('transform', 'translate(0,' + height + ')')
        .call(
          d3
            .axisBottom(x)
            .ticks(10)
            .tickFormat(d3.timeFormat('%d %B'))
        )
      // Add Y axis
      const y = d3
        .scaleLinear()
        .domain([20, 50])
        .range([height, 0])
      svg.append('g').call(d3.axisLeft(y))
      // Add the line
      svg
        .append('path')
        .datum(data)
        .attr('fill', 'none')
        .attr('stroke', '#69b3a2')
        .attr('stroke-width', 1.5)
        .attr(
          'd',
          d3
            .line()
            .x(function(d) {
              return x(d.date)
            })
            .y(function(d) {
              return y(d.price)
            })
        )
      // Add the points
      // svg
      //   .append('g')
      //   .selectAll('dot')
      //   .data(this.input)
      //   .enter()
      //   .append('circle')
      //   .attr('cx', function(d) {
      //     return x(d.Date)
      //   })
      //   .attr('cy', function(d) {
      //     return y(d.Close)
      //   })
      //   .attr('r', 5)
      //   .attr('fill', '#69b3a2')
    }
  }
}
</script>
<style>
.svg-container {
  display: inline-block;
  position: relative;
  width: 80%;
  vertical-align: top;
  overflow: hidden;
}
.svg-content {
  display: inline-block;
  position: absolute;
  top: 0;
  left: 0;
}
</style>
