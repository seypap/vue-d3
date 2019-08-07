<template>
  <v-layout justify-space-between>
    <v-flex xs9>
      <div class="keyword-container">
        <h1>Top Keywords</h1>
        <svg />
      </div>
    </v-flex>
    <v-flex xs3 text-xs-right>
      <div class="saved-container">
        <div class="">
          <v-btn @click="saveKeyword()">Save Keyword</v-btn>
        </div>
        <svg :height="10 + saved.length * 60" width="150" style="margin-top: 20px;">
          <g
            v-for="(keyword, index) in saved"
            v-bind:key="keyword[0]"
            :transform="`translate(0, ${20 + index * 20})`"
            style="cursor: pointer;"
            v-on:mouseover="savedMouseOver(keyword)"
            v-on:mouseout="savedMouseOut(keyword)"
          >
            <circle
              :id="'saved_circle_' + getSelector(keyword[0])"
              r="10"
              cx="23"
              :cy="23 + index * 40"
              style="fill: black; stroke: black; stroke-width: 1"
            ></circle>
            <text
              :id="'saved_text_' + getSelector(keyword[0])"
              :transform="`translate(${-10 - index * 18}, ${40 - index * 8}) rotate(-20)`"
              style="fill: black;"
            >
              <tspan v-for="(k, ind) in keyword[0].split(/[\ -]/g)" :key="k" x="60" :y="index * 50 + ind * 15" >{{ k }}</tspan>
            </text>
          </g>
        </svg>
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
  import * as d3 from 'd3'

  export default {
    data: () => ({
      elWidth: 50,
      elHeight: 100,
      animation: {
        delay: 250,
        duration: 800
      },
      svg: null,
      saved: [],
      acticeKeyword: [],
      prevActive: []
    }),
    props: {
      data: Array
    },
    mounted () {
      d3.select('.keyword-container')
        .select('svg')
        .style('width', '100%')
        .style('height', '300px')
      this.initialDraw()
    },
    watch: {
      data () {
        this.initialDraw()
      }
    },
    methods: {
      initialDraw() {
        const svg = d3.select('.keyword-container').select('svg')
        this.svg = svg
        const containerWidth = parseInt(d3.select('.keyword-container').style('width').replace('px', ''))
        const paddingLeft = 50
        const groupWidth = (containerWidth - paddingLeft * 2) / this.data.length
        const values = this.data.map(r => r[1])
        const max = Math.max(...values)

        svg.selectAll('g').remove()
        svg.selectAll('path').remove()

        this.data.forEach((kw, i) => {
          const selector = this.getSelector(kw[0])
          const group = svg.append('g')
                          .attr('id', `group_${selector}`)
                          .attr('width', groupWidth)
                          .attr('height', 300)
                          .attr('transform', `translate(${i * groupWidth}, 0)`)
                          .attr('opacity', 0)
          group.append('rect')
            .attr('width', this.elWidth)
            .attr('height', this.elHeight / max * kw[1])
            .attr('x', (groupWidth - this.elWidth) / 2)
            .attr('y', 100 - this.elHeight / max * kw[1])
            .attr('rx', 3)
            .attr('ry', 3)
            .style('stroke-width', 1)
            .style('stroke', 'black')
            .style('fill', 'grey')
          const label = group.append('text')
            .attr('x', 0)
            .attr('y', 150)
            .attr('id', `keyword_${selector}`)
            .attr('transform', `translate(-20, 0) rotate(-20)`)
            kw[0].split(/[ -]/g).forEach((txt, i) => label.append('tspan').text(txt).attr('x', 0).attr('y', 150 + i * 15))
          group.append('circle')
            .attr('id', `circle_${selector}`)
            .attr('r', 20)
            .attr('cx', groupWidth / 2)
            .attr('cy', 200)
            .style('fill', 'black')
            .style('stroke', 'black')
            .style('stroke-width', 1)
            .style('cursor', 'pointer')
            // .on('mouseover', () => this.hoverCircle(kw))
            // .on('mouseout', () => this.hoverCircle(kw, 'out'))
            .on('click', () => this.clickCircle(kw))
        })
        svg.selectAll('g')
          .transition()
          .delay(this.animation.delay)
          .duration(this.animation.duration)
          .style('opacity', 1)
      },
      // hoverCircle(item, e) {
      //   const selector = '#circle_' + this.getSelector(item[0])
      //   this.svg.selectAll('circle:not(.active)')
      //     .transition()
      //     .duration(this.animation.duration/4)
      //     .style('stroke', 'black')
      //   if(e !== 'out') {
      //     this.svg.select(selector)
      //       .transition()
      //       .duration(this.animation.duration/2)
      //       .style('stroke', 'green')
      //   }
      // },
      clickCircle(item) {
        const circle_selector = '#circle_' + this.getSelector(item[0])
        const isActive = this.svg.select(circle_selector).attr('class') === 'active'
        this.svg.selectAll('circle')
          .attr('class', '')
          .style('fill', 'black')
        this.svg.selectAll(`text`)
          .style('fill', 'black')
        this.svg.selectAll('path').remove()
        if(!isActive) {
          this.svg.select(circle_selector)
            .transition()
            .duration(this.animation.duration/2)
            .style('fill', 'green')
            .attr('class', 'active')
          this.svg.select(`#keyword_${this.getSelector(item[0])}`)
            .transition()
            .duration(this.animation.duration/2)
            .style('fill', 'green')
          this.activeKeyword = item

          const relatedKeywords = this.getRelated(item)
          relatedKeywords.forEach(relative => this.drawArc(item, relative))
        }
      },
      drawArc(a, b) {
        const selectorA = this.getSelector(a[0])
        const selectorB = this.getSelector(b[0])

        const groupA = this.svg.select(`#group_${selectorA}`)
        const baseACoord = groupA.attr('transform').replace('translate(', '').replace(/[ )]/g, '').split(',').map(parseFloat)
        const ax = parseFloat(groupA.select('circle').attr('cx')) + baseACoord[0]
        const ay = 200

        const groupB = this.svg.select(`#group_${selectorB}`)
        const baseBCoord = groupB.attr('transform').replace('translate(', '').replace(/[ )]/g, '').split(',').map(parseFloat)
        const bx = parseFloat(groupB.select('circle').attr('cx')) + baseBCoord[0]

        const x = Math.abs((ax - bx)/2)
        const radius = x * x / 40 + 1
        const alpha = Math.atan(40 * x / (x * x - 760))
        const param = {
          startAngle: -alpha,
          endAngle: alpha,
          innerRadius: radius,
          outerRadius: radius + 1
        }
        this.svg.append('path')
          .attr('d', d3.arc()(param))
          .attr('transform', `translate(${Math.min(ax, bx) + x}, ${ay - radius + 40}) rotate(180)`)
          .style('opacity', 0)
        this.svg.selectAll('path')
          .transition()
          .delay(this.animation.delay)
          .duration(this.animation.duration)
          .style('opacity', 1)
      },
      getSelector(keyword) {
        return keyword.split(/[ -]/g).join('_').replace(/&|,/g, '')
      },
      getRelated(item) {
        return this.data.filter((node) => {
            if(node[0] == item[0] && node[1] == item[1]) {
              return false
            }
            let isRelated = false
            item[0].split(' ').map(k => {
              if (node[0].indexOf(k) !== -1) {
                isRelated = true
              }
              return k
            })
            return isRelated
          })
      },
      saveKeyword() {
        if(!this.activeKeyword) {
          return
        }
        let alreadySaved = false
        this.saved.map(k => {
          if (k[0] == this.activeKeyword[0]) {
            alreadySaved = true
          }
        })
        if (!alreadySaved) {
          this.saved.push(this.activeKeyword)

          const savedSVG = d3.select('.saved-container').select('svg')
          savedSVG.selectAll('g')
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('opacity', 1)
        }
      },
      savedMouseOver(item) {
        const savedSVG = d3.select('.saved-container').select('svg')
        savedSVG.selectAll('text')
          .style('fill', 'black')
        savedSVG.selectAll('circle')
          .style('fill', 'black')
        if(savedSVG.select('#saved_text_' + this.getSelector(item[0]))) {
          savedSVG.select('#saved_text_' + this.getSelector(item[0]))
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('fill', 'green')
          savedSVG.select('#saved_circle_' + this.getSelector(item[0]))
            .transition()
            .delay(this.animation.delay)
            .duration(this.animation.duration)
            .style('fill', 'green')
          if(this.svg.select(`#circle_${this.getSelector(item[0])}`).attr('class') !== 'active') {
            this.clickCircle(item)
          }
        }
      },
      savedMouseOut() {
        const savedSVG = d3.select('.saved-container').select('svg')
        savedSVG.selectAll('text')
          .transition()
          .duration(this.animation.duration)
          .style('fill', 'black')
        savedSVG.selectAll('circle')
          .transition()
          .duration(this.animation.duration)
          .style('fill', 'black')
      }
    }
  }
</script>

<style>
.keyword-container, .saved-container {
  margin-top: 30px;
}
.keyword-container h1 {
  text-align: center;
}
</style>