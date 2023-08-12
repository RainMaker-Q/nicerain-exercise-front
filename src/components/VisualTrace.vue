<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import * as echarts from 'echarts/core'
import { GridComponent } from 'echarts/components'
import { LineChart } from 'echarts/charts'
import { UniversalTransition } from 'echarts/features'
import { CanvasRenderer } from 'echarts/renderers'

import axios from 'axios'

echarts.use([GridComponent, LineChart, CanvasRenderer, UniversalTransition])

enum DisplayType {
  TURNS,
  DISTANCE
}

type IResData = { id: number; time_stamp: string; turns: number }[]

const type = ref<DisplayType>(DisplayType.DISTANCE)

const myChart = ref()

const resData = ref<IResData>([])

const option = ref({
  xAxis: {
    type: 'category',
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  yAxis: {
    type: 'value',
    name: '距离(m)'
  },
  series: [
    {
      data: [820, 932, 901, 934, 1290, 1330, 1320],
      type: 'line',
      smooth: true
    }
  ]
})

onMounted(async () => {
  const chartDom = document.getElementById('main') as HTMLElement
  myChart.value = echarts.init(chartDom)
  myChart.value.setOption(option.value)

  const res = await axios.get<any, { data: IResData }>('http://192.168.0.109:3000/nicerain')

  resData.value = res?.data || []
})

watch([resData, type], () => {
  const data = resData.value

  const xData = data?.map((item) => item?.time_stamp.slice(0, 10) || '')

  const unit = type.value === DisplayType.TURNS ? 1 : (Math.PI * 17) / 100
  const yData = data?.map((item) => item?.turns * unit || 0)

  const unitDesc = type.value === DisplayType.TURNS ? '圈数' : '距离(m)'
  option.value.xAxis.data = xData
  option.value.yAxis.name = unitDesc
  option.value.series[0].data = yData
  myChart.value.setOption(option.value)
})

const handleClick = () => {
  if (type.value === DisplayType.DISTANCE) {
    type.value = DisplayType.TURNS
  } else {
    type.value = DisplayType.DISTANCE
  }
}
</script>

<template>
  <div id="main" class="main"></div>
  <div class="btn" :onclick="handleClick">
    {{ type === DisplayType.TURNS ? '切换为距离' : '切换为圈数' }}
  </div>
</template>

<style scoped>
.main {
  width: 100%;
  height: 100%;
}

.btn {
  position: absolute;
  top: 30px;
  right: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 30px;
  padding: 10px;
  border-radius: 4px;
  background-color: lightblue;
  color: #222;
}
</style>
