<template>
  <div>
    <h2>{{record.name}}</h2>
    <br>
    <div>
      <Switch size="small" v-model="enableYAxisDataMin" @on-change="changeYAxisMinType" />
      <strong title="停用/启用: 以 0/数据中的最小值 作为Y轴起点"> Reactive YAxis </strong> 
    </div>
    <div :style="{position:'relative'}">            
      <LineChart :opt="opt"></LineChart>
      <Spin fix v-if="spinShow">
        <Icon type="load-c" size=18 class="demo-spin-icon-load"></Icon>
        <div>Loading</div>
      </Spin>
    </div>
    <br>
    <DateSlider :startTime="originalDateValue[0]" :endTime="originalDateValue[1]" :onchange="sliderChanged"></DateSlider>
  </div>
</template>
<style>
.demo-spin-icon-load {
  animation: ani-demo-spin 1s linear infinite;
}
</style>
<script>
import LineChart from './LineChart.vue'
import DateSlider from './DateSlider.vue'
import moment from 'moment'
import Vue from 'vue'

const KEY_ENABLE_YAXIS_DATA_MIN = "MemberInfo-EnableYAxisDataMin";
export default {
  components: { LineChart, DateSlider },
  props: {
    record: Object
  },
  data() {
    return {
      spinShow: true,
      originalDateValue: [this.record.ctime, +moment()],
      dateValue: [this.record.ctime, +moment()],
      opt: {},
      enableYAxisDataMin: true
    }
  },
  mounted() {
    this.enableYAxisDataMin = localStorage.getItem(KEY_ENABLE_YAXIS_DATA_MIN) === 'true'
    this.fetchMemberInfo(this.dateValue)
  },
  methods: {
    fetchMemberInfo(timerange) {
      this.spinShow = true
      const url = `/trace/memberInfo/${this.record.mid}`
      this.$api.get(url, { st: timerange[0], et: timerange[1] }, r => {
        if (!r.data.list) return
        this.opt = this.createOpt(r.data.list)
        this.spinShow = false
      })
    },
    createOpt(dataList) {
      const fields = ['fans','archiveView']
      let obj = {
        tooltip: {
          trigger: 'axis',
          formatter: v => {
            let str = `${moment(v[0].value[0]).format('YYYY-MM-DD HH:mm:ss')}`
            for(let i=0; i < v.length; i++){
              str += `<br/>${v[i].marker} ${v[i].name}: ${v[i].value[1]}`
            }
            return str
          }
        },
        legend: {
          data: fields
        },
        xAxis: {
          type: 'time',
          // maxInterval: 3600 * 12 * 1000,
          splitLine: {
            show: true
          }
        },
        yAxis: [
          {
            name: 'fans',
            type: 'value',
            min: this.enableYAxisDataMin ? 'dataMin' : null
          },
          {
            name: 'archiveView',
            type: 'value',
            min: this.enableYAxisDataMin ? 'dataMin' : null
          },
          
        ],
        series: fields.map(field => {
          return {
            name: field,
            type: 'line',
            showSymbol: false,
            hoverAnimation: false,
            yAxisIndex: field == 'fans' ? 0 : 1,
            data: dataList.filter(v => v[field] > 0).map(v => {
              return {
                name: field,
                value: [v.ctime, v[field]]
              }
            })
          }
        })
      }
      return obj
    },
    sliderChanged(left, right) {
      this.dateValue = [left, right]
      this.fetchMemberInfo([left, right])
    },
    changeYAxisMinType(status){
      localStorage.setItem(KEY_ENABLE_YAXIS_DATA_MIN, status)
      this.fetchMemberInfo(this.dateValue)
    },
    str2timestamp(str) {
      return +moment(str)
    },
    timestamp2str(arr){
      return arr.map(v => moment(v).format('YYYY-MM-DD HH:mm:ss'))
    }
  }
}
</script>
