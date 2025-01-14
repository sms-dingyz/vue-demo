<template>
  <div id="main" style="width: 100%; height: 600px;"/>
</template>

<script>
import * as echarts from 'echarts';
import { getData } from './data3'
var xData = []
function getXData() {
  for (let i = 0; i < 120; i++) {
    xData.push(i)
  }
}
getXData()
//  生成随机颜色
function getRandomColor() {
  var letters = '0123456789ABCDEF';
  var color = '#';
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}
export default {
  data() {
    return {
      echartData: getData(),
      colorList: []
    };
  },
  mounted() {
    var chartDom = document.getElementById('main');
    var myChart = echarts.init(chartDom);
    var list = []

    for (var i = 0; i < this.echartData.length; i++) {
      this.colorList.push(getRandomColor())
      list.push(
        {
          name: '',
          type: 'line',
          stack: 'Total',
          smooth: true,
          lineStyle: {
            width: 0
          },
          showSymbol: false,
          areaStyle: {
            opacity: 0.8,
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              {
                offset: 0,
                color: this.colorList[i]
              },
              {
                offset: 1,
                color: this.colorList[i]
              }
            ])
          },
          emphasis: {
            focus: 'series'
          },
          data: this.echartData[i]
        }
      )
    }
    var option = {
      color: this.colorList,
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'cross',
          label: {
            backgroundColor: '#6a7985'
          }
        }
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
      },
      xAxis: {
        type: 'category',
        boundaryGap: false,
        data: xData
      },
      yAxis: [
        {
          type: 'value',
          position: 'right'
        }
      ],
      graphic: [
        {
          type: 'group',
          left: '10%',
          top: 'center'
        }
      ],
      series: list
    };

    option && myChart.setOption(option);
  }
}



</script>

<style>
</style>
