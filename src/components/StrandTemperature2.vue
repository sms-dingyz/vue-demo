<template>
  <div class="echart-page">
    <div class="y-desc">Temperature[C]</div>
    <canvas id="myCanvas"/>
    <div style="margin-top: 20px;font-weight: bold">Distance from mold level[m]</div>
    <!-- <div class="echart-footer">
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
    </div> -->
  </div>

</template>

<script>
  import * as echarts from 'echarts';
  import {watch, nextTick} from 'vue';
  import { getData } from './temperatureCurve'
  var canvas = null;
  var ctx = null;
  var canvasWidth = 0,
    canvasHeight = 0,
    originX = 0, //  坐标系原点横坐标值
    originY = 0, //  坐标系原点纵坐标值
    endX = 0, //  横坐标终点
    endY = 0, //  纵坐标终点
    driftX = 40, //  纵坐标数字的宽度（横向空出的距离，600、700、800的宽度）
    driftY = 10,//  横坐坐标数字的高度（纵向空出的距离，0、2、4的高度）
    realX = 0,//  横坐标实际长
    realY = 0;//  纵坐标实际高

  var gXAxisOffset = 128 ; //not good solution
  const rowLineCnt = 7 ; //maxLine
  var _this = null
  var dpr = window.devicePixelRatio || 1;
  export default {
    props: {
      webScoketData: {
        type: Object,
        default: () => {
          return {}
        }
      }
    },
    setup(props) {
      watch(
        () => props.webScoketData,
        (newValue, oldValue) => {
          if(newValue) {
            nextTick(() => {
              _this.echartData =  {...newValue}
              _this.drawAxis();
            })
          }
        },{deep: true, immediate: true}
      )
    },
    data() {
      return {
        echartData: {}
      };
    },
    mounted() {
      _this = this
      canvas = document.getElementById("myCanvas");
      ctx = this.setupCanvas();
    },
    methods: {
      setupCanvas() {
        var rect = canvas.getBoundingClientRect();
        canvas.width = rect.width * dpr;
        canvas.height = rect.height * dpr;
        var context = canvas.getContext('2d');
        context.scale(dpr, dpr);
        return context
      },
      //绘制坐标轴的方法
      drawAxis() {
        console.log(this.echartData)
        // 4.拿到canvas的宽高
        canvasWidth = Number(ctx.canvas.width) / dpr;
        canvasHeight = Number(ctx.canvas.height) / dpr;

        // 1.计算坐标系原点的位置
        originX = 30;
        originY = canvasHeight - 20;
        // 2.计算X轴终点的位置
        endX = canvasWidth - driftX;
        // 3.x，y实际长度
        realX = endX - originX;
        realY = originY - driftY;
        // 4.绘制X轴
        ctx.beginPath();
        ctx.moveTo(originX, originY);
        //modified at 2023/9/7 minus 128
        ctx.lineTo(endX + originX - gXAxisOffset + 3, originY);
        ctx.strokeStyle = "#000";
        ctx.stroke();

        // 5.计算Y轴终点的位置
        endY = 0 - canvasHeight + driftY;
        // 6.绘制Y轴
        ctx.beginPath();
        ctx.moveTo(originX, originY);
        ctx.lineTo(originX, endY);
        ctx.strokeStyle = "#000";
        ctx.stroke();
        this.drawHorizontalAxisTicks(this.echartData.fLiquidus);
        this.drawVerticalAxisTicks();
        this.draw_fLiquidus_fSolidus(this.echartData.fLiquidus, 'Liquidus')
        this.draw_fLiquidus_fSolidus(this.echartData.fSolidus, 'Solidus')
        this.draw(this.echartData.fTempCoreObj, "#e0190a", 'Core',0);
        this.draw(this.echartData.fTempMeanObj, "#a0264f", 'Average',6);
        this.draw(this.echartData.fTempSideObj, "#2196f3", 'Center loose',10);
      },
      //横坐标标尺及刻度
      drawHorizontalAxisTicks(yAxis) {
        var deltaX,num = 0, spacingX = parseFloat((realX / 120).toFixed(1)), spacingY = parseFloat((realY / 100).toFixed(4));;
        for (var i = 0;i < 113;i++){ //change 120 to 113 2003/09/07
          ctx.beginPath();
          ctx.strokeStyle = '#000';
          if(i % 10 === 0){
            deltaX = 8;
            text();
            num += 2;
          }else {
            deltaX = 4;
          }
          ctx.moveTo(originX + i * spacingX, originY - deltaX);
          ctx.lineTo(originX + i * spacingX, originY);
          ctx.stroke();
          function text(){
            ctx.font = "12px Calibri";
            if(num > 9) {
              ctx.fillText(num,originX + (i - 0.6) * spacingX, canvasHeight);
            } else {
              ctx.fillText(num,originX + (i - 0.5) * spacingX, canvasHeight);
            }
          }
          drawRow();
          function drawRow() {
            if(i % 112 === 0 && i !== 0) {//change 10 to 112
              ctx.beginPath();
              ctx.strokeStyle = '#a2dba1';
              ctx.moveTo(originX + i * spacingX, originY - deltaX);
              ctx.lineTo(originX + i * spacingX, realY - ( yAxis * realY / 1000 - 3 * spacingY));
              ctx.stroke();
            }
          }
        }
      },
      //纵坐标标尺及刻度
      drawVerticalAxisTicks() {
        var deltaY, num = 600, spacing = parseFloat((realY / 100).toFixed(4));
        var drawCnt = 0 ;
        for (var i = 0; i < 120; i++) {
          ctx.beginPath();
          ctx.strokeStyle = '#000';
          if (i % 10 === 0) {
            deltaY = 8;
            text();
            num += 100;
          } else {
            deltaY = 4;
          }
          ctx.moveTo(originX + deltaY, originY - i * spacing);
          ctx.lineTo(originX, originY - i * spacing);
          ctx.stroke();

          function text() {
            ctx.font = "12px Calibri";
            ctx.fillText(num, 0, originY - (i - 1) * spacing);
          }
          drawRow();
          function drawRow() {
            if ( drawCnt < rowLineCnt ) {


              if(i % 10 === 0 && i !== 0) {
                ctx.beginPath();
                ctx.strokeStyle = '#a2dba1';
                ctx.moveTo(originX + deltaY, originY - i * spacing);
                //minus 128
                ctx.lineTo(endX + originX - gXAxisOffset, originY - i * spacing);
                ctx.stroke();
                drawCnt ++ ;
              }
            }

          }
        }
      },
      draw(list, color, name,nameyOffset) {
        var spacing = parseFloat((realY / 100).toFixed(4));
        ctx.strokeStyle = color;
        ctx.lineWidth = '2';
        ctx.beginPath();
        var item = list[0];
        ctx.moveTo(item.x * realX / 24 + originX, realY - (item.y * realY / 1000 - 3 * spacing));

        for (let i = 1; i < list.length; i++) {
          var item = list[i];
          ctx.lineTo(item.x * realX / 24 + originX, realY - (item.y * realY / 1000 - 3 * spacing));
        }
        const lastP = list[list.length - 2];
        ctx.font = "16px Calibri";
        ctx.fillStyle = color;
        ctx.fillText(name, lastP.x * realX / 24 + originX + 4 , realY - (lastP.y * realY / 1000 - 3 * spacing - nameyOffset  ));
        ctx.stroke()
      },
      draw_fLiquidus_fSolidus( yAxis, name) {
        var fEndX = this.echartData.fTempCoreObj[this.echartData.fTempCoreObj.length - 1].x
        var spacing = parseFloat((realY / 100).toFixed(4));
        ctx.beginPath();
        ctx.strokeStyle = 'grey';
        ctx.moveTo(originX, realY - ( yAxis * realY / 1000 - 3 * spacing));
        ctx.lineTo(fEndX * realX / 24 + originX - 2, realY - ( yAxis * realY / 1000 - 3 * spacing));
        ctx.font = "16px Calibri";
        ctx.fillStyle = "grey";
        ctx.fillText(name, fEndX * realX / 24 + originX + 4, realY - ( yAxis * realY / 1000 - 3 * spacing));
        ctx.stroke();
      },
    }
  }



</script>

<style scoped>
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
    padding: 80px;
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
  #myCanvas{
    width: 80vw;
    height: 400px;
  }
</style>
