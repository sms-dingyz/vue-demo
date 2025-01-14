<template>
  <div class="echart-page">
    <div class="y-desc">Temperature[C]</div>
    <canvas id="myCanvas"/>
    <div style="margin-top: 20px;font-weight: bold">Distance from mold level[m]</div>
  </div>

</template>

<script>
  //import * as echarts from 'echarts';
  //import { getData } from './temperatureCurve'
  import { getLocalRawData } from './temperatureCurve'
  
  import {getCurveData} from './temperatureCurve'
  var canvas = null;
  var ctx = null;
  var canvasWidth = 0,
    canvasHeight = 0,
    originX = 0,
    originY = 0,
    endX = 0,
    endY = 0,
    driftX = 40, //  The width of the ordinate number
    driftY = 10,//  The height of the coordinate numbers
    realX = 0,//  The actual length of the horizontal coordinate
    realY = 0;//  The actual height of the ordinate

  //var gXAxisOffset = 128 ; //not good solution
  const rowLineCnt = 7 ; //maxLine

  //COLOR CONSTANT
  const CoreColor = "#e0190a"
  const MeanColor = "#a0264f"
  const SideColor = "#2196f3"
  const strokeBlackColor = 'black'
  const strokeGreyColor = 'grey'
  const fillBlackColor = 'black'
  const fillGreyColor = 'grey'

  const strokeColor = "grey"
  const strokeGreenColor = '#a2dba1'
  const fontSize12 = "12px Calibri"
  const fontSize16 = '16px Calibri'


   const CoreLineName = 'Core'
   const MeanLineName = 'Average'
   const SideLineName = 'Center loose'

   const SocketServer = 'ws://127.0.0.1:50000'

   const yAxisBase = 600
   const yAxisOffset = 100

   const verticalPortions = 100  //yAxsi
   const horizontalPortions = 24  //24

   const originxOffset = 30
   const originyOffset = 20

   const rowDataCounter = 240
   const scaleSpan = 9.6 
   const scaleStartPosx = 30 // 50



  const BottomOffset = 18 
  const slabLength = 34.5 


  var dpr = window.devicePixelRatio || 1;


  const isLocalData = 1
  export default {
    data() {
      return {
        // echartData: getData()  //test data
        echartData: {}
      };
    },
    async mounted() {
      canvas = document.getElementById("myCanvas");
      ctx = this.setupCanvas();

      if ( isLocalData === 1 ) {
       
      
        this.handleRawData(  ) 
      }
      else {
        await this.getMessageData()
        this.drawAxis();
      }
      
    },
    methods: {
      handleRawData () {
        this.echartData = getCurveData( getLocalRawData() )
        this.drawAxis()
      },
      getMessageData() {
        return new Promise((resolve) =>{
          //var messagetype = 'undefined'
          var socket = new WebSocket( SocketServer );
          socket.onmessage = (event) => {
            if(typeof event.data === 'string') {
              var tmpData = JSON.parse(event.data)
              //messagetype = tmpData.messagetype
            }

            if ( 'fTempCurve' === tmpData.messagetype ) {
              this.echartData = getCurveData( JSON.parse(event.data) )
              this.drawAxis()
            }
            resolve()
          }
        })
      },
  setupCanvas() {
    var rect = canvas.getBoundingClientRect();
    canvas.width = rect.width * dpr;
    canvas.height = rect.height * dpr;
    var context = canvas.getContext('2d');
    context.scale(dpr, dpr);
    return context
  },

	drawBackground(){

		ctx.fillStyle = 'white';
		//ctx.fillRect(originX, originY, endX + originX - gXAxisOffset + 3, endY);
		ctx.fillRect( scaleStartPosx , 0,canvasWidth,canvasHeight);
	
	},

    drawAxis() {
    //draw background
    this.drawBackground()


    //var xOffset = 20 
    canvasWidth = Number(ctx.canvas.width) / dpr;
    canvasHeight = Number(ctx.canvas.height) / dpr;
    originX = originxOffset;
    originY = canvasHeight - originyOffset;
    endX = canvasWidth - driftX;
    realX = endX - originX;
    realY = originY - driftY;
  //horizon
    ctx.beginPath();
    // ctx.moveTo(originX + 0, originY);
    // ctx.lineTo( canvasWidth - 50, originY );
    
    var starty = canvasHeight - BottomOffset;
    ctx.moveTo(scaleStartPosx, starty );
    ctx.lineTo( canvasWidth  , starty ) 
    ctx.fillColor = fillBlackColor
    ctx.stroke();

    endY = 0 - canvasHeight + driftY;
  //vertical
    ctx.beginPath();
    // ctx.moveTo(originX + xOffset , originY );
    // ctx.lineTo(originX + xOffset, endY);
    ctx.moveTo(scaleStartPosx , originY + 1 );
    ctx.lineTo(scaleStartPosx, endY + 1 );
    ctx.strokeStyle = strokeBlackColor;
    ctx.stroke();

    
    //this.drawHorizontalAxisTicks(this.echartData.fLiquidus);

    this.drawHorizontalAxisTicks();
    this.drawVerticalAxisTicks();
    this.draw_fLiquidus_fSolidus(this.echartData.fLiquidus, 'Liquidus')
    this.draw_fLiquidus_fSolidus(this.echartData.fSolidus, 'Solidus')
    this.draw(this.echartData.fTempCoreObj, CoreColor, CoreLineName , 0 );
    this.draw(this.echartData.fTempMeanObj, MeanColor, MeanLineName ,6 );
    this.drawSideLine(this.echartData.fTempSideObj, SideColor, SideLineName ,10 );
  },




	drawText( num , index , span ){
		var starty = canvasHeight
    var startx = scaleStartPosx
		ctx.font = fontSize12
		if( num > 9) {
			ctx.fillText( num ,startx + ( index - 0.8) * span - 2, starty );
		}
		else if( num === 0 ) {
			ctx.fillText( num ,startx + ( index - 0 ) * span, starty);
		}
		else {
			ctx.fillText( num ,startx + ( index - 0.2) * span - 2, starty);
		}
	},

  drawHorizontalAxisTicks(){
		var num = 0, spacingX = parseFloat((canvasWidth / rowDataCounter ).toFixed(1));
		var perScaleLength = canvasWidth/slabLength
		var span = (perScaleLength/10).toFixed(2)
		spacingX = span

    //patch
    spacingX = span - 0.08 //2025.1.13

		var deltaY = 8 
		var startx = 0 
		var starty = canvasHeight   	
		starty = canvasHeight - BottomOffset ;

    startx = scaleStartPosx
    var length = slabLength*10 
		for ( var i = 0;i < length ; i++ ){ 
			ctx.lineWidth = 1
			ctx.beginPath();
			if(i % 20 === 0){
				ctx.strokeStyle = fillBlackColor
				ctx.fillStyle = strokeBlackColor
				//text();

				this.drawText( num , i , spacingX );
				num += 2;				
				ctx.moveTo( startx + i * spacingX, starty );
				ctx.lineTo( startx + i * spacingX, starty + deltaY );
			}else {
				if ( i % 2 === 0 )
				{
          ctx.strokeStyle = fillGreyColor
          ctx.fillStyle = strokeGreyColor
					ctx.moveTo( startx + i * spacingX, starty );
					ctx.lineTo( startx + i * spacingX, starty + 4 );
				}
			}		
			ctx.stroke();
			
		}
	},  

  //drawHorizontalAxisTicks(yAxis) {	
	drawHorizontalAxisTicks1( ) {		
    var textDeltax = 18;
		var deltaX,num = 0, spacingX = parseFloat((canvasWidth / rowDataCounter ).toFixed(1));
		//console.log(" ______ dyz drawHorizontalAxisTicks spacingX:" + spacingX)
		
		var scaleStartx = scaleStartPosx // offset : 50
		var scaleStarty = 0	
		//var fillStyle_black = 'black'
		//var strokeStyle_black = 'black'
		//var fillStyle_grey = 'grey'
		var strokeStyle_grey = 'grey'
		spacingX = 9.6;//spacingX.toFixed(0);
		spacingX = 8.98
							
		scaleStarty = canvasHeight - 20
		

		//ctx.save()
		ctx.beginPath();
		for (var i = 0;i < 178 ;i++){ 
			ctx.lineWidth = 1
			ctx.fillStyle = 'black';
			if(i % 10 === 0 ){
				//ctx.fillStyle = fillStyle_black;
				deltaX = 8;
				text();
				num += 2;				
				ctx.moveTo(scaleStartx + i * spacingX, scaleStarty + deltaX  );
				ctx.lineTo(scaleStartx + i * spacingX, scaleStarty  );
		
			}else {
				deltaX = 4;
				ctx.fillStyle = strokeStyle_grey;
				ctx.moveTo(scaleStartx + i * spacingX, scaleStarty + deltaX );
				ctx.lineTo(scaleStartx + i * spacingX, scaleStarty );
			}

			ctx.stroke();

    
    } 

		function text(){
			ctx.font = fontSize12
      if(num > 9) {
			ctx.fillText(num,scaleStartx + (i - 0.6) * spacingX, scaleStarty + textDeltax );
			}
			else if(num == 0) {
				ctx.fillText(num,scaleStartx + (i - 0) * spacingX, scaleStarty+ textDeltax );
			}
			else {
				ctx.fillText(num,scaleStartx + (i - 0.2) * spacingX, scaleStarty+ textDeltax );
			}
		}
	//ctx.closePath()
	//ctx.restore()
	},  

  drawHorizontalAxisTicks2(yAxis) {
    var deltaX,num = 0, spacingX = parseFloat((realX / rowDataCounter ).toFixed(1)), spacingY = parseFloat((realY / verticalPortions ).toFixed(4));;     	
		spacingX = 10.5
		for (var i = 0;i < 150;i++){ 
      ctx.beginPath();
      ctx.fillStyle = 'grey' ;
      if(i % 10 === 0){
        deltaX = 8;
        text();
        num += 2;
        ctx.fillStyle = 'black' ;
      }else {
        deltaX = 4;
        ctx.fillStyle = 'grey' ;
      }
      ctx.moveTo(originX + i * spacingX, originY - deltaX);
      ctx.lineTo(originX + i * spacingX, originY);
      //ctx.stroke();
      // eslint-disable-next-line no-inner-declarations
      function text(){
        ctx.font = fontSize12;
        if(num > 9) {
          ctx.fillText(num,originX + (i - 0.6) * spacingX, canvasHeight);
        } else {
          ctx.fillText(num,originX + (i - 0.5) * spacingX, canvasHeight);
        }
      }
      ctx.stroke();
      drawRow();
      // eslint-disable-next-line no-inner-declarations
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

  drawVerticalAxisTicks() {
    var num = yAxisBase, spacing = parseFloat((realY / verticalPortions ).toFixed(4));
    var drawCnt = 0 ;	
    var startx = 0
    for (var i = 0; i < 1000; i++) {
      ctx.beginPath();
      ctx.fillStyle = strokeGreyColor;
      if (i % 10 === 0) {
        //deltaY = 8;
        //ctx.fillStyle = strokeBlackColor
        text();
        num += yAxisOffset; //100
  
      } else {
        //deltaY = 4;
        ctx.fillStyle = strokeGreyColor;
      }

      // ctx.moveTo(originX + deltaY + 20, originY - i * spacing);
      // ctx.lineTo(originX + 20 , originY - i * spacing);
      ctx.moveTo(startx, originY - i * spacing);
      ctx.lineTo(startx , originY - i * spacing);
      ctx.stroke();
      
      // eslint-disable-next-line no-inner-declarations
      function text() {
        ctx.font = fontSize12;
        // ctx.fillText(num , 0 + 20, originY - (i - 1) * spacing);
        ctx.fillText(num , 0 , originY - (i - 1) * spacing);
      }
	
      ctx.save()
      drawRow()
      // eslint-disable-next-line no-inner-declarations
      function drawRow() {
        if ( drawCnt < rowLineCnt ) {

    //ctx.save()
      if(i % 10 === 0 && i !== 0) {
        ctx.beginPath();
        ctx.strokeStyle = strokeGreenColor;
        if ( i !== 0 ) { //don't draw the first line because it override the bottom
          ctx.moveTo(startx + scaleStartPosx , originY - i * spacing);                             
          ctx.lineTo(canvasWidth    , originY - i * spacing);
          ctx.stroke();
        }
        drawCnt ++ ;
      }
    ctx.restore()
  }

  }
  }
      },
	

  drawVerticalAxisTicks1() {
    var deltaY, num = yAxisBase, spacing = parseFloat((realY / verticalPortions ).toFixed(4));
    var drawCnt = 0 ;

    for (var i = 0; i < 120; i++) {
      ctx.beginPath();
      ctx.fillStyle = fillGreyColor
      if (i % 10 === 0) {
        deltaY = 8;
        ctx.fillStyle = fillBlackColor
        text();
        num += yAxisOffset;
      } 
      else {
        deltaY = 4;
        ctx.fillStyle = fillGreyColor
      }
      ctx.moveTo(originX + deltaY, originY - i * spacing);
      ctx.lineTo(originX, originY - i * spacing);

      ctx.stroke();

    // eslint-disable-next-line no-inner-declarations
    function text() {
      ctx.font = fontSize12;
      ctx.fillText(num, 0, originY - (i - 1) * spacing);
    }
    drawRow();
    // eslint-disable-next-line no-inner-declarations
    function drawRow() {
    if ( drawCnt < rowLineCnt ) {

    ctx.save()
    if(i % 10 === 0 ) {
    ctx.beginPath();
    ctx.strokeStyle = strokeGreenColor;
    if ( i === 0 ) {
    ctx.strokeStyle = strokeColor;
    }
    ctx.moveTo(originX + deltaY, originY - i * spacing);               
    //ctx.lineTo(endX + originX - gXAxisOffset, originY - i * spacing);
    //ctx.lineTo(endX + originX , originY - i * spacing);
    ctx.lineTo(canvasWidth , originY - i * spacing);
    ctx.stroke();
    drawCnt ++ ;
    }
    ctx.restore()
    }

    }
    }
    },


	drawPoint(ctx, x, y, radius, fillColor) {
      ctx.beginPath();
      ctx.arc(x, y, radius, 0, 2 * Math.PI, false);
      ctx.fillStyle = fillColor;
      ctx.fill();
    },
 
   
  drawSideLine(list, color, name,nameyOffset) {
    var startxOffset = scaleStartPosx - 2
    var w = canvasWidth - 50 // 50
    var h = realY - 30
    var spacing = parseFloat((realY / verticalPortions ).toFixed(4));
    var hPortions = horizontalPortions
    // hPortions = 37
    hPortions = 34.1


    spacing = scaleSpan
    ctx.strokeStyle = color;
    ctx.lineWidth = '1';

    
		
		//ctx.save()
		ctx.beginPath();
		var item = list[0];
		//ctx.moveTo(item.x * realX  / hPortions + originX + 20 , realY - (item.y * realY / 1000 - 3 * spacing + 20));
    //ctx.moveTo(item.x * w  / hPortions + originX + 20 , h - (item.y * h / 1000 - 3 * spacing + 20));	
    ctx.moveTo(item.x * w  / hPortions + startxOffset , h - (item.y * h / 1000 - 3 * spacing + 15));
	
		for (let i = 0; i < list.length; i++) {
      
      item = list[i];			  
      //ctx.lineTo(item.x * realX / hPortions + originX + 20, realY - (item.y * realY / 1000 - 3 * spacing + 20));
      if ( item.y > -600 )
      // ctx.lineTo(item.x * w / hPortions + originX + 20, h - (item.y * h / 1000 - 3 * spacing + 20));
      ctx.lineTo(item.x * w / hPortions + startxOffset , h - (item.y * h / 1000 - 3 * spacing + 15));
      // 绘制点
      //this.drawPoint(ctx, item.x * w / hPortions + originX + 20 , h - (item.y * h / 1000 - 3 * spacing + 20), 1, 'red');
	
		}
		
	
		//fault-tolerant
		if ( list.length < 2 ) {
       return ;
		}
		const lastP = list[list.length - 2];
		ctx.font = fontSize16
		//ctx.fillStyle = color;
		ctx.fillText(name, lastP.x * realX / horizontalPortions + originX + 4 , realY - (lastP.y * realY / 1000 - 3 * spacing - nameyOffset  ));
		ctx.stroke()
		
		ctx.closePath()
		ctx.restore()
		
    },  
    
    draw(list, color, name,nameyOffset) {
      var w = canvasWidth - 50 // 50
      var h = realY + 70
      var spacing = parseFloat((realY / verticalPortions ).toFixed(4));
      var hPortions = horizontalPortions
      //hPortions = 37
      hPortions = 34.1

      spacing = scaleSpan
      ctx.strokeStyle = color;
      ctx.lineWidth = '1';

      //ctx.save()
      ctx.beginPath();
      var item = list[0];
		//ctx.moveTo(item.x * realX  / hPortions + originX + 20 , realY - (item.y * realY / 1000 - 3 * spacing + 20));
      ctx.moveTo(item.x * w  / hPortions + scaleStartPosx - 3 , h - (item.y * h / 1000 - 3 * spacing + 20));			
      for (let i = 0; i < list.length; i++) {
        item = list[i];			  
        //ctx.lineTo(item.x * realX / hPortions + originX + 20, realY - (item.y * realY / 1000 - 3 * spacing + 20));
        if ( item.y > -600 ){
          ctx.lineTo(item.x * w / hPortions + scaleStartPosx  - 3 , h - (item.y * h / 1000 - 3 * spacing + 20));
        }
        // 绘制点
        //this.drawPoint(ctx, item.x * w / hPortions + originX + 20 , h - (item.y * h / 1000 - 3 * spacing + 20), 1, 'red');
    
      }
		
	
      //fault-tolerant
      if ( list.length < 2 ) {
        return ;
      }
		const lastP = list[list.length - 2];
		ctx.font = fontSize16
		ctx.fillText(name, lastP.x * realX / horizontalPortions + originX + 4 , realY - (lastP.y * realY / 1000 - 3 * spacing - nameyOffset  ));
		ctx.stroke()
		
		ctx.closePath()
		ctx.restore()
		
    },  
	
    draw1(list, color, name,nameyOffset) {
        var spacing = parseFloat((realY / verticalPortions ).toFixed(4));
        ctx.strokeStyle = color;
        ctx.lineWidth = '1';
        ctx.beginPath();
        var item = list[0];
        ctx.moveTo(item.x * realX / horizontalPortions + originX, realY - (item.y * realY / 1000 - 3 * spacing));

        for (let i = 1; i < list.length; i++) {
          var item = list[i];
          ctx.lineTo(item.x * realX / horizontalPortions + originX, realY - (item.y * realY / 1000 - 3 * spacing));
        }
		
		//fault-tolerant
		if ( list.length < 2 ) {
		 return ;
		}
        const lastP = list[list.length - 2];
        ctx.font = fontSize16
        ctx.fillStyle = color;
        ctx.fillText(name, lastP.x * realX / horizontalPortions + originX + 4 , realY - (lastP.y * realY / 1000 - 3 * spacing - nameyOffset  ));
        ctx.stroke()
      },

        
    draw_fLiquidus_fSolidus( yAxis, name) {
      if ( this.echartData.fTempCoreObj === 0  ) {
        return
      }
      var spacing = parseFloat((realY / verticalPortions).toFixed(4));
      ctx.beginPath();
      ctx.strokeStyle = strokeGreyColor
      ctx.moveTo(scaleStartPosx, realY - ( yAxis * realY / 1000 - 3 * spacing));
      ctx.lineTo(canvasWidth - scaleStartPosx - 30 , realY - ( yAxis * realY / 1000 - 3 * spacing));
      ctx.font = fontSize16
      ctx.fillStyle = fillGreyColor
          //ctx.fillText(name, fEndX * realX / horizontalPortions + originX + 4, realY - ( yAxis * realY / 1000 - 3 * spacing));
      ctx.fillText(name, canvasWidth - scaleStartPosx - 25, realY - ( yAxis * realY / 1000 - 3 * spacing));
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
