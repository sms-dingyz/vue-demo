<template>
  <div class="echart-page">
    <div class="y-desc">Temperature[C]</div>
    <canvas id="fiostherms"/>
    <div style="margin-top: 20px;font-weight: bold">Distance from mold level[m]</div>
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

</template>

<script>
  import * as echarts from 'echarts';
  import { getData } from './data3'
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

  var dpr = window.devicePixelRatio || 1;
  export default {
    data() {
      return {
        echartData: getData(),
        colorList: []
      };
    },
    mounted() {
      this.getColorList()
      canvas = document.getElementById("fiostherms");
      ctx = this.setupCanvas();
      this.drawAxis();
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
        console.log(" ___________ aaaa ")
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
        this.drawHorizontalAxisTicks();
        // this.drawVerticalAxisTicks();
        this.drawFiostherms();
        // this.draw_fLiquidus_fSolidus(this.echartData.fLiquidus, 'Liquidus')
        // this.draw_fLiquidus_fSolidus(this.echartData.fSolidus, 'Solidus')
        // this.draw(this.echartData.fTempCoreObj, "#e0190a", 'Core',0);
        // this.draw(this.echartData.fTempMeanObj, "#a0264f", 'Average',6);
        // this.draw(this.echartData.fTempSideObj, "#2196f3", 'Center loose',10);
      },
      //横坐标标尺及刻度
      drawHorizontalAxisTicks() {
        var deltaX,num = 0, spacing = parseFloat((realX / 120).toFixed(1));
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
          ctx.moveTo(originX + i * spacing, originY - deltaX);
          ctx.lineTo(originX + i * spacing, originY);
          ctx.stroke();
          function text(){
            ctx.font = "12px Calibri";
            if(i > 9) {
              ctx.fillText(num,originX + (i - 0.7) * spacing, canvasHeight);
            } else {
              ctx.fillText(num,originX + (i - 0.5) * spacing, canvasHeight);
            }
          }
          drawRow();
          function drawRow() {
            if(i % 112 === 0 && i !== 0) {//change 10 to 112
              ctx.beginPath();
              ctx.strokeStyle = '#a2dba1';
              ctx.moveTo(originX + i * spacing, originY - deltaX);
              ctx.lineTo(originX + i * spacing, endY);
              ctx.stroke();
            }
          }
        }
      },
      //  生成随机颜色
      getRandomColor(row , colum) {
        var letters = '0123456789ABCDEF';
        var color = '#';
        for (var i = 0; i < 6; i++) {
          color += letters[Math.floor(Math.random() * 16)];
        }

        var data = this.echartData[row];
        if ( colum < 20 && row >2 && row < 18)
          return color = '#FFDA2D';
        else
          return color;
      },
      getColorList() {
        for (var i = 0; i < 120; i++) {
          this.colorList.push(this.getRandomColor())
        }
      },
      drawFiostherms(){

        var deltaY, num = 100, spacing = parseFloat((realY / 100).toFixed(4));
        var drawCnt = 0 ;
        console.log("_________ originX:",originX)
        console.log("_________ originY:",originY)
        console.log("_________ spacing:",spacing)
        console.log("_________ realY:",realY)
        ctx.beginPath();
        ctx.lineWidth = 11.3 ;


        const gnt = ctx.createLinearGradient(originX, 0, originX + 119*ctx.lineWidth - 20 , originY - 0 * spacing -  4 + 4)
        gnt.addColorStop(0, '#E89B9B')
        gnt.addColorStop(0.5, '#A52A2A')
        gnt.addColorStop(1, '#A52A2A')
        ctx.fillStyle = gnt
        ctx.fillRect(originX, 0, originX + 119*ctx.lineWidth - 20, originY - 0 * spacing -  4 + 4 )

        ctx.beginPath();
        ctx.lineWidth = 2 ;
        ctx.moveTo(originX, 0);
        ctx.quadraticCurveTo(50,200,50,20);
        ctx.stroke();
        for (var i = 0; i < 0 ; i++) {
          for ( var j = 0 ; j < 0 ; j ++ ){
            ctx.beginPath();
            ctx.strokeStyle = 'black';
            ctx.strokeStyle = this.getRandomColor(i,j);
            ctx.moveTo(originX + j*ctx.lineWidth , originY - i * spacing - 4 + i );
            ctx.lineTo(originX + (j+ 1)*ctx.lineWidth ,  originY - i * spacing -  4 + i );
            ctx.stroke();

            ctx.font = "12px Calibri";
            ctx.fillText(num, 0, originY - (i - 1) * spacing);
          }

          // ctx.strokeStyle = 'black';
          // if (i % 20 === 0 && i !== 0 ) {
          // deltaY = 118;
          // text();
          // num += 100;
          // } else {
          // deltaY =84;
          // }


          // ctx.lineWidth = 10 ;
          // spacing = 10 ;
          // ctx.moveTo(originX, originY - i * spacing - 4 );
          // ctx.lineTo(originX + deltaY, originY - i * spacing -  4);
          // ctx.stroke();

          function text() {
            ctx.font = "12px Calibri";
            ctx.fillText(num, 0, originY - (i - 1) * spacing);
          }
          // drawRow();
          // function drawRow() {
          //   if ( drawCnt < rowLineCnt ) {
          //     if(i % 10 === 0 && i !== 0) {
          //       ctx.beginPath();
          //       ctx.strokeStyle = '#a2dba1';
          //       ctx.moveTo(originX + deltaY, originY - i * spacing);
          //       //minus 128
          //       ctx.lineTo(endX + originX - gXAxisOffset, originY - i * spacing);
          //       ctx.stroke();
          //       drawCnt ++ ;
          //     }
          //   }

          // }
        }

      },

      //纵坐标标尺及刻度
      drawVerticalAxisTicks() {
        var deltaY, num = 100, spacing = parseFloat((realY / 100).toFixed(4));
        var drawCnt = 0 ;
        console.log("_________ originX:",originX)
        console.log("_________ originY:",originY)
        console.log("_________ spacing:",spacing)
        console.log("_________ realY:",realY)

        for (var i = 0; i < 120; i++) {
          ctx.beginPath();
          ctx.strokeStyle = 'black';
          if (i % 10 === 0 ) {
            deltaY = 8;
            text();
            num += 100;
          } else {
            deltaY =84;
          }



          ctx.moveTo(originX, originY - i * spacing );
          ctx.lineTo(originX + deltaY, originY - i * spacing);
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
  #fiostherms{
    width: 80%;
    height: 400px;
  }
</style>
