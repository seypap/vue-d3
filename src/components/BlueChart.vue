<template>
  <div class="bluebar-container">
    <h1>Topic Groups</h1>
    <svg />
  </div>
</template>


<script>
  import * as d3 from 'd3'

  export default {
    data: () => ({
      elWidth: 20,
      elHeight: 100,
      animation: {
        delay: 250,
        duration: 1000
      },
      svg: null
    }),
    props: {
      data: Array,
      sortBy: Number,
      isBar: Boolean,
      showCounts: Boolean
    },
    mounted () {
      d3.select('.bluebar-container')
        .select('svg')
        .style('width', '100%')
        .style('height', '250px')
      this.initialDraw()
    },
    watch: {
      data () {
        this.initialDraw()
      },
      showCounts(newData) {
        this.displayCounts(newData)
      },
      sortBy(newData) {
        this.sortByAlphaCount(newData)
      },
      isBar(newData) {
        this.toggleShape(newData)
      }
    },
    methods: {
      initialDraw() {
        const svg = d3.select('.bluebar-container').select('svg')
        this.svg = svg
        const containerWidth = parseInt(d3.select('.bluebar-container').style('width').replace('px', ''))
        const paddingLeft = 50
        const groupWidth = (containerWidth - paddingLeft * 2) / this.data.length
        const values = this.data.map(r => r[1])
        const max = Math.max(...values)
        const min = Math.min(...values)
        const colors = d3.scaleLinear().domain([min, max]).range(['lightblue', 'darkblue'])

        svg.selectAll('g').remove()

        this.data.forEach((tg, i) => {
          const selector = tg[0].split(/[ -]/g).join('_').replace(/&|,/g, '')
          const group = svg.append('g')
                          .attr('id', `group_${selector}`)
                          .attr('width', groupWidth)
                          .attr('height', 300)
                          .attr('transform', `translate(${i * groupWidth}, 0)`)
                          .attr('opacity', 0)
          const label = group.append('text')
            .attr('x', 0)
            .attr('y', 90)
            .attr('id', `topic_${selector}`)
            .attr('transform', `translate(-30, 0) rotate(-30)`)
            tg[0].split(/[ -]/g).forEach((txt, i) => label.append('tspan').text(txt).attr('x', 0).attr('y', (110 - tg[0].split(/[ -]/g).length * 15) + i * 15))
          group.append('rect')
            .attr('width', this.elWidth)
            .attr('height', this.elHeight)
            .attr('x', (groupWidth - this.elWidth) / 2)
            .attr('y', 100)
            .attr('rx', 3)
            .attr('ry', 3)
            .style('stroke-width', 1)
            .style('stroke', 'black')
            .style('fill', '#fafafa')
          group.append('rect')
            .attr('width', this.elWidth)
            .attr('height', this.elHeight / max * tg[1])
            .attr('x', (groupWidth - this.elWidth) / 2)
            .attr('y', 100)
            .attr('rx', 3)
            .attr('ry', 3)
            .style('stroke-width', 1)
            .style('stroke', 'black')
            .style('fill', colors(tg[1]))
          group.append('circle')
            .attr('r', groupWidth / 2 - 10)
            .attr('cx', groupWidth / 2)
            .attr('cy', 150)
            .style('fill', '#fafafa')
            .style('stroke', 'black')
            .style('stroke-width', 1)
            .style('opacity', 0)
          group.append('circle')
            .attr('r', (groupWidth / 2 - 10) / max * tg[1])
            .attr('cx', groupWidth / 2)
            .attr('cy', 150)
            .style('fill', colors(tg[1]))
            .style('stroke', 'black')
            .style('stroke-width', 1)
            .style('opacity', 0)
          group.append('text')
            .attr('id', `count_${selector}`)
            .attr('class', 'counts')
            .attr('x', (groupWidth - this.elWidth) / 2)
            .attr('y', this.elHeight + 130)
            .text(tg[1])
        })
        svg.selectAll('g')
          .transition()
          .delay(this.animation.delay)
          .duration(this.animation.duration).style('opacity', 1)
      },
      displayCounts(isShow) {
        if (isShow) {
          this.svg.selectAll('.counts')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 1)
        } else {
          this.svg.selectAll('.counts')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 0)
        }
      },
      sortByAlphaCount(alphaOrCount) {
        if (!alphaOrCount) return

        const containerWidth = parseInt(d3.select('.bluebar-container').style('width').replace('px', ''))
        const paddingLeft = 50
        const groupWidth = (containerWidth - paddingLeft * 2) / this.data.length

        const sorted = this.data.map(a => a).sort((a, b) => {
          if (alphaOrCount === 2) {
            return a[1] < b[1] ? 1 : -1
          }
          return a[0] > b[0] ? 1 : -1
        })
        sorted.forEach((tg, i) => {
          const selector = tg[0].split(/[ -]/g).join('_').replace(/&|,/g, '')
          this.svg.select(`#group_${selector}`)
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .attr('transform', `translate(${i * groupWidth}, 0)`)
        })
      },
      toggleShape(shape) {
        if(shape) {
          this.svg.selectAll('circle')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 0)
          this.svg.selectAll('rect')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 1)
        } else {
          this.svg.selectAll('rect')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 0)
          this.svg.selectAll('circle')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 1)
        }
      }
    }
  }
</script>

<style>
.bluebar-container h1 {
  text-align: center;
}
</style>