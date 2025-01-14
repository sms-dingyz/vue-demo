<template>
  <canvas id="fiostherms"/>
  <div style="margin-top: 20px;font-weight: bold">Solidification[mm]:{{this.BottomfSoliLength}}</div>
</template>

<script>
  import * as echarts from 'echarts/index';
  import { getData } from './fiostherm'
  import { getFiosthermData } from './fiostherm'
  var canvas = null;
  var ctx = null;
  var canvasWidth = 0,
    canvasHeight = 0,
    originX = 0,
    originY = 0,
    endX = 0,
    endY = 0,
    driftX = 40,
    driftY = 30,
    realX = 0,
    realY = 0;
  var originxOffset = 30


  const SocketServer = 'ws://127.0.0.1:50000'

  var dpr = window.devicePixelRatio || 1;
  export default {
    data() {
      return {
		fSoliLength: 6.24,
		BottomfSoliLength:5300,
        echartData: [],
        colorObj: []
      };
    },
    // mounted() {
    //   const data = getData();
    //   this.echartData = data.list;
    //   this.colorObj = data.color;
    //   canvas = document.getElementById("fiostherms");
    //   ctx = this.setupCanvas();
    //   this.drawAxis();
    // },

	mounted() {
	  canvas = document.getElementById("fiostherms");
	  ctx = this.setupCanvas();
	  this.getTemperatureData()
	  this.drawFiostherm();
	},

    methods: {
	getTemperatureData() {
	  return new Promise((resolve) =>{
	    var messagetype = 'undefined'
	    var socket = new WebSocket( SocketServer );
	    socket.onmessage = (event) => {
	      if(typeof event.data === 'string') {
	        var tmpData = JSON.parse(event.data)
	        messagetype = tmpData.messagetype
	      }
	      if ( 'fIsotherm' === tmpData.messagetype ) {
	        this.echartData = getFiosthermData( JSON.parse(event.data) )
	      }
		  else if ( tmpData.messagetype === "ModelResult") {
			  var data = JSON.parse(event.data )
			  const ModelResultData = JSON.parse( data.ModelResultData )
			  this.fSoliLength = ModelResultData.fSoliLength/1000
			  this.fSoliLength = this.fSoliLength.toFixed(0)
			  this.BottomfSoliLength = ModelResultData.fSoliLength.toFixed(0)

			   this.drawAxis();
			  console.log(" FiosTHERMS3____________ fSoliLength:",this.fSoliLength/1000)
		  }
	      resolve()
	    }
	  })
	},

      getColorList() {
        for (var i = 0; i < 120; i++) {
          this.colorList.push(getRandomColor())
        }
      },
      setupCanvas() {
        var rect = canvas.getBoundingClientRect();
        canvas.width = rect.width * dpr;
        canvas.height = rect.height * dpr;
        var context = canvas.getContext('2d');
        context.scale(dpr, dpr);
        return context
      },

	  drawFiostherm(){
		  this.drawAxis();
	  },

      drawAxis() {
        canvasWidth = Number(ctx.canvas.width) / dpr;
        canvasHeight = Number(ctx.canvas.height) / dpr;

        originX = canvasWidth - driftX - originxOffset;
        originY = canvasHeight - driftY;
        endX = 30 ;
        realX = canvasWidth - driftX - originxOffset ;
        realY = canvasHeight - driftY;

        ctx.beginPath();
        ctx.moveTo(originX - 50, originY);
        ctx.lineTo(endX , originY);
        ctx.strokeStyle = "#000";
        ctx.stroke();

        endY = 15;
        // ctx.beginPath();
        // ctx.moveTo(originX, originY);
        // ctx.lineTo(originX, endY);
        // ctx.strokeStyle = "#000";
        // ctx.stroke();
        console.log('colorList=' + this.colorList)

        var spacingX = parseFloat((realX / 120).toFixed(1))

        ctx.save();
        ctx.moveTo(originX - driftX - 0.5 * spacingX, endY);
        ctx.lineTo(endX, endY);
        ctx.lineTo(endX, originY);
        ctx.lineTo(originX - driftX - 0.5 * spacingX, originY);
        ctx.closePath();
        ctx.fillStyle = '#000';
        ctx.stroke();
        ctx.clip();

        const gnt = ctx.createLinearGradient(endX,endY,realX,originY);
        gnt.addColorStop(0, '#E89B9B');
        gnt.addColorStop(0.5, '#A52A2A');
        gnt.addColorStop(1, '#A52A2A');
        ctx.fillStyle = gnt;
        ctx.fillRect(endX,endY,canvasWidth,originY);
        // 850
        console.log('originX=' + originX)
        console.log('endX=' + endX)
        var oval_first = this.fSoliLength/1
        var oval_second = 15
        var oval_third =  23
        var oval_fourth = 31
        var oval_fifth = 51
        var ovalEdge = 0.1
        this.drawFiostherms('#ec443a', oval_fifth )
        this.drawFiostherms('#ed5d3d', oval_fourth  )
        this.drawFiostherms('#ef7941', oval_third )
        this.drawFiostherms('#000', oval_second, ovalEdge );
        this.drawFiostherms('#f5b84e', oval_second );
        this.drawFiostherms('#fff', oval_first, ovalEdge );
        this.drawFiostherms('#fad955', oval_first )
        ctx.restore();
        this.drawHorizontalAxisTicks();
        this.drawVerticalAxisTicks();
		 // this.drawHorizantalCursor();

        //绘制游标值
        ctx.beginPath();
        ctx.strokeStyle = '#666';
		ctx.font = "16px Calibri";
        // ctx.moveTo(this.fSoliLength / 0.2 * spacingX + endX, endY);
        // ctx.lineTo(this.fSoliLength / 0.2 * spacingX + endX, originY);
        // ctx.setLineDash([8]);
        // ctx.lineDashOffset=-50;
        ctx.fillText(this.BottomfSoliLength, this.fSoliLength / 0.2 * spacingX + endX - 20, 10);
        ctx.stroke();
        console.log(this.fSoliLength / 0.2 * spacingX)
      },

	  drawHorizantalCursor(){
		var deltaX,num = 0, spacingX = parseFloat((realX / 120).toFixed(1));
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
		  // ctx.moveTo(endX + i * spacingX, originY - deltaX - endY);
		  // ctx.lineTo(endX + i * spacingX, originY - endY);



		  ctx.moveTo(endX + i * spacingX, originY - deltaX - 216 );
		  ctx.lineTo(endX + i * spacingX, originY - 216  );




		  ctx.stroke();
		  function text(){
		    ctx.font = "18px Calibri";
		    if(num > 9) {
		      ctx.fillText(num,endX + (i - 0.6) * spacingX, canvasHeight - 236);
		    }
			else if(num == 0) {
			  ctx.fillText(num,endX + (i - 0) * spacingX, canvasHeight);
			}
			else {
		      ctx.fillText(num,endX + (i - 0.2) * spacingX, canvasHeight);
		    }
		  }
		}
	  },

      drawHorizontalAxisTicks() {
        // var deltaX,num = 0, spacing = parseFloat((realX / 120).toFixed(1));
        // for (var i = 120;i > -1;i--){ //change 120 to 113 2003/09/07
        //   ctx.beginPath();
        //   ctx.strokeStyle = '#000';
        //   if(i % 20 === 0){
        //     deltaX = 8;
        //     text();
        //     num += 20;
        //   }if(i % 10 === 0){
        //     deltaX = 4;
        //   } else {
        //     deltaX = 0;
        //   }
        //   ctx.moveTo(originX - i * spacing, originY - deltaX);
        //   ctx.lineTo(originX - i * spacing, originY);
        //   ctx.stroke();
        //   function text(){
        //     ctx.font = "12px Calibri";
        //     if(num === 0) {
        //       ctx.fillText(num,originX - (i - 0.5) * spacing, originY + driftY);
        //     } else {
        //       ctx.fillText(num,originX - (i + 0.8) * spacing, originY + driftY);
        //     }
        //   }
        // }
        var deltaX,num = 0, spacingX = parseFloat((realX / 120).toFixed(1));
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
          ctx.moveTo(endX + i * spacingX, originY - deltaX);
          ctx.lineTo(endX + i * spacingX, originY);
          ctx.stroke();
          function text(){
            ctx.font = "12px Calibri";
            if(num > 9) {
              ctx.fillText(num,endX + (i - 0.6) * spacingX, canvasHeight);
            }
			else if(num == 0) {
			  ctx.fillText(num,endX + (i - 0) * spacingX, canvasHeight);
			}
			else {
              ctx.fillText(num,endX + (i - 0.2) * spacingX, canvasHeight);
			  // ctx.fillText(num,endX + (i - 0.5) * spacingX, canvasHeight);
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

	  drawFiostherms(color, num, c = 0){
	    console.log(`originX=${originX}`)
      console.log(`endX=${endX}`)
      var spacingX = parseFloat((realX / 120).toFixed(1));
      console.log(`spacingX=${spacingX}`)
      ctx.beginPath();
      ctx.moveTo(endX, endY);
      ctx.quadraticCurveTo((num + c) / 0.2 * spacingX * 2 + endX, realY / 2, endX, originY);
      ctx.strokeStyle = color;
      ctx.fillStyle = color;
      ctx.fill();
      ctx.stroke();
    },


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


    drawVerticalAxisTicks() {
        var deltaY, num = 0, spacing = parseFloat((realY / 20).toFixed(4));
        for (var i = 0; i < 20; i++) {
		      ctx.beginPath();
          ctx.strokeStyle = 'black';
          if (i % 5 === 0 ) {
            deltaY = 8;
            text();
            num += 5;
          } else {
            deltaY = 0;
          }
          ctx.moveTo(originX, originY - i * spacing );
          ctx.lineTo(originX - deltaY, originY - i * spacing);
          // ctx.stroke();

          function text() {
            ctx.font = "12px Calibri";
            // ctx.fillText(num, originX + 5, originY - (i - 0.1) * spacing);
          }
        }
      }
    }
  }



</script>

<style scoped>
  #fiostherms{
    width: 80vw;
    height: 12.63vw;
  }
</style>
