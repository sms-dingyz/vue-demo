<template>
  <div class="echart-page">
    <div class="y-desc">Temperature[C]</div>
    <div class="echart-box">
      <div id="strand" style="width: 1800px; height: 600px;"/>
    </div>
    <div class="echart-footer">
      <div class="echart-footer-left">
        <div><span>Heat ID</span>   B2223k37806</div>
        <div>Strand 1269 X 53mm</div>
        <div>Steel grade Q235B</div>
      </div>
      <div class="echart-footer-center">
        <div>Casting speed  4,70 m/min</div>
      </div>
      <div class="echart-footer-right">
        <div><span>project</span> Masteel</div>
        <div><span>Date:</span> 3t-Aug-23</div>
      </div>
    </div>
  </div>

  <div>
      <input v-model="message" @keyup.enter="sendMessage">
      <button @click="sendMessage">Send</button>
      <ul>
        <li v-for="msg in messages">{{ msg }}</li>
      </ul>
    </div>

</template>

<script>
import * as echarts from 'echarts';
import { getData } from './temperatureCurve.'

export default {
  data() {
    return {
		 message: '',
		 messages: [],
		 socket: null,
     echartData: getData()
	};
  },
  mounted() {
	this.socket = new WebSocket('ws://localhost:3000');
	this.socket.onmessage = (event) => {
		console.log("_____ event.data",event.data)
		this.messages.push(event.data);
		// this.echartData = getData(event.data)
	};

    var chartDom = document.getElementById('strand');
    var myChart = echarts.init(chartDom);
    var option;
    const axisLabel = []
      myChart.setOption(
        (option = {
          title: {
            left: 'center',
            text: 'Strand Temperature'
          },
          animation: false,
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: "cross",
              show: false,
              crossStyle: {
                color: "#999",
              }
            },
            textStyle: {
              align:'left'
            },
            formatter: function(params){
              let str = params[0].axisValue + "m<br />";
              params.forEach((item) => {
                if(item.seriesName === 'Core') {
                    item.color = '#e0190a';
                  }
                  if(item.seriesName === 'Average') {
                    item.color = '#a0264f';
                  }
                  if(item.seriesName === 'center loose') {
                    item.color = '#2196f3';
                  }
                str +=
                  '<span style="display:inline-block;margin-right:5px;border-radius:50%;width:10px;height:10px;left:5px;background-color:'+item.color+'"></span>' + item.seriesName + " : " + item.value + "<br />";
              });
              return str;
            }
          },
          axisPointer: {
            link: [{ xAxisIndex: 'all' }]
          },
          grid: [
            {
              left: '10%',
              right: '8%',
              bottom: 150
            }
          ],
          xAxis: [
            {
              type: "category",
              data: this.echartData.xData,
              axisLabel:{
              //   formatter(v){
              //     if(axisLabel.indexOf(parseInt(v)) === -1) {
              //       debugger
              //       axisLabel.push(parseInt(v))
              //       console.log('v=' + v)
              //       console.log('v=' + parseInt(v))
              //       console.log(axisLabel)
              //       if(parseInt(v) % 2 === 0) {
              //         return v;
              //       } else {
              //         return ''
              //       }
              //     } else {
              //       return ''
              //     }
              //   }
                interval: 80
              },
              splitLine: {
                show: false
              },
              // show: false
            },
            // {
            //   type:'category',
            //   name:'Distance from mold level[m]',
            //   nameLocation: 'center',
            //   nameTextStyle: {
            //     color: '#000',
            //     fontSize: '16',
            //     padding: [20, 0, 0]
            //   },
            //   position: 'bottom',
            //   axisLabel:{ interval: 9 },
            //   data: xData,
            //   axisPointer: {
            //     type: 'none'
            //   },
            // },
          ],
          yAxis: [
            {
              scale: true,
              splitArea: {
                show: true
              },
              min: 600,
              interval: 100
            }
          ],
          series: [
            {
              name: 'Core',
              type: 'line',
              showSymbol: false,
              smooth:true, //将折线变成平滑的线条，不是直线
              endLabel: {
                show: true,
                formatter: function (params) {
                  return params.seriesName
                },
                color: '#e0190a',
                fontSize: 18
              },
              lineStyle: {
                color: '#e0190a'
              },
              data: this.echartData.fTempCoreArray
            },
            {
              name: 'Average',
              type: 'line',
              showSymbol: false,
              smooth:true, //将折线变成平滑的线条，不是直线
              endLabel: {
                show: false,
                  formatter: function (params) {
                  return params.seriesName
                },
                color: '#a0264f',
                  fontSize: 18
              },
              lineStyle: {
                color: '#a0264f'
              },
              data: this.echartData.fTempMeanArray
            },
            {
              name: 'center loose',
              type: 'line',
              showSymbol: false,
              smooth:true, //将折线变成平滑的线条，不是直线
              endLabel: {
                show: false,
                formatter: function (params) {
                  return params.seriesName
                },
                color: '#2196f3',
                fontSize: 18
              },
              lineStyle: {
                color: '#2196f3'
              },
              data: this.echartData.fTempSideArray
            }
          ]
        }),
        true
      );

    option && myChart.setOption(option);

  },
  methods: {
      sendMessage() {
        if (this.message.trim() !== '') {
          this.socket.send(this.message);
          this.message = '';
        }
      },
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
  .echart-box{
    width: 100%;
    display: flex;
    justify-content: center;
  }
</style>
