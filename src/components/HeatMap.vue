<template>
 <div id="heat" style="width: 600px;height:400px;"></div>

</template>

<script>
import * as echarts from 'echarts';

export default {
  data() {
    return {};
  },
  mounted() {

    var chartDom = document.getElementById('heat');
    var myChart = echarts.init(chartDom);
    var option;
	
	var option = {
	  tooltip: {
	    position: 'left',
	  },
	  animation: false,
	  xAxis: {
	    type: 'category',
	    data: ['04-10-2020', '05-10-2020', '06-10-2020'],
	    splitArea: {
	      show: false
	    },
	    position: 'top'
	  },
	  yAxis: {
	    type: 'category',
	    data: ['A', 'B'],
	    splitLine: {
	      show: true,
	      lineStyle: {
	        width: 2
	      }
	    }
	  },
	  visualMap: [{
	      type: 'continuous',
	      show: false,
	      min: 0,
	      max: 15,
	      seriesIndex : 0,
	      inRange: {
	        color: ['#ffffff', '#990000']
	      },
	      dimension: '2',
	    }, {
	      type: 'continuous',
	      show: false,
	      min: 5,
	      max: 15,
	      seriesIndex: 1,
	      inRange: {
	        color: ['#ffffff', '#990000']
	      },
	      dimension: '2',
	    }
	
	  ],
	  series: [{
	    name: "test1",
	    type: 'heatmap',
	    data: [
	      [0, 0, 5],
	      [1, 0, 10],
	      [2, 0, 15]
	    ],
	    label: {
	      show: false
	    },
	    coordinateSystem: 'cartesian2d',
	    emphasis: {
	      itemStyle: {
	        shadowBlur: 10,
	        shadowColor: 'rgba(0, 0, 0, 0.5)'
	      }
	    }
	  },
	  {
	    name: "test2",
	    type: 'heatmap',
	    data: [
	      [0, 1, 5],
	      [1, 1, 10],
	      [2, 1, 15]
	    ],
	    label: {
	      show: false
	    },
	    coordinateSystem: 'cartesian2d',
	    emphasis: {
	      itemStyle: {
	        shadowBlur: 10,
	        shadowColor: 'rgba(0, 0, 0, 0.5)'
	      }
	    }
	  }
	  ]
	}
	
	var myChart = echarts.init(document.getElementById('heat'));
	myChart.setOption(option);


  }
}



</script>

<style>
  .echart-page{
    position: relative;
  }
  .y-desc{
    transform: rotate(-90deg);
    transform-origin: left;
    position: absolute;
    left: 80px;
    top: 50%;
    font-size: 17px;
    font-weight: bold;
  }
  .echart-footer{
    margin-top: -80px;
    padding: 0 80px;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    font-weight: 500;
    text-align: left;
  }
  .echart-footer span {
    display: inline-block;
    width: 70px;
  }
</style>
