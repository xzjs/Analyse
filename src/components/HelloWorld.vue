<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import * as echarts from 'echarts';

const bianceData = ref([])

function showData() {
  let myChart = echarts.init(document.getElementById('echarts1'));
  let data = []
  bianceData.value.forEach(element => {
    let key = Math.round(element.fundingRate * 10000 * 2) / 2
    data.push([element.fundingTime / 1000, key])
  })
  myChart.setOption({
    title: {
      text: '资金费率图'
    },
    tooltip: {
      trigger: 'axis'
    },
    dataZoom: [
    {
      type: 'inside',
      start: 0,
      end: 100
    },
    {
      start: 0,
      end: 100
    }
  ],
    legend: {
    },
    xAxis: {
      name: "时间(s)",
      type: 'value',
      min: 'dataMin'
    },
    yAxis: {
      name: "资金费率",
      type: 'value'
    },
    series: [{
      data: data,
      type: 'line'
    }]
  });
}

function statistics1() {
  let myChart = echarts.init(document.getElementById('echarts2'));
  let orderArr = []
  let lastDict = {};
  let dict = {};
  bianceData.value.forEach(element => {
    let key = Math.round(element.fundingRate * 10000 * 2) / 2
    if (orderArr.length == 0) {
      orderArr.push(key)
      orderArr.sort()
      lastDict[key] = 8
    } else {
      let needRemoveArr = []
      // 遍历有序数组，挨个比较，小于等于key 就增加持续时间，大于 key 就记入移除数组，后续操作
      orderArr.forEach(e => {
        if (key >= e) {
          lastDict[e] += 8
        } else {
          needRemoveArr.push(e)
        }
      })
      // 遍历移除数组，统计超过当前 key 的持续时间，并从 orderArr 和 lastDict 中移除
      needRemoveArr.forEach(e => {
        if (e in dict) {
          let duration = lastDict[e]
          if (duration in dict[e]) {
            dict[e][duration] += 1
          } else {
            dict[e][duration] = 1
          }
        } else {
          dict[e] = { [lastDict[e]]: 1 }
        }
        const index = orderArr.indexOf(e)
        orderArr.splice(index, 1)
        delete lastDict[e]
      })
      // 若当前 key 不在有序数组里，加入其中
      if (!orderArr.includes(key)) {
        orderArr.push(key)
        lastDict[key] = 8
      }
    }
  })
  // lastdict 中还剩一部分数据
  for (let k in lastDict) {
    let duration = lastDict[k]
    if (k in dict) {
      if (duration in dict[k]) {
        dict[k][duration] += 1
      } else {
        dict[k][duration] = 1
      }
    } else {
      dict[k] = { [duration]: 1 }
    }
  }

  let series = []
  for (let key in dict) {
    let data = []
    for (let k in dict[key]) {
      data.push([k, dict[key][k]])
    }
    series.push({
      name: key,
      type: 'line',
      data: data
    })
  }

  // 绘制图表
  myChart.setOption({
    title: {
      text: '增势图'
    },
    tooltip: {
      trigger: 'axis'
    },
    dataZoom: [
    {
      type: 'inside',
      start: 0,
      end: 100
    },
    {
      start: 0,
      end: 100
    }
  ],
    legend: {
      data: Object.keys(dict).sort()
    },
    xAxis: {
      name: "持续时间",
      type: 'value',
      min: 'dataMin'
    },
    yAxis: {
      name: "采样数量",
      type: 'value'
    },
    series: series
  });
}

function statistics2() {
  var myChart = echarts.init(document.getElementById('echarts3'));
  let dict = {}
  let duration = 0
  let lastKey = null
  bianceData.value.forEach(element => {
    let key = Math.round(element.fundingRate * 10000 * 2) / 2
    if (lastKey == null) {
      duration = 8
      lastKey = key
    } else if (lastKey == key) {
      duration += 8
    } else if (lastKey != key) {
      if (lastKey in dict) {
        if (duration in dict[lastKey]) {
          dict[lastKey][duration] += 1
        } else {
          dict[lastKey][duration] = 1
        }
      } else {
        dict[lastKey] = { [duration]: 1 }
      }

      lastKey = key
      duration = 8

    }
  });

  let series = []
  for (let key in dict) {
    let data = []
    for (let k in dict[key]) {
      data.push([k, dict[key][k]])
    }
    series.push({
      name: key,
      type: 'line',
      data: data
    })
  }

  // 绘制图表
  myChart.setOption({
    title: {
      text: '单独统计'
    },
    tooltip: {
      trigger: 'axis'
    },
    dataZoom: [
    {
      type: 'inside',
      start: 0,
      end: 100
    },
    {
      start: 0,
      end: 100
    }
  ],
    legend: {
      data: Object.keys(dict).sort()
    },
    xAxis: {
      name: "持续时间",
      type: 'value',
      min: 'dataMin'
    },
    yAxis: {
      name: "采样数量",
      type: 'value'
    },
    series: series
  });
}

onMounted(() => {
  axios.get('https://fapi.binance.com/fapi/v1/fundingRate', {
    params: {
      symbol: 'BTCUSDT',
      limit: 1000
    }
  })
    .then(function (response) {
      bianceData.value = response.data.reverse()
      showData()
      statistics1()
      statistics2()
    })
    .catch(function (error) {
      console.log(error)
    })
})
</script>

<template>
  <div id="echarts1" style="width:400px;height:400px;"></div>
  <div id="echarts2" style="width:400px;height:400px;"></div>
  <div id="echarts3" style="width:400px;height:400px;"></div>
</template>

<style scoped></style>
