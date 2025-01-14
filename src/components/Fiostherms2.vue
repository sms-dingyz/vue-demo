<template>
  <canvas id="fiostherms"/>

</template>

<script>
  import * as echarts from 'echarts';
  import { getData } from './data3'
  import {getTemperatureData} from './data3'
  var canvas = null;
  var ctx = null;
  var canvasWidth = 0,
    canvasHeight = 0,
    originX = 0, //  坐标系原点横坐标值
    originY = 0, //  坐标系原点纵坐标值
    endX = 0, //  横坐标终点
    endY = 0, //  纵坐标终点
    driftX = 40, //  纵坐标数字的宽度（横向空出的距离，600、700、800的宽度）
    driftY = 30,//  横坐坐标数字的高度（纵向空出的距离，0、2、4的高度）
    realX = 0,//  横坐标实际长
    realY = 0;//  纵坐标实际高

  var gXAxisOffset = 128 ; //not good solution
  const rowLineCnt = 7 ; //maxLine
  const rowDataCounter = 120
  
 const flexibleHeightOffset = 100 
 
 const basicRGB_R = 238
 const basicRGB_G = 126
 const basicRGB_B = 48
 // 238 126 48    241 138 39
 
  var dpr = window.devicePixelRatio || 1;
  export default {
    data() {
      return {
		
		topLayertemperatureData: [],
		topLayerColor: "#fad955",
		secondLayerColor: "#f5b84e",
		
		
        temperatureColorData: [],
		temperatureData: [],
		FSolidus: "1497",
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
		
		
      };
    },
    mounted() {
		
	 
      this.convertTemperatrueToColor()
      // this.temperatureColorData = getData()
      canvas = document.getElementById("fiostherms")
      ctx = this.setupCanvas()
	  this.initiateParameter()
	  
      this.drawAxis()
	  var oval_second = 15
	  var oval_third =  23
	  var oval_fourth = 31
	  var oval_fifth = 51
	  var ovalEdge = 0.1
	  this.drawLayerBackGround('#ec443a', oval_fifth )
	  // this.drawLayerBackGround('#ed5d3d', oval_fourth  )
	  // this.drawLayerBackGround('#ef7941', oval_third )
	  
	  
	  
	
	  // this.drawFiostherms('#ec443a', oval_fifth )
	  // this.drawFiostherms('#ed5d3d', oval_fourth  )
	  // this.drawFiostherms('#ef7941', oval_third )
	  
	  
	  
	
	   this.drawBezierSixLayer('#ec443a',
	   this.SecondBezierStartx,this.SixthBezierStarty, 
	   this.SixthControlAxisx,this.SixthControlAxisy,
	   this.SecondBezierEndx,this.SixthBezierEndy)
	   
	   this.drawBezierLayer('#ed5d3d',
	   this.SecondBezierStartx,this.FifthBezierStarty, 
	   this.FifthControlAxisx,this.FifthControlAxisy,
	   this.SecondBezierEndx,this.FifthBezierEndy)
	   
	   this.drawBezierLayer('#ef7941',
	   this.SecondBezierStartx,this.FourthBezierStarty, 
	   this.FourthControlAxisx,this.FourthControlAxisy,
	   this.SecondBezierEndx,this.FourthBezierEndy)
	  
	   this.drawBezierLayer('#f5b84e',
	   this.SecondBezierStartx,this.SecondBezierStarty, 
	   this.SecondControlAxisx,this.SecondControlAxisy,
	   this.SecondBezierEndx,this.SecondBezierEndy)
	   
	   
	   //draw top layer
	   ctx.beginPath();
	   ctx.strokeStyle = this.rgbToHex(242,42,36 );
	   ctx.strokeStyle = this.topLayerColor
	   let temArray = []	   
	   let tempItem ;
	   console.log("________lineWidth:"+ctx.lineWidth +" len:"+ this.topLayertemperatureData.length)
	   this.topLayertemperatureData.forEach((item,index)=>{		
		ctx.moveTo( item.startx , item.starty  );
		ctx.lineTo( item.endx, item.endy );		
		ctx.stroke()
		
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
	   
	   
	    ctx.beginPath(); 
		ctx.strokeStyle = this.rgbToHex(255,255,255 );
		
		
		// ctx.strokeStyle = "#f5b84e"
		ctx.lineWidth = 1
		
		//drawing outline
		let middleValue = (temArray.length/2).toFixed(0)
		let lineOffset = 6
		if ( temArray.length%2 !== 0 ){ //odd
			middleValue  = middleValue -1 
		}
		
		
		
	   for ( let i = 0 ; i < temArray.length - 1 ; i ++ ) {
		   
		 //   if ( i === 8 ){
			//    // ctx.beginPath(); 
			//    // ctx.strokeStyle = "blue"
		 //   }
		   	
			// else{
			// 	ctx.beginPath(); 
			// 	ctx.strokeStyle = this.rgbToHex(255,255,255 );
			// }
			if ( i === ( middleValue -1 ) ) {
				
				const gnt = ctx.createLinearGradient(temArray[i + 1].endx , temArray[i + 1].endy - ctx.lineWidth - lineOffset,
				temArray[i ].endx, temArray[i ].endy + lineOffset );
						gnt.addColorStop(0, this.topLayerColor);
						gnt.addColorStop(0.5, this.secondLayerColor);
						gnt.addColorStop(1, "#f5b84e");
						// ctx.fillStyle = gnt;	
						ctx.strokeStyle = gnt;
				
				ctx.moveTo( temArray[i + 1].endx , temArray[i + 1].endy - ctx.lineWidth - lineOffset);
				ctx.lineTo( temArray[i ].endx, temArray[i ].endy + lineOffset  );	
				ctx.stroke()   
			}	
		   else if ( i < middleValue ) {
			   ctx.moveTo( temArray[i].endx , temArray[i].endy + ctx.lineWidth + lineOffset);
			   ctx.lineTo( temArray[i + 1 ].endx, temArray[i+1].endy + lineOffset );		
			   ctx.stroke()
			   
		   }
		  	   
			else{
				ctx.moveTo( temArray[i].endx , temArray[i].endy - ctx.lineWidth - lineOffset);
				ctx.lineTo( temArray[i + 1 ].endx, temArray[i+1].endy - lineOffset );		
				ctx.stroke()
			}
		   
		   
	   }
	   
	   
		ctx.beginPath();
		// ctx.strokeStyle = this.rgbToHex(242,42,36  );
		ctx.strokeStyle = this.topLayerColor
		ctx.lineWidth = 1
		for ( let i = 0 ; i < temArray.length - 1 ; i ++ ) {

			if ( i === ( middleValue -1 ) ) {
			// ctx.moveTo( temArray[i + 1].endx , temArray[i + 1].endy - ctx.lineWidth - lineOffset);
			// ctx.lineTo( temArray[i ].endx, temArray[i ].endy + lineOffset  );	
			// ctx.stroke()   
			}	
			else if ( i < middleValue ) {

			for ( let ii = 0 ; ii < 20 ; ii ++ ){
			ctx.moveTo( temArray[i].endx , temArray[i].endy - ii*(ctx.lineWidth + 1 ) + 4 );
			ctx.lineTo( temArray[i + 1 ].endx -  ii*(ctx.lineWidth + 1 ), temArray[i+1].endy + 4 );		
			ctx.stroke()
			}

		}

		else{
			for ( let ii = 0 ; ii < 30 ; ii ++ ){
			
			ctx.moveTo( temArray[i].endx - ii*(ctx.lineWidth + 1) - 6, temArray[i].endy - ctx.lineWidth - lineOffset);
			ctx.lineTo( temArray[i + 1 ].endx - ii*(ctx.lineWidth + 1) - 6, temArray[i+1].endy - lineOffset );		
			ctx.stroke()
			}
		}
	  		   
	  		   
	  }
	   
	   
	 
	   
	  
		// this.testCode()
	  
	
	  
	 
	  
		
    },
    methods: {
		
		drawLayerBackGround(color, num, c = 0){
			debugger
		  var spacingX = parseFloat((realX / 120).toFixed(1));
		  ctx.beginPath();
		  // ctx.moveTo(endX, endY);
		  // ctx.quadraticCurveTo((num + c) / 0.2 * spacingX * 2 + endX, realY / 2, endX, originY);
		  
		  ctx.moveTo(this.startX, this.startY);
		  ctx.quadraticCurveTo(this.startX, this.startY,this.originX, this.originY);
		  ctx.strokeStyle = color;
		  ctx.fillStyle = color;
		  ctx.fill();
		  ctx.stroke();
		},
		
		drawBezierSixLayer(color, startx,starty,
								controlx,controly, endx, endy ){
		
		// const gnt = ctx.createLinearGradient(startx,starty,endx,endy);
		// gnt.addColorStop(0, '#ec443a');
		// gnt.addColorStop(0.5, '#ec443a');
		// gnt.addColorStop(1, '#ec443a');
		// ctx.fillStyle = gnt;
		// ctx.strokeStyle = gnt;
		ctx.beginPath();
		ctx.moveTo(startx , starty);
		  
		  
		ctx.quadraticCurveTo( controlx*2 - startx  , controly, endx , endy );
		ctx.strokeStyle = color;
		ctx.fillStyle = color;
		ctx.fill();
		ctx.stroke();
		
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
		
	  testCode(){
		  
		  // console.log(" ______ FSolidusAxisX:",this.FSolidusAxisX)
		  // console.log(" ______ FSolidusAxisY:",this.FSolidusAxisY)
		  // console.log(" ______ FourthControlAxisx:",this.FourthControlAxisx)
		  // console.log(" ______ FourthControlAxisy:",this.FourthControlAxisy)
		  
		// Define the points as {x, y}
		// let start = { x: 0, y: 108 };
		// let cp1 = { x: 80, y: 136 };
		// let cp2 = { x: 120, y: 200 };
		// let end = { x: 330, y: 240 };
		
		let yValue = this.FSolidusAxisY - this.spacingY*2 - 8
		let xValue = this.FSolidusAxisX
		// console.log("------ this.FSolidusAxisY:" + this.FSolidusAxisY + " this.spacingY" + this.spacingY)
		// console.log("------ this.FSolidusAxisX:" + this.FSolidusAxisX + " xValue" + xValue)
		// console.log("------ yValue:" + yValue)
		
		
		let start = { x: 0, y: 114 };
		let cp1 = { x: 80, y: 120 };
		let cp2 = { x: 120, y: yValue - 30 };
		let end = { x: 330, y: yValue };
		// ctx.strokeStyle = "#ee7940";
		ctx.strokeStyle = "#ffffff";
		ctx.lineWidth = 2
		// Cubic Bézier curve
		ctx.beginPath();
		ctx.moveTo(start.x, start.y);
		ctx.bezierCurveTo(cp1.x, cp1.y, cp2.x, cp2.y, end.x, end.y);
		ctx.stroke();
		
		
		
		//bottom edge
		let bottom = { x: 30, y: originY - 10 };
		let bottomcp1 = { x: 80, y: originY - 20 };
		let bottomcp2 = { x: 120, y: originY -80 };
		let bottomend = { x: 330, y: originY - 120 };
		// ctx.strokeStyle = "#ee7940";
		ctx.strokeStyle = "#ffffff";
		ctx.lineWidth = 2
		// Cubic Bézier curve
		ctx.beginPath();
		ctx.moveTo(bottom.x, bottom.y);
		ctx.bezierCurveTo(bottomcp1.x, bottomcp1.y, bottomcp2.x, bottomcp2.y, bottomend.x, bottomend.y);
		ctx.stroke();
		
		
		
		
		
		// // Start and end points
		// ctx.fillStyle = "blue";
		// ctx.beginPath();
		// ctx.arc(start.x, start.y, 5, 0, 2 * Math.PI); // Start point
		// ctx.arc(end.x, end.y, 5, 0, 2 * Math.PI); // End point
		// ctx.fill();
		
		// // Control points
		// ctx.fillStyle = "red";
		// ctx.beginPath();
		// ctx.arc(cp1.x, cp1.y, 5, 0, 2 * Math.PI); // Control point one
		// ctx.arc(cp2.x, cp2.y, 5, 0, 2 * Math.PI); // Control point two
		// ctx.fill();
		
		
		ctx.fillStyle = "blue";
		ctx.beginPath();
		ctx.arc(this.SecondBezierStartx , this.SecondBezierStarty  , 5, 0, 2 * Math.PI); // Start point
		// ctx.arc(this.FourthBezierStartx , this.FourthBezierStarty  , 5, 0, 2 * Math.PI); // Start point
		// ctx.arc(this.SixthBezierStartx , this.SixthBezierStarty  , 5, 0, 2 * Math.PI); // Start point
		
		ctx.arc(this.SecondBezierEndx , this.SecondBezierEndy  , 5, 0, 2 * Math.PI);
		// ctx.arc(this.FourthBezierStartx + 10 , this.FourthBezierStarty +  flexibleHeightOffset + 10 , 5, 0, 2 * Math.PI); // Start point
		// ctx.arc(this.FourthBezierEndx , this.FourthBezierEndy  , 5, 0, 2 * Math.PI);
		// ctx.arc(this.SixthBezierEndx , this.SixthBezierEndy  , 5, 0, 2 * Math.PI);
		
		// ctx.arc(end.x, end.y, 5, 0, 2 * Math.PI); // End point
		ctx.fill();
		
		ctx.fillStyle = "red";
		ctx.beginPath();
		
		ctx.arc(this.SecondControlAxisx, this.SecondControlAxisy, 5, 0, 2 * Math.PI); // Control point one
		// ctx.arc(this.FourthControlAxisx, this.FourthControlAxisy, 5, 0, 2 * Math.PI); // Control point one
		// ctx.arc(this.SixthControlAxisx, this.SixthControlAxisy, 5, 0, 2 * Math.PI); // Control point one
		// ctx.arc(cp2.x, cp2.y, 5, 0, 2 * Math.PI); // Control point two
		ctx.fill();
	  },
	  convertTemperatrueToColor(){
		  
		var temperature = []
		var colors = []
		var temperatureData = getTemperatureData()
		
		temperatureData.forEach((item,index)=>{ 
			temperature.push(item)
			
			if (item >= this.FSolidus ) {
				colors.push(this.rgbToHex(basicRGB_R,basicRGB_G,basicRGB_B))
				// 238 126 48
			}
			
			else if ( item >= 1400 && item < this.FSolidus ){
				colors.push(this.rgbToHex(242,42,36 )) // 142-->42
			}
			else if ( item >= 1300 && item < 1400 ){
				colors.push(this.rgbToHex(240,140 ,44))
			}
			else if ( item >= 1200 && item < 1300 ){
				colors.push(this.rgbToHex(242,141 ,23))
			}
			else if ( item >= 1100 && item < 1200 ){
				colors.push(this.rgbToHex(248,165 ,20))
			}
			else if ( item >= 1000 && item < 1100 ){
				colors.push(this.rgbToHex(253,182 ,7))
			}
			else if ( item >= 900 && item < 1000 ){
							
			// colors.push(this.rgbToHex(255,190 ,0))				
				colors.push(this.rgbToHex(254,180 ,4))	
			}
			else{
				 // colors.push(this.rgbToHex(basicRGB_R,basicRGB_G  ,basicRGB_G ))
				 colors.push(this.rgbToHex(254,197  ,3 ))
				 // colors.push(this.rgbToHex(254,187  ,3 ))
			}
			// console.log(item);
			// console.log(colors);
		});
		
		let arr = [],list = [];
		colors.forEach(c => {
			arr.push(c)
			if ( arr.length >= rowDataCounter ) {
				arr = []
				list.push(arr)
			}		
		})
		this.temperatureColorData = [...list]
		
		arr = []
		list = []
		temperature.forEach(c => {
			arr.push(c)
			if ( arr.length >= rowDataCounter ) {
				arr = []
				list.push(arr)
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
	
	
	drawBezier(color, num, c = 0){
	  // console.log(`originX=${originX}`)
	  // console.log(`endX=${endX}`)
	  var spacingX = parseFloat((realX / rowDataCounter ).toFixed(1));
	  // console.log(`spacingX=${spacingX}`)
	  ctx.beginPath();
	  ctx.moveTo(endX, endY + flexibleHeightOffset + 10	);
	  
	  // console.log("____endX:"+endX +" endY:"+endY)
	  ctx.quadraticCurveTo((num + c) / 0.2 * spacingX * 2 + endX, realY / 2 + flexibleHeightOffset, endX, originY - flexibleHeightOffset/2 + 30);
	  ctx.strokeStyle = color;
	  ctx.fillStyle = color;
	  ctx.fill();
	  ctx.stroke();
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
		
      setupCanvas() {
        var rect = canvas.getBoundingClientRect()
        canvas.width = rect.width * dpr
        canvas.height = rect.height * dpr
        var context = canvas.getContext('2d')
        context.scale(dpr, dpr)
        return context
      },
      //绘制坐标轴的方法
      drawAxis() {
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
		
        // 4.绘制X轴		
		var xlineOffset = 20 ;
		var startX = 0 ;
		var startY = originY ;
		
		
		
		ctx.beginPath();
		ctx.moveTo(startX, startY );
		startX = originX - xlineOffset
		ctx.lineTo(startX , startY);
		ctx.strokeStyle = "#000";
		ctx.stroke();
		ctx.closePath()
		
		startX = 0 
		startY = 0
		
        const gnt = ctx.createLinearGradient(startX,startY,originX,originY);
        gnt.addColorStop(0, '#E89B9B');
        gnt.addColorStop(0.5, '#A52A2A');
        gnt.addColorStop(1, '#A52A2A');
        ctx.fillStyle = gnt;		
		ctx.fillRect( startX , startY + flexibleHeightOffset  , originX - xlineOffset , originY - flexibleHeightOffset );
		
        //this.drawVerticalAxisTicks();
        for (var i = 0; i < this.temperatureColorData.length; i++) {
          this.drawFiostherms(this.temperatureColorData[i], i)
        }
        // this.drawFiostherms(this.temperatureColorData[0], 0)
        this.drawHorizontalAxisTicks();

      },
      //横坐标标尺及刻度
      drawHorizontalAxisTicks() {
        console.log('originX=' + originX)
        console.log('endX=' + endX)
        var deltaX,num = 0, spacing = parseFloat((realX / 23).toFixed(1));
        for (var i = 22;i > -1;i--){ //change 120 to 113 2003/09/07
          ctx.beginPath();
          ctx.strokeStyle = '#000';
          if(i % 2 === 0){
            text();
            num += 2;
            deltaX = 20
          } else {
            deltaX = 0
          }
          ctx.moveTo(originX - (i + 1) * spacing, originY - deltaX);
          ctx.lineTo(originX - (i + 1) * spacing, originY);
          ctx.stroke();
          function text(){
            ctx.font = "12px Calibri";
            ctx.fontStyle = "#000";
            ctx.fillText(num,originX - (i + 1) * spacing, originY + driftY);
          }
        }
      },
	  
	  
	
	  
	  drawFiostherms(data, index){
		var height = realY - flexibleHeightOffset 
		var spacingX = parseFloat((realX / 120).toFixed(1)), spacingY = parseFloat((height / 20).toFixed(4));
	
		ctx.lineWidth = spacingY;
		this.spacingY = spacingY 
		
		var secondAxisX = 0 
		var tmpSolidus = this.FSolidus 
		// this.tmpSolidus = this.FSolidus
		var temperatureData = this.temperatureData[index]
		
		var offsetx = 8 
		var offsetx1 = 0 
		ctx.beginPath();
		var drawable = 1 
		for (var i = 0; i < data.length - 1; i++) {
			
			drawable = 1 
			// ctx.beginPath();
			ctx.strokeStyle = data[i];
			
			var startx = endX + i * spacingX - offsetx1
			var starty = originY - spacingY * index - spacingX 
			var endx = endX + (i + 1) * spacingX  - offsetx1
			var endy = originY - spacingY * index - spacingX
			
			
			ctx.moveTo(endX + i * spacingX - offsetx1 , originY - spacingY * index - spacingX  );
			ctx.lineTo(endX + (i + 1) * spacingX  - offsetx1 , originY - spacingY * index - spacingX  );
			
			//remember the cooridinate system of Temperature solidification 
			if ( temperatureData[i] >= this.FSolidus  ) {
				
				
				this.topLayertemperatureData.push(
					{
					index: index,
					startx: startx, starty: starty, 
					endx: endx , endy: endy , 
					value: temperatureData[i]					
					}
				)
				
				
				
				if ( this.tmpSolidus == 0  ) {
					this.tmpSolidus = temperatureData[i]
					this.FSolidusAxisX = endX + (i + 1) * spacingX  - offsetx1
					this.FSolidusAxisY = originY - spacingY * index - spacingX 
				}
				else							
				{
					if ( temperatureData[i] < this.tmpSolidus  ) {
						
						this.tmpSolidus = temperatureData[i]
						this.FSolidusAxisX = endX + (i + 1) * spacingX  - offsetx1
						this.FSolidusAxisY = originY - spacingY * index - spacingX 
					}
				}			
			}
			else if ( temperatureData[i] >= this.SecondTemperature
				&& temperatureData[i]  < (this.SecondTemperature + this.temperatureOffset ) ) {			
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
			else if ( temperatureData[i] >= this.FourthTemperature 
				&& temperatureData[i]  < (this.FourthTemperature + this.temperatureOffset ) ) {			
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
			else if ( temperatureData[i] >=  this.FifthTemperature
				&& temperatureData[i]  < (this.FifthTemperature + this.temperatureOffset ) ) {			
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
			else if ( temperatureData[i] >=  this.SixthTemperature
				&& temperatureData[i]  < (this.SixthTemperature + this.temperatureOffset ) ) {			
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
				//< 900
				if ( i > 117 ) {
					drawable = 0 
				}
				
			}		
		
			if ( index !== 0 && drawable === 1 ){
				
				// ctx.stroke();
				
				
				// 	endX + (i + 1) * spacingX , originY - spacingY * index - spacingX  );
				// ctx.fillStyle =  data[i];
				// ctx.fill();
				// ctx.stroke();
			}
		}
	  },
      //纵坐标标尺及刻度
      drawVerticalAxisTicks() {
        var deltaY, num = 0, spacing = parseFloat((realY / 20).toFixed(4));
        console.log("_________ originX:",originX)
        console.log("_________ originY:",originY)
        console.log("_________ spacing:",spacing)
        console.log("_________ realY:",realY)

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
          ctx.stroke();

          function text() {
            ctx.font = "12px Calibri";
            ctx.fillText(num, originX + 5, originY - (i - 0.1) * spacing);
          }
        }
      }
    }
  }



</script>

<style scoped>
  #fiostherms{
    width: 80%;
    height: 400px;
  }
</style>
