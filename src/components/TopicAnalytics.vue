<template>
  <v-container class="main">
    <v-layout justify-space-between>
      <v-flex xs12 md4>
        <v-text-field placeholder="Search" @keydown.enter="searchByKeyword($event)" append-icon="fas fa-search"></v-text-field>
      </v-flex>
      <v-flex xs12 md6 text-xs-right>
        <v-btn @click="sortByAlphaCount()">Sort Alpha/Count</v-btn>
        <v-btn @click="toggleShape()">View Circle/Bar</v-btn>
        <v-btn @click="toggleCount()">Show/Hide Count</v-btn>
      </v-flex>
    </v-layout>
    <BlueChart
      v-bind:data="topicsGroup"
      v-bind:sortBy="sortBy"
      v-bind:isBar="isBar"
      v-bind:showCounts="showCounts"
    />
    <div class="empty-area"></div>
    <KeywordChart
      v-bind:data="keywords"
    />
  </v-container>
</template>

<script>
  import * as d3 from 'd3'
  import BlueChart from './BlueChart'
  import KeywordChart from './KeywordChart'

  export default {
    components: {
      BlueChart,
      KeywordChart
    },
    data: () => ({
      data: [],
      topicsGroup: [],
      keywords: [],
      sortBy: 0,
      isBar: true,
      showCounts: true
    }),
    created () {
      window.d3 = d3
      this.fetchData()
    },
    mounted () {

    },
    methods: {
      fetchData () {
        d3.json('/data.json')
          .then(res => {
            this.data = res
            this.topicsGroup = this.getTopicsGroupData(res)
            this.keywords = this.getKeywordsData(res)
          })
      },
      sortByAlphaCount() {
        this.sortBy = this.sortBy === 0 ? 1 : 3 - this.sortBy
      },
      toggleCount() {
        this.showCounts = !this.showCounts
      },
      toggleShape() {
        this.isBar = !this.isBar
      },
      getTopicsGroupData(input) {
        const topicGroups = input.reduce((acc, cur) => {
          cur['Topic Groups'].forEach(topic => {
            if (acc[topic]) {
              acc[topic] += 1
            } else {
              acc[topic] = 1
            }
          })

          return acc
        }, {})
        return Object.keys(topicGroups).map(tg => [tg, topicGroups[tg]])
      },
      getKeywordsData(input, filter) {
        const filtered = !filter ? input : input.filter(row => row['Keywords'].join(',').indexOf(filter) !== -1)
        const keywords = filtered.reduce((acc, cur) => {
          cur['Keywords'].forEach(keyword => {
            const kw = keyword.toLowerCase().replace('- ', '')
            if (acc[kw]) {
              acc[kw] += 1
            } else {
              acc[kw] = 1
            }
          })

          return acc
        }, {})

        return Object.keys(keywords).map(k => [k, keywords[k]]).sort((a, b) => a[1] < b[1] ? 1 : -1).slice(0, 12)
      },
      searchByKeyword(e) {
        const key = e.target.value.trim()
        // mimic backend api call
        d3.json(`/data.json?key=${key}`)
          .then(res => {
            this.keywords = this.getKeywordsData(res, key)
          })
      }
    }
  }
</script>

<style>
.fa-search:before {
  font-size: 16px;
}
.empty-area {
  width: 100%;
  height: 400px;
  border: 1px solid #ccc;
}
.main.container {
  padding: 10px;
}
</style>
