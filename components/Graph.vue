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
    }
  },
  data() {
    return {
      svgId: 'graph',
      svg: {},
      finalStockPrice: 40.0,
      callStrike: 37.0,
      putStrike: 32.0,
      minDate: new Date(2020, 0, 1),
      maxDate: new Date(2020, 3, 10),
      now: new Date(2020, 1, 29),
      expiry: new Date(2020, 2, 31),
      domain: [20, 50],
      margin: { top: 30, right: 30, bottom: 30, left: 50 },
      activeDragName: 'drag-active'
    }
  },
  computed: {
    callProfit() {
      return this.round(Math.max(0, this.finalStockPrice - this.callStrike))
    },
    putProfit() {
      return this.round(Math.max(0, this.putStrike - this.finalStockPrice))
    },
    dragCircle() {
      const self = this
      function dragged(d) {
        const y = d3.event.dy
        const circle = d3.select(this)
        const attributes = {
          cx: parseInt(circle.attr('cx')),
          cy: parseInt(circle.attr('cy')) + y
        }
        const yVal = Math.min(Math.max(attributes.cy, self.maxY), self.minY)
        circle.attr('cx', attributes.cx)
        circle.attr('cy', yVal)

        // update the value in component
        const val = self.round(self.y.invert(yVal))
        self.finalStockPrice = val
        const label = d3.select('.priceLabel')
        label.attr('y', yVal + 2)
      }
      function dragstarted() {
        d3.select(this).classed(self.activeDragName, true)
      }
      function dragended() {
        d3.select(this).classed(self.activeDragName, false)
      }
      return d3
        .drag()
        .on('start', dragstarted)
        .on('drag', dragged)
        .on('end', dragended)
    },
    dragLine() {
      const self = this
      function dragged(d) {
        const y = d3.event.dy
        const line = d3.select(this)
        // Update the line properties
        const attributes = {
          x1: parseInt(line.attr('x1')),
          y1: parseInt(line.attr('y1')) + y,

          x2: parseInt(line.attr('x2')),
          y2: parseInt(line.attr('y2')) + y
        }
        const yVal = Math.min(Math.max(attributes.y1, self.maxY), self.minY)
        line.attr('y1', yVal)
        line.attr('y2', yVal)

        const classVal = this.classList[0]
        const label = d3.select('.' + classVal + 'Label')
        const name = d3.select('.' + classVal + 'Name')
        label.attr('y', yVal + 2)
        name.attr('y', yVal - 2)
        // update the value in component
        const val = self.round(self.y.invert(yVal))
        switch (classVal) {
          case 'call':
            self.callStrike = val
            label.text('strike price: $' + self.callStrike)
            break
          case 'put':
            self.putStrike = val
            label.text('strike price: $' + self.putStrike)
            break

          default:
            break
        }
      }
      function dragstarted() {
        d3.select(this).classed(self.activeDragName, true)
      }
      function dragended() {
        d3.select(this).classed(self.activeDragName, false)
      }
      return d3
        .drag()
        .on('start', dragstarted)
        .on('drag', dragged)
        .on('end', dragended)
    },
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
    },
    maxY() {
      return this.y(this.domain[1])
    },
    minY() {
      return this.y(this.domain[0])
    }
  },
  watch: {
    finalStockPrice(val) {
      this.updatePriceMovement(val)
    },
    callProfit(val) {
      this.svg.select('.callName').text('CALL profit: $' + this.callProfit)
    },
    putProfit(val) {
      this.svg.select('.putName').text('PUT profit: $' + this.putProfit)
    }
  },
  mounted() {
    this.renderChart()
  },
  methods: {
    round(n) {
      return Math.round((n + Number.EPSILON) * 100) / 100
    },
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
        .ease(d3.easeQuadInOut)
        .duration(200)
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
        .attr('viewBox', '0 0 800 400')
        .append('g')
        .attr(
          'transform',
          'translate(' + self.margin.left + ',' + self.margin.top + ')'
        )
      // add X axis
      this.svg
        .append('g')
        .attr('transform', 'translate(0,' + self.height + ')')
        .call(d3.axisBottom(x).tickFormat(d3.timeFormat('%m/%d')))

      this.svg
        .append('text')
        .attr(
          'transform',
          'translate(' + self.width * 0.99 + ' ,' + self.height * 0.99 + ')'
        )
        .attr('font-size', '12px')
        .style('text-anchor', 'middle')
        .text('Date')

      // Add Y axis
      this.svg.append('g').call(d3.axisLeft(y))
      this.svg
        .append('text')
        .attr('transform', 'translate(' + 35 + ' ,' + 5 + ')')
        .attr('font-size', '12px')
        .style('text-anchor', 'middle')
        .text('Stock Price')

      // Add stock price line
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

      const now = this.now
      const expiry = this.expiry

      // mark Today
      this.svg
        .append('line')
        .attr('x1', x(now))
        .attr('y1', self.minY)
        .attr('x2', x(now))
        .attr('y2', self.maxY)
        .attr('stroke', 'black')
        .attr('stroke-width', 0.5)
      this.svg
        .append('text')
        .attr('x', x(now) + 20)
        .attr('y1', self.minY)
        .attr('font-size', '12px')
        .style('text-anchor', 'middle')
        .text('Today')

      // mark expiry date
      this.svg
        .append('line')
        .attr('x1', x(expiry))
        .attr('y1', self.minY)
        .attr('x2', x(expiry))
        .attr('y2', self.maxY)
        .attr('stroke', 'black')
        .attr('stroke-width', 0.5)
      this.svg
        .append('text')
        .attr('x', x(expiry) + 45)
        .attr('y1', self.minY)
        .attr('font-size', '12px')
        .style('text-anchor', 'middle')
        .text('Expiration date')

      // draw circle + label to drag final stock price
      this.svg
        .append('circle')
        .attr('cx', x(expiry))
        .attr('cy', y(self.finalStockPrice))
        .attr('r', 5)
        .style('opacity', 1.0)
        .style('fill', '#69b3a2')
        .style('opacity', 0.6)
        .call(self.dragCircle)
      this.svg
        .append('text')
        .attr('class', 'priceLabel')
        .attr('x', x(expiry) + 5)
        .attr('y', y(self.finalStockPrice) + 2)
        .attr('font-size', '10px')
        .attr('fill', '#69b3a2')
        .text('stock price: $' + self.finalStockPrice)

      // draw SVG elements for call price
      this.svg
        .append('line')
        .attr('class', 'call')
        .attr('x1', x(self.minDate))
        .attr('y1', y(self.callStrike))
        .attr('x2', x(expiry))
        .attr('y2', y(self.callStrike))
        .attr('stroke', '#734B5E')
        .attr('stroke-width', 2)
        .style('opacity', 0.6)
        .call(self.dragLine)
      this.svg
        .append('text')
        .attr('class', 'callLabel')
        .attr('x', x(expiry) + 5)
        .attr('y', y(self.callStrike) + 2)
        .attr('font-size', '10px')
        .attr('fill', '#734B5E')
        .text('strike price: $' + self.callStrike)
      this.svg
        .append('text')
        .attr('class', 'callName')
        .attr('x', x(expiry) - 5)
        .attr('y', y(self.callStrike) - 2)
        .attr('font-size', '16px')
        .attr('fill', '#734B5E')
        .style('text-anchor', 'end')
        .text('CALL profit: $' + self.callProfit)

      // draw SVG elements for put price
      this.svg
        .append('line')
        .attr('class', 'put')
        .attr('x1', x(self.minDate))
        .attr('y1', y(self.putStrike))
        .attr('x2', x(expiry))
        .attr('y2', y(self.putStrike))
        .attr('stroke', '#335C81')
        .attr('stroke-width', 2)
        .style('opacity', 0.6)
        .call(self.dragLine)
      this.svg
        .append('text')
        .attr('class', 'putLabel')
        .attr('x', x(expiry) + 5)
        .attr('y', y(self.putStrike) + 2)
        .attr('font-size', '10px')
        .attr('fill', '#335C81')
        .text('strike price: $' + self.putStrike)
      this.svg
        .append('text')
        .attr('class', 'putName')
        .attr('x', x(expiry) - 5)
        .attr('y', y(self.putStrike) - 2)
        .attr('font-size', '16px')
        .attr('fill', '#335C81')
        .style('text-anchor', 'end')
        .text('PUT profit: $' + self.putProfit)
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

.drag-active {
  opacity: 1 !important;
}
</style>
