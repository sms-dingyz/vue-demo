<template>
  <canvas id="cutview"/>
  <!-- <div style="margin-top: 20px;font-weight: bold">Solidification[mm]:{{this.BottomfSoliLength}}</div> -->
</template>

<script>
  import * as echarts from 'echarts/index';
  import { getData } from './fiostherm'
  import { getFiosthermData } from './fiostherm'
  
  import {getTemperatureData} from './data3'
  import {getTemperature3dData} from './data3d'
  
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
  
  
  
  var gXAxisOffset = 128 ; //not good solution
   const rowLineCnt = 7 ; //maxLine
   const rowDataCounter = 120
   var columnCounter = 50 
  const flexibleHeightOffset = 0  //100
  
  // const basicRGB_R = 238
  // const basicRGB_G = 126
  // const basicRGB_B = 48
  
  const basicRGB_R = 250
  const basicRGB_G = 217
  const basicRGB_B = 85
  
  const totalRow = 50 


  const SocketServer = 'ws://127.0.0.1:50000'

// https://github.com/szymslo/temp-color

  var dpr = window.devicePixelRatio || 1;
  export default {
    data() {
      return {
		  
		topLayertemperatureData: [],
		topLayerColor: "#fad955",
		secondLayerColor: "#f5b84e",
		
		
		temperatureColorData: [],
		temperatureData: [],
		FSolidus: "1480",
		SecondTemperature: 1400,
		FourthTemperature: 1100,
		FifthTemperature: 1000,
		SixthTemperature: 900,
		
		temperatureOffset: 100,
		
		FSolidusAxisX: 0,
		FSolidusAxisY: 0,
		spacingY: 0,
		tmpSolidus: 0,
		
		SecondBezierStartx: 0,
		SecondBezierStarty: 0,
		SecondBezierEndx: 0,
		SecondBezierEndy: 0,		
		SecondControlAxisx: 0,
		SecondControlAxisy: 0,
		
		
		FourthBezierStartx: 0,
		FourthBezierStarty: 0,
		FourthBezierEndx: 0,
		FourthBezierEndy: 0,		
		FourthControlAxisx: 0,
		FourthControlAxisy: 0,
		
		
		FifthBezierStartx: 0,
		FifthBezierStarty: 0,
		FifthBezierEndx: 0,
		FifthBezierEndy: 0,		
		FifthControlAxisx: 0,
		FifthControlAxisy: 0,
		
		SixthBezierStartx: 0,
		SixthBezierStarty: 0,
		SixthBezierEndx: 0,
		SixthBezierEndy: 0,		
		SixthControlAxisx: 0,
		SixthControlAxisy: 0,
		
		
		SecondStartAxisX: 0,
		SecondStartAxisY: 0,
		SecondEndAxisX: 0,
		SecondEndAxisY: 0,  
		  
		  
		  
		fSoliLength: 6.24,
		BottomfSoliLength:5300,
        echartData: [],
        colorObj: []
      };
    },
  

	mounted() {
	  
	  canvas = document.getElementById("cutview");
	  ctx = this.setupCanvas();
	  this.getServerTemperatureData()
	  this.convertTemperatrueToColor()
	  this.initiateParameter()
	  console.log("-----dyz aaa  cnt.linewidth:"+ctx.lineWidth)
	  this.drawFiostherm();
	  console.log("-----dyzffff  cnt.linewidth:"+ctx.lineWidth)
	},

    methods: {
	getServerTemperatureData() {
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
		
        // endX = 30 ;
        // realX = canvasWidth - driftX - originxOffset ;
        // realY = canvasHeight - driftY;

        endY = 30;
        var spacingX = parseFloat((realX / 120).toFixed(1))
		
		
		
		// 2.计算X轴终点的位置
		endX = 0;
		// 3.x，y实际长度
		realX = originX;
		realY = originY;
		
		// 4.绘制X轴		
		var xlineOffset = 20 ;
		var startX = 0 ;
		var startY = originY ;
		
		

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
        ctx.fillRect(endX,endY,canvasWidth,originY );
		
		console.log(" _________  this.temperatureColorData.length:"+  this.temperatureColorData.length)
		for (var i = 0; i < this.temperatureColorData.length ; i++) { // this.temperatureColorData.length -1
			// console.log(" " + i +":_________  this.temperatureColorData:"+  this.temperatureColorData[i])
			this.getElementInformation(this.temperatureColorData[i], i)
		}
		
		
		// this.getElementInformation(this.temperatureColorData[9], 9)
		// 	console.log(" " + 9 +":_________  this.temperatureColorData:"+  this.temperatureColorData[9])
		
        var oval_second = 15
        var oval_third =  23
        var oval_fourth = 31
        var oval_fifth = 51
        var ovalEdge = 0.1
		
		// this.drawBackgroud('#ec443a', oval_fifth )
		// this.drawSecondBackgroud('#ed5d3d', oval_fourth  )
		// this.drawThirdBackgroud('#ed5d3d', oval_third  )
		
		
		
		//draw top layer
		// ctx.beginPath();
		ctx.strokeStyle = this.rgbToHex(242,42,36 );
		ctx.strokeStyle = this.topLayerColor
		let temArray = []	   
		let tempItem ;
		
		var lineWidth = ctx.lineWidth.toFixed(2) 
		
		var mIndex = 0 
		var colorIndex = 0
		var row = 0 
		
		var itemOffset = 0  // 24
		console.log("________dyz lineWidth:"+ lineWidth )
		console.log("________dyz lineWidth:"+ctx.lineWidth +" len:"+ this.topLayertemperatureData.length)
		this.topLayertemperatureData.forEach((item,index)=>{	
			
			
			
			if ( index > 0 &&  ( index % rowDataCounter ) === 0 ) {
				row ++ 			
				colorIndex ++ 
				mIndex = 0 
			}
			
			
			
			ctx.beginPath();
			ctx.lineWidth = lineWidth 
			
			
			if ( row === 0 ) {
				ctx.lineWidth = lineWidth + itemOffset
				console.log("________dyz index:" + index+ " ctx.lineWidth:"+ ctx.lineWidth)
				console.log("--- dyz index:" + index +  "  startx:"+item.startx +" starty:"+ item.starty
							+ " endx:" + item.endx + " endy:" + item.endy +  " value:"+item.value)  
				
			}
			
			
			// console.log("________dyz index:" + index+ " ctx.lineWidth:"+ ctx.lineWidth)
			var color = this.temperatureColorData[colorIndex]
			
			if ( colorIndex > 8  ) {
				// console.log(" ------color:"+color)
			}
			
			// if ( row === (totalRow -1) ){
			// console.log("--- dyz index:" + index + " row:" + row + " colorIndex:" + colorIndex + " mIndex:" + mIndex + "  startx:"+item.startx +" starty:"+ item.starty
			// 			+ " endx:" + item.endx + " endy:" + item.endy + " color"+ color[mIndex] +  " value:"+item.value)  
			// }
			
			
			ctx.strokeStyle = color[mIndex]
			ctx.fillStyle = color[mIndex]
			
			
			// console.log("color[" + index + "]:" + color[index] )  
			if ( item.value  >= 0 ) { // 1250
				// console.log(" ------item.value:"+item.value)
				if ( mIndex === 0 ){
				// console.log("index:" + index + " row:" + row + " colorIndex:" + colorIndex + " mIndex:" + mIndex + "  startx:"+item.startx +" starty:"+ item.starty
				// 			+ " endx:" + item.endx + " color"+ color[row] +  " value:"+item.value)  
				}
		
		
			// ctx.moveTo( item.startx -0.8 , item.starty - itemOffset );
			ctx.moveTo( item.startx -0.8 , item.starty - itemOffset );
			ctx.lineTo( item.endx, item.endy  - itemOffset );		
			
			// ctx.rect(item.startx,item.starty,item.endx,item.endy)
			
			ctx.closePath()
			ctx.stroke()
			}
			
			
			
			mIndex ++ 
			
			if ( index ===  0 ) {
				tempItem = item
				//temArray.push( item )
			}
			else {	
				if ( item.index !== tempItem.index ) {				
					temArray.push( tempItem )
					if ( index === this.topLayertemperatureData.length -1 ) {
						temArray.push( item )
					}
				}
				tempItem = item
			}
				
		});
		
		
		// this.drawTopLayer() 
        //ctx.restore();

      },
	  
	getElementInformation(data ,index){

		var startyOffset = 60 // 44  115 
		var xlineOffset = 20
		  
		var height = realY - 126 //70
		// var spacingX = parseFloat((realX / 120).toFixed(1)), spacingY = parseFloat((height / 20).toFixed(4));
		var spacingX = parseFloat(( (realX - xlineOffset) / rowDataCounter).toFixed(1))
		var spacingY = parseFloat(( height  / columnCounter).toFixed(4));
		spacingY = spacingY + 2
		
		ctx.lineWidth = spacingY.toFixed(0); 
		// spacingX = spacingX.toFixed(0); 
		// spacingY = spacingY.toFixed(0); 
		
		this.spacingY = spacingY 
		// console.log(" ______ spacingX:" + spacingX)
		// console.log(" ______ spacingY:" + spacingY)
		// console.log(" ______ dyz ctx.lineWidth:" + ctx.lineWidth)
		
		var secondAxisX = 0 
		var tmpSolidus = this.FSolidus 		
		var temperatureData = this.temperatureData[index]
		
		
		var offsetx = 8 		
		// ctx.beginPath();
	
	
	
		var startx = 0
		var starty = 0
		var endx = 0
		var endy = 0
		
		var elementStartyOffset = 4 
		
		if ( index > 48 ) {
			elementStartyOffset = 0 
		}
		
		// console.log(" ______ realY:" + realY)
		// console.log(" ______ endX:" + endX)
		// console.log(" ______ originY:" + originY)
		// console.log(" ______ spacingX:" + spacingX)
		// console.log(" ______ spacingY:" + spacingY)
		// console.log(" ______ index:" + index)
		for (var i = 0; i < data.length  ; i++) {
			// console.log(" ______ data[" + i + "]:" + data[i])
			// ctx.beginPath();
			// ctx.lineWidth = spacingY;
			ctx.strokeStyle = data[i];
			
		
			// var startx = endX + i * spacingX - offsetx
			// var starty = originY - spacingY * index - spacingX 
			// var endx = endX + (i + 1) * spacingX  - offsetx
			// var endy = originY - spacingY * index - spacingX
			// ctx.moveTo(endX + i * spacingX - offsetx , originY - spacingY * index - spacingX  );
			// ctx.lineTo(endX + (i + 1) * spacingX  - offsetx , originY - spacingY * index - spacingX  );
			
			startx = i*spacingX 		
			startx = Number(startx.toFixed(1))			
			// starty =  startyOffset + index*spacingY
			starty = originY -  index*spacingY - elementStartyOffset
			
			endx += spacingX
			endx = Number(endx.toFixed(1))
			
			// endy =  startyOffset + index *spacingY
			endy = originY -  index*spacingY
			
			
			// endy = starty
			
			// ctx.moveTo( startx , starty  );
			// ctx.lineTo( endx ,endy );
			// ctx.stroke();
			// ctx.closePath()
			
			if ( index === 0 ) {
				console.log(" dyz i:" + index + "______ startx:" + startx + " starty:"+starty + " endx:" + endx + " endy:"+endy)
				
			}
			// console.log(" i:" + i + " ______ endx:" + endx + " endy:"+endy)
			//remember the cooridinate system of Temperature solidification 
			
			// this.FSolidus = Number(temperatureData[0])
		
			
			if ( Number( temperatureData[i] ) >= Number( this.FSolidus)  ) {
				
				this.topLayertemperatureData.push(
					{
					index: index,
					startx: startx, starty: starty, 
					endx: endx , endy: endy , 
					value:  Number( temperatureData[i] )					
					}
				)
				
				if ( this.tmpSolidus == 0  ) {
					this.tmpSolidus =  Number( temperatureData[i] )
					this.FSolidusAxisX = endX + (i + 1) * spacingX  - offsetx
					this.FSolidusAxisY = originY - spacingY * index - spacingX 
				}
				else							
				{
					if (  Number( temperatureData[i] ) < this.tmpSolidus  ) {
						
						this.tmpSolidus =  Number( temperatureData[i] )
						this.FSolidusAxisX = endX + (i + 1) * spacingX  - offsetx
						this.FSolidusAxisY = originY - spacingY * index - spacingX 
					}
				}			
			}
			else if (  Number( temperatureData[i] ) >= this.SecondTemperature
				&&  Number( temperatureData[i] )  < (this.SecondTemperature + this.temperatureOffset ) ) {	
					this.topLayertemperatureData.push(
						{
						index: index,
						startx: startx, starty: starty, 
						endx: endx , endy: endy , 
						value:  Number( temperatureData[i] )					
						}
					)
					
				var tmpx = endX + (i + 1) * spacingX  -offsetx
				var tmpy = originY - spacingY * index - spacingX
				if ( this.SecondBezierStartx === 0  ) {	
					//start point of bezier
					this.SecondBezierStartx = (endX + i * spacingX - offsetx ).toFixed(1)
					this.SecondBezierStarty = (originY - spacingY * index - spacingX).toFixed(1)				
				}
				if ( index !== 0 ) {
					//end point of bezier
					if ( this.SecondBezierEndy === 0  ) {
						this.SecondBezierEndy = tmpy.toFixed(1)
					}
					else {
						if ( tmpy < this.SecondBezierEndy ) {
							this.SecondBezierEndx = tmpx.toFixed(1)
							this.SecondBezierEndy = tmpy.toFixed(1)
						}
					}
				}
			
				var x = endX + (i + 1) * spacingX  - offsetx 
				if ( this.SecondControlAxisx === 0 ) {					
					this.SecondControlAxisx = endX + (i + 1) * spacingX - offsetx 
					this.SecondControlAxisx = this.SecondControlAxisx.toFixed(1)
				}
				else {
					if (x > this.SecondControlAxisx ) {
						secondAxisX = endX + (i + 1) * spacingX - offsetx 
						this.SecondControlAxisx = secondAxisX.toFixed(1)
						this.SecondControlAxisy = originY - spacingY * index - spacingX*2
						this.SecondControlAxisy = this.SecondControlAxisy.toFixed(1)
					}									
				}
			}
			else if (  Number( temperatureData[i] ) >= this.FourthTemperature 
				&&  Number( temperatureData[i] )  < (this.FourthTemperature + this.temperatureOffset ) ) {	
					this.topLayertemperatureData.push(
						{
						index: index,
						startx: startx, starty: starty, 
						endx: endx , endy: endy , 
						value:  Number( temperatureData[i] )					
						}
					)
				var tmpx = endX + (i + 1) * spacingX - offsetx
				var tmpy = originY - spacingY * index - spacingX
				if ( this.FourthBezierStartx === 0  ) {	
					//start point of bezier
					this.FourthBezierStartx = (endX + i * spacingX - offsetx ).toFixed(1)
					this.FourthBezierStarty = (originY - spacingY * index - spacingX).toFixed(1)				
				}
				if ( index !== 0 ) {
					//end point of bezier
					if ( this.FourthBezierEndy === 0  ) {
						this.FourthBezierEndy = tmpy.toFixed(1)
					}
					else {
						if ( tmpy < this.FourthBezierEndy ) {
							this.FourthBezierEndx = tmpx.toFixed(1)
							this.FourthBezierEndy = tmpy.toFixed(1)
						}
					}
				}
			
			var x = endX + (i + 1) * spacingX - offsetx 
			if ( this.FourthControlAxisx === 0 ) {					
								
				this.FourthControlAxisx = endX + (i + 1) * spacingX - offsetx 
				this.FourthControlAxisx = this.FourthControlAxisx.toFixed(1)
			}
			else {
				if (x > this.FourthControlAxisx ) {
					secondAxisX = endX + (i + 1) * spacingX - offsetx 
					this.FourthControlAxisx = secondAxisX.toFixed(1)
					this.FourthControlAxisy = originY - spacingY * index - spacingX*2
					this.FourthControlAxisy = this.FourthControlAxisy.toFixed(1)
				}									
			}
		}
		else if (  Number( temperatureData[i] ) >=  this.FifthTemperature
			&&  Number( temperatureData[i] )  < (this.FifthTemperature + this.temperatureOffset ) ) {	
			this.topLayertemperatureData.push(
				{
				index: index,
				startx: startx, starty: starty, 
				endx: endx , endy: endy , 
				value:  Number( temperatureData[i] )					
				}
			)
				
			var tmpx = endX + (i + 1) * spacingX - offsetx
			var tmpy = originY - spacingY * index - spacingX
			if ( this.FifthBezierStartx === 0  ) {	
				//start point of bezier
				this.FifthBezierStartx = (endX + i * spacingX - offsetx).toFixed(1)
				this.FifthBezierStarty = (originY - spacingY * index - spacingX).toFixed(1)				
			}
			if ( index !== 0 ) {
				//end point of bezier
				if ( this.FifthBezierEndy === 0  ) {
					this.FifthBezierEndy = tmpy.toFixed(1)
				}
				else {
					if ( tmpy < this.FifthBezierEndy ) {
						this.FifthBezierEndx = tmpx.toFixed(1)
						this.FifthBezierEndy = tmpy.toFixed(1)
					}
				}
			}
		
			var x = endX + (i + 1) * spacingX - offsetx 
			if ( this.FifthControlAxisx === 0 ) {					
				this.FifthControlAxisx = endX + (i + 1) * spacingX - offsetx 
			}
			else {
				if (x > this.FifthControlAxisx ) {
					secondAxisX = endX + (i + 1) * spacingX - offsetx 
					this.FifthControlAxisx = secondAxisX
					this.FifthControlAxisy = originY - spacingY * index - spacingX*2
				}									
			}
		}
		else if (  Number( temperatureData[i] ) >=  this.SixthTemperature
			&&  Number( temperatureData[i] )  < (this.SixthTemperature + this.temperatureOffset ) ) {	
				
			this.topLayertemperatureData.push(
				{
					index: index,
					startx: startx, starty: starty, 
					endx: endx , endy: endy , 
					value:  Number( temperatureData[i] )					
				}
			)
				
				
			var tmpx = endX + (i + 1) * spacingX - offsetx
			var tmpy = originY - spacingY * index - spacingX
			if ( this.SixthBezierStartx === 0  ) {	
				//start point of bezier
				this.SixthBezierStartx = (endX + i * spacingX - offsetx ).toFixed(1)
				this.SixthBezierStarty = (originY - spacingY * index - spacingX).toFixed(1)				
			}
			if ( index !== 0 ) {
				//end point of bezier
				if ( this.SixthBezierEndy === 0  ) {
					this.SixthBezierEndy = tmpy.toFixed(1)
				}
				else {
					if ( tmpy < this.SixthBezierEndy ) {
						this.SixthBezierEndx = tmpx.toFixed(1)
						this.SixthBezierEndy = tmpy.toFixed(1)
					}
				}
			}
		
			var x = endX + (i + 1) * spacingX - offsetx 
			if ( this.SixthControlAxisx === 0 ) {					
				this.SixthControlAxisx = endX + (i + 1) * spacingX - offsetx 
			}
			else {
				if (x > this.SixthControlAxisx ) {
					secondAxisX = endX + (i + 1) * spacingX - offsetx 
					this.SixthControlAxisx = secondAxisX
					this.SixthControlAxisy = originY - spacingY * index - spacingX*2
				}									
			}
		}
		else {
			this.topLayertemperatureData.push(
				{
				index: index,
				startx: startx, starty: starty, 
				endx: endx , endy: endy , 
				value:  Number( temperatureData[i] )					
				}
			)
		}		
	
		
	}
	

	},

	  
	drawBackgroud(color, num ){
		var spacingX = parseFloat((realX / 120).toFixed(1));
		ctx.beginPath();
		ctx.moveTo(endX , endY);
		
		console.log("---cx:" + num / 0.2 * spacingX * 2  +" cy:" + realY / 2)
		console.log("---ex:" + endX +" ey:" + originY)
		// ctx.quadraticCurveTo((num + c) / 0.2 * spacingX * 2 + endX, realY / 2, endX, originY);
		var control_startx = num / 0.2 * spacingX * 2 + endX
		control_startx = control_startx*2
		var control_starty = originY
		ctx.quadraticCurveTo( control_startx , control_starty, endX , originY);
		ctx.strokeStyle = color;
		ctx.fillStyle = color;
		ctx.fill();
		ctx.stroke();
	},
	drawSecondBackgroud(color, num ){
	  var spacingX = parseFloat((realX / 120).toFixed(1));
	  ctx.beginPath();
	  ctx.moveTo(endX , endY);
	  // ctx.quadraticCurveTo((num + c) / 0.2 * spacingX * 2 + endX, realY / 2, endX, originY);
	  var control_startx = num / 0.2 * spacingX * 2 + endX
	  control_startx = control_startx*2
	  var control_starty = originY
	  ctx.quadraticCurveTo( control_startx , control_starty, endX , originY);
	  ctx.strokeStyle = color;
	  ctx.fillStyle = color;
	  ctx.fill();
	  ctx.stroke();
	},

	
	drawThirdBackgroud(color, num ){
		
	const gnt = ctx.createLinearGradient(endX,endY,realX,originY);
	// gnt.addColorStop(0, '#ff9525');
	// gnt.addColorStop(0.5, '#FF7020');
	// gnt.addColorStop(1, '#FF7225');
	
	
	
	
	// gnt.addColorStop(0, '#ffb725');
	// gnt.addColorStop(0, '#ffd17d');
	// gnt.addColorStop(0.5, '#FF7020');
	
	
	gnt.addColorStop(0, '#ed5d3d');
	gnt.addColorStop(0.5, '#ffd17d');
	gnt.addColorStop(1, '#FF7225');
	ctx.fillStyle = gnt;
	ctx.strokeStyle = gnt;
		
		
	  var spacingX = parseFloat((realX / 120).toFixed(1));
	  ctx.beginPath();
	  ctx.moveTo(endX , endY);
	  var control_startx = num / 0.2 * spacingX * 2 + endX
	  control_startx = control_startx*2
	  var control_starty = originY
	  ctx.quadraticCurveTo( control_startx , control_starty, endX , originY );
	  // ctx.strokeStyle = "#ffffff";
	  // ctx.fillStyle = "#ffffff";
	  ctx.fill();
	  ctx.stroke();
	  ctx.closePath()
	  
	
	  
	  
	},


	  
	convertTemperatrueToColor(){
		var temperature = []
		var colors = []
		var temperatureData = getTemperature3dData( 10 )
		console.log(" ________ temperatureData.length:"+temperatureData.length)
		columnCounter = temperatureData.length/rowDataCounter
		
		var firstTemperature = temperatureData[0].toFixed(1)
		
		console.log(" ________ columnCounter:"+columnCounter)
		console.log(" ________ firstTemperature:"+firstTemperature)
		
		var row = 0 
		temperatureData.forEach((item,index)=>{ 
					
					
					
					item = item.toFixed(2)
					temperature.push(item)
					
					item = Number(item)
					firstTemperature = Number(firstTemperature)
					firstTemperature = 1480
					row = Number( (index/120).toFixed(0) )
					var rgb = this.tempToColor(item,900,1800,"")
					colors.push(this.rgbToHex(rgb.r,rgb.g,rgb.b))
					// console.log(" ________ index:"+ index + " row:" +row)
					
					// else if (row ===8){
					// 	firstTemperature = 1283
					// }
					// else if ( row === 5){
					// 	firstTemperature = 1484
					// }
					
					// console.log(" ________ index:"+ index + " item:" +item)
					if (item >= firstTemperature ) {
						
						// const basicRGB_R = 250
						// const basicRGB_G = 217
						// const basicRGB_B = 85
						////colors.push(this.rgbToHex(basicRGB_R,basicRGB_G,basicRGB_B))	
									
					}
					else if ( item >= 1400 && item < firstTemperature ){
						// colors.push(this.rgbToHex(242,42,36 )) 
						// colors.push(this.rgbToHex(basicRGB_R,basicRGB_G,basicRGB_B -10 )) 
						// colors.push(this.rgbToHex(238,126,48 )) 
						////colors.push(this.rgbToHex(238,126,48))
						
					
					}
					else if ( item >= 1300 && item < 1400 ){
						// colors.push(this.rgbToHex(240,140 ,44))
						////colors.push(this.rgbToHex(255,183 ,44))
						
					}
					else if ( item >= 1200 && item < 1300 ){
						// colors.push(this.rgbToHex(242,141 ,23))
						////colors.push(this.rgbToHex(255,183 ,44))
						
					}
					
					
					
					else if ( item >= 1100 && item < 1200 ){
						////colors.push(this.rgbToHex(248,165 ,20))
					}
					else if ( item >= 1000 && item < 1100 ){
						////colors.push(this.rgbToHex(253,182 ,7))
					}
					else if ( item >= 900 && item < 1000 ){
									
						////colors.push(this.rgbToHex(254,180 ,4))	
					}
					else{
						// colors.push(this.rgbToHex(254,180 ,4))	
					}
					
					
					
				});
				
					
				
				let arr = [],list = [];
				colors.forEach(c => {
					arr.push(c)
					if ( arr.length >= rowDataCounter ) {
						list.push(arr)
						arr = []
					}		
				})
				this.temperatureColorData = [...list]
				// console.log("_____ temperatureColorData.length:" + this.temperatureColorData[0].length)
				// console.log("_____ temperatureColorData0:" + this.temperatureColorData[0])
				// console.log("_____ temperatureColorData1:" + this.temperatureColorData[1])
				// console.log("_____ temperatureColorData2:" + this.temperatureColorData[2])
				// console.log("_____ temperatureColorData3:" + this.temperatureColorData[3])
				// console.log("_____ temperatureColorData4:" + this.temperatureColorData[4])
				// console.log("_____ temperatureColorData5:" + this.temperatureColorData[5])
				// console.log("_____ temperatureColorData6:" + this.temperatureColorData[6])
				// console.log("_____ temperatureColorData7:" + this.temperatureColorData[7])
				// console.log("_____ temperatureColorData8:" + this.temperatureColorData[8])
				// console.log("_____ temperatureColorData9:" + this.temperatureColorData[9])
				
				arr = []
				list = []
				
				
				temperature.forEach(t => {
					arr.push(t)
					if ( arr.length >= rowDataCounter ) {
						list.push(arr)
						arr = []
					}		
				})
				this.temperatureData = [...list]
				
				//handle edge
		  
		  
	  },	
	  decToHex(dec) {
	  		return dec.toString(16);
	  },
	  padToTwo(str) {
	  		return str.padStart(2, '0')
	  },
	  
	  rgbToHex(r, g, b) {
	  		const hexR = this.padToTwo(this.decToHex(r));
	  		const hexG = this.padToTwo(this.decToHex(g));
	  		const hexB = this.padToTwo(this.decToHex(b));		
	  		return `#${hexR}${hexG}${hexB}`
	  },
	initiateParameter(){
	  	// 4.拿到canvas的宽高
	  	canvasWidth = Number(ctx.canvas.width) / dpr;
	  	canvasHeight = Number(ctx.canvas.height) / dpr;
	  		
	  	// 1.计算坐标系原点的位置
	  	originX = canvasWidth - driftX;
	  	originY = canvasHeight - driftY;
	  	// 2.计算X轴终点的位置
	  	endX = 0;
	  	// 3.x，y实际长度
	  	realX = originX;
	  	realY = originY;
	},

	drawBezierLayer(color, startx,starty, 
	  						controlx,controly, endx, endy ){
	  	
	  
	  const gnt = ctx.createLinearGradient(startx,starty,endx,endy);
	  gnt.addColorStop(0, '#E89B9B');
	  gnt.addColorStop(0.5, '#A52A2A');
	  gnt.addColorStop(1, '#A52A2A');
	  // ctx.fillStyle = gnt;
	  
	    ctx.beginPath();
	    ctx.moveTo(startx , starty);
	    
	    
	    ctx.quadraticCurveTo( controlx*2 - startx  , controly, endx , endy );
	    ctx.strokeStyle = color;
	    ctx.fillStyle = color;
	    ctx.fill();
	    ctx.stroke();
	},
	
	
	
	
	getColorFromTemperature(kelvin){
		
		console.log(" ________ kelvin:" +kelvin)
		 var temperature = kelvin / 10.0;
		  var red, green, blue;
		
		  if (temperature <= 66.0) {
		    red = 0;
		  } else 
		  {
		    red = temperature - 60.0;
		    red = 329.698727446 * Math.pow(red, -0.1332047592);
		    if (red < 0) red = 0;
		    if (red > 255) red = 255;
		  }
		
		  /* Calculate green */
		
		  if (temperature <= 66.0) {
		    green = temperature;
		    green = 99.4708025861 * Math.log(green) - 161.1195681661;
		    if (green < 0) green = 0;
		    if (green > 255) green = 255;
		  } else 
		  {
		    green = temperature - 60.0;
		    green = 288.1221695283 * Math.pow(green, -0.0755148492);
		    if (green < 0) green = 0;
		    if (green > 255) green = 255;
		  }
		
		  /* Calculate blue */
		
		  if (temperature >= 66.0) {
		    blue = 255;
		  } else 
		  {
		
		    if (temperature <= 19.0) {
		      blue = 0;
		    } else {
		      blue = temperature - 10;
		      blue = 138.5177312231 * Math.log(blue) - 305.0447927307;
		      if (blue < 0) blue = 0;
		      if (blue > 255) blue = 255;
		    }
		  }
		
		  return {red: Math.round(red), blue: Math.round(blue), green: Math.round(green)};
		  // return this.rgbToHex(red,green,blue)	
		},
		
		
		
		tempToColor (t, min, max, mode) {
			  if (!Number.isFinite(t) || !Number.isFinite(min) || !Number.isFinite(max)) {
			    log.Error("--temperature beyond range")
			  }
			
			  if (min > max) {
			    log.Error("--temperature min > max")
			  }
			
			  if (t < min) {
			    t = min;
			  } else if (t > max) {
			    t = max;
			  }
			
			  const nT = (t - min) / (max - min);
			  let rValue = 255;
			  let gValue = 255;
			  let bValue = 255;
			
			  switch (mode) {
			    case 'extended': {
			      const regions = [1 / 6, (1 / 6) * 2, (1 / 6) * 3, (1 / 6) * 4, (1 / 6) * 5];
			      if (nT <= regions[0]) {
			        rValue = 128 - 6 * nT * 127.999;
			        gValue = 0;
			        bValue = 255;
			      } else if (nT > regions[0] && nT <= regions[1]) {
			        rValue = 0;
			        gValue = 1280 - 6 * (1 - nT) * 255.999;
			        bValue = 255;
			      } else if (nT > regions[1] && nT <= regions[2]) {
			        rValue = 0;
			        gValue = 255;
			        bValue = 768 - 6 * nT * 255.999;
			      } else if (nT > regions[2] && nT <= regions[3]) {
			        rValue = 768 - 6 * (1 - nT) * 255.999;
			        gValue = 255;
			        bValue = 0;
			      } else if (nT > regions[3] && nT <= regions[4]) {
			        rValue = 255;
			        gValue = 1280 - 6 * nT * 255.999;
			        bValue = 0;
			      } else {
			        rValue = 255;
			        gValue = 0;
			        bValue = 128 - 6 * (1 - nT) * 127.999;
			      }
			      break;
			    }
			
			    case 'half': {
			      const regions = [1 / 4, (1 / 4) * 2, (1 / 4) * 3];
			      if (nT <= regions[0]) {
			        rValue = 0;
			        gValue = 128 + 4 * nT * 127.999;
			        bValue = 0;
			      } else if (nT > regions[0] && nT <= regions[1]) {
			        rValue = 768 - 4 * (1 - nT) * 255.999;
			        gValue = 255;
			        bValue = 0;
			      } else if (nT > regions[1] && nT <= regions[2]) {
			        rValue = 255;
			        gValue = 768 - 4 * nT * 255.999;
			        bValue = 0;
			      } else {
			        rValue = 128 + 4 * (1 - nT) * 127.999;
			        gValue = 0;
			        bValue = 0;
			      }
			      break;
			    }
			
			    default: {
					// const basicRGB_R = 250
					// const basicRGB_G = 217
					// const basicRGB_B = 85
					
			      const regions = [1 / 4, (1 / 4) * 2, (1 / 4) * 3];
			      if (nT <= regions[0]) {
			        rValue = 0;
			        gValue = 4 * nT * 255.999;
			        bValue = 255;
			      } else if (nT > regions[0] && nT <= regions[1]) {
			        rValue = 0;
			        gValue = 255;
			        bValue = 512 - 4 * nT * 255.999;
			      } else if (nT > regions[1] && nT <= regions[2]) {
			        rValue = 512 - 4 * (1 - nT) * 255.999;
			        gValue = 255;
			        bValue = 0;
			      } else {
			        rValue = 255;
			        gValue = 4 * (1 - nT) * 255.999;
			        bValue = 0;
			      }
			      break;
			    }
			  }
			
			  return {
			    r: this.checkRange(Math.trunc(rValue)),
			    g: this.checkRange(Math.trunc(gValue)),
			    b: this.checkRange(Math.trunc(bValue))
			  }
			// }

		},
		checkRange (value) {
		  if (value <= 0) {
		    return 0;
		  }
		  if (value > 255) {
		    return 255;
		  }
		  return value;
		}
		
	  
	  
    }
  }



</script>

<style scoped>
  #cutview{
   /* width: 80vw;
    height: 12.63vw; */
	width: 80%;
	height: 360px;
  }
</style>
