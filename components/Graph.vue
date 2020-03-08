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
    putStrike: {
      type: Number,
      default: () => 32
    },
    callStrike: {
      type: Number,
      default: () => 32
    }
  },
  data() {
    return {
      svgId: 'graph',
      svg: {},
      finalStockPrice: 40,
      minDate: new Date(2020, 0, 1),
      maxDate: new Date(2020, 3, 10),
      now: new Date(2020, 1, 29),
      expiry: new Date(2020, 2, 31),
      domain: [20, 50],
      margin: { top: 30, right: 30, bottom: 30, left: 50 }
    }
  },
  computed: {
    width() {
      return 800 - this.margin.left - this.margin.right
    },
    height() {
      return 400 - this.margin.top - this.margin.bottom
    },
    x() {
      const self = this
      return d3
        .scaleTime()
        .domain([this.minDate, self.maxDate])
        .range([0, this.width])
    },
    y() {
      const self = this
      return d3
        .scaleLinear()
        .domain(self.domain)
        .range([self.height, 0])
    }
  },
  watch: {
    finalStockPrice(val) {
      this.updatePriceMovement(val)
    }
  },
  mounted() {
    this.renderChart()
  },
  methods: {
    generateData(target) {
      const ary = []
      const initPrice = 33.2
      const diff = target - initPrice
      for (let index = 31; index > 0; index--) {
        const date = new Date(2020, 2, index)
        const pos = Math.random() > 0.5 ? 1 : -1
        const noise = 1.1 * pos * Math.random() * Math.random()
        let price = initPrice + (diff * index) / 32 + noise
        if (index === 31) {
          price = target
        } else if (index === 1) {
          price = initPrice
        }
        const dataPoint = {
          date,
          price
        }
        ary.push(dataPoint)
      }
      return ary
    },
    transformDate(ary) {
      const parser = d3.utcParse('%m/%d/%Y')
      return ary.map((element) => {
        return { price: element.Close, date: parser(element.Date) }
      })
    },
    updatePriceMovement(price) {
      const self = this
      const x = self.x
      const y = self.y
      const newData = [].concat(
        self.generateData(price),
        self.transformDate(self.input)
      )
      this.svg
        .selectAll('.stockPrice')
        .datum(newData)
        .transition()
        .duration(1000)
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
    },
    renderChart() {
      const self = this
      const x = self.x
      const y = self.y
      const data = [].concat(
        self.generateData(40),
        self.transformDate(self.input)
      )
      // append the svg object to the body of the page
      this.svg = d3
        .select('#' + this.svgId)
        .attr('preserveAspectRatio', 'xMinYMin meet')
        .attr('viewBox', '0 0 720 500')
        .append('g')
        .attr(
          'transform',
          'translate(' + self.margin.left + ',' + self.margin.top + ')'
        )
      this.svg
        .append('g')
        .attr('transform', 'translate(0,' + self.height + ')')
        .call(d3.axisBottom(x).tickFormat(d3.timeFormat('%m/%d')))
      // Add Y axis
      this.svg.append('g').call(d3.axisLeft(y))
      // Add the line
      this.svg
        .append('path')
        .datum(data)
        .attr('class', 'stockPrice')
        .attr('fill', 'none')
        .attr('stroke', '#69b3a2')
        .attr('stroke-width', 1)
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
      const maxY = y(50)
      const minY = y(20)
      const drag = d3
        .drag()
        .on('start', dragstarted)
        .on('drag', dragged)
        .on('end', dragended)
      function dragstarted() {
        // d3.select(this).classed(activeClassName, true)
      }

      function dragged(selector) {
        const y = d3.event.dy

        const line = d3.select(this)
        // Update the line properties
        const attributes = {
          x1: parseInt(line.attr('x1')),
          y1: parseInt(line.attr('y1')) + y,

          x2: parseInt(line.attr('x2')),
          y2: parseInt(line.attr('y2')) + y
        }

        line.attr('y1', Math.min(Math.max(attributes.y1, maxY), minY))
        line.attr('y2', Math.min(Math.max(attributes.y2, maxY), minY))
      }

      function dragended() {
        // d3.select(this).classed(activeClassName, false)
      }

      const now = this.now
      this.svg
        .append('line')
        .attr('x1', x(now))
        .attr('y1', minY)
        .attr('x2', x(now))
        .attr('y2', maxY)
        .attr('stroke', 'black')
        .attr('stroke-width', 0.5)

      const expiry = this.expiry
      this.svg
        .append('line')
        .attr('x1', x(expiry))
        .attr('y1', minY)
        .attr('x2', x(expiry))
        .attr('y2', maxY)
        .attr('stroke', 'black')
        .attr('stroke-width', 0.5)

      this.svg
        .append('line')
        .attr('x1', x(self.minDate))
        .attr('y1', y(self.callStrike))
        .attr('x2', x(expiry))
        .attr('y2', y(self.callStrike))
        .attr('stroke', 'red')
        .attr('stroke-width', 2)
        .call(drag)
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
