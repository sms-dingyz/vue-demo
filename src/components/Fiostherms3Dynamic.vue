<template>
  <canvas id="fiosthermsDynamic"/>
  <!-- <div style="margin-top: 20px;font-weight: bold">Solidification[mm]:{{this.BottomfSoliLength}}</div> -->
</template>

<script>
  //import * as echarts from 'echarts/index';
  //import { getData } from './fiostherm'
  //import { getFiosthermData } from './fiostherm'
  //import {getTemperatureData} from './data3'
  //import {getTemperature3dData} from './data3d'
  import { getFiosthermRawData } from './fiostherm'
  
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
	//driftY = 0,
	
    realX = 0,
    realY = 0;
  var originxOffset = 30
  const slabLength = 34.5 
  const rowDataCounter = 120
  var columnCounter = 50 

  //temperarture bar
  const temperatureBarOffsetY  = 2
  const temperatureBarHeight = 32 //  
  const BottomOffset = 20 
  const graphicHeightOffset = 15

  const colorGrey = 'grey'
  const colorBlack = 'black'
  const colorWhite = 'white'

  const fontSize8 = '8px Calibri'
  const fontSize12 = '12px Calibri'
  const fontSize14 = '14px Calibri'

  //const SocketServer = 'ws://127.0.0.1:50000'

  const SocketServer = 'ws://10.183.7.101:50000'
  
  var firstLayerBezCurveTemperature = 1080
  var secondLayerBezCurveTemperature = 900
  //var thirdLayerBezCurveTemperature = 700

  const  isLocalData = 1
  var dpr = window.devicePixelRatio || 1;
  export default {
    data() {
      return {
		websock: null,//Established connection
		lockReconnect: false,//Whether the connection is really established
		
		
        timeout: 28*1000,//30 second heartbeat
        timeoutObj: null,//Heartbeat heartbeat countdown
        serverTimeoutObj: null,//heartbeat countdown
        timeoutnum: null,//Disconnect reconnect countdown
		reconnectInterval: 5000, // timeout 5 seconds
		
		
		
	
		topLayertemperatureData: [],
		topLayerColor: "#fad955",
		secondLayerColor: "#f5b84e",
		
		
		temperatureColorData: [],
		temperatureData: [],
		rowsElementsCoordinateInfo:[],
		
		SecondTemperature: 1400,
		FourthTemperature: 1100,
		FifthTemperature: 1000,
		SixthTemperature: 900,
		
		temperatureOffset: 100,
		
		FSolidusAxisX: 0,
		FSolidusAxisY: 0,
		spacingY: 0,
		tmpSolidus: 0,
		
		
		FirstBezierEndx: 0,
		FirstBezierStarty: 0,
		
		SecondBezierStartx: 0,
		SecondBezierStarty: 0,
		SecondBezierEndx: 0,
		SecondBezierEndy: 0,		
		SecondControlAxisx: 0,
		SecondControlAxisy: 0,
		
		ThirdBezierEndx: 0,
		
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
		
		
		temperatureBarIndicatorStartx: 0,
		fLiquidus: "1518.0",
		FSolidus: "1468.0",
		
		fSolidusPositionx: 0,
		fSolidusPositiony: 0,
		fLiquidusPositionx: 0, 
		fLiquidusPositiony: 0,
		//min
		fSolidusMinPositionx: 0,
		
		fSoliLength: 6.24,
		BottomfSoliLength:5300,
        fIsothermData: [],
        colorObj: [],
		reverseFlag: 0
      };
    },
  

	mounted() {
		canvas = document.getElementById("fiosthermsDynamic");
		ctx = this.setupCanvas();
		this.initWebSocket()
		this.initiateParameter()

		if ( isLocalData === 1) {

			this.handleRawData( getFiosthermRawData () ) 
		}
		else {
			this.getServerTemperatureData()
			this.connect()
		}

		// this.test()
		
	},

    methods: {


		
// Draw arc
drawArc([p0, p1, p2], r) {
  ctx.beginPath();
  ctx.moveTo(p0.x, p0.y);
  ctx.arcTo(p1.x, p1.y, p2.x, p2.y, r);
  ctx.lineTo(p2.x, p2.y);
  ctx.stroke();
},


loop(t) {
	let radius = 100 // match with init control value
	const p1 = { x: 100, y: 100 };
	const p2 = { x: 350, y: 150 };
	const p3 = { x: 100, y: 300 };

	// const p1 = { x: 100, y: 100 };
	// const p2 = { x: 100, y: 50 };
	// const p3 = { x: 200, y: 100 };
  const angle = (t / 1000) % (2 * Math.PI);
  const rr = Math.abs(Math.cos(angle) * radius);

//   ctx.clearRect(0, 0, canvas.width, canvas.height);

  this.drawArc([p1, p2, p3], rr);
  requestAnimationFrame(this.loop);
},

	test(){
	const p1 = { x: 100, y: 100 };
	const p2 = { x: 150, y: 50 };
	const p3 = { x: 200, y: 100 };
	let radius = 40 // match with init control value



	this.loop(0);
	},

	handleRawData( data ){
		this.fIsothermData = data
		this.topLayertemperatureData = []
		this.temperatureColorData = []
		this.temperatureData = []
		this.rowsElementsCoordinateInfo = []
		this.FirstBezierEndx = 0 
		this.SecondBezierEndx = 0 
		this.ThirdBezierEndx = 0 
		this.convertTemperatrueToColor()
		this.initiateParameter()
		this.drawBackground()
		// this.drawFiostherm( )

		this.getCoordinate()

		this.drawIndicatorOfTemperature(  )
		this.drawRowTemperatureColor()	
		this.drawHorizontalAxisTicks()
	},
		//page destruction, close long connection:
	destroyed() {
		//Close the long connection when the page is destroyed
		this.websocketclose();
	},
	
	initWebSocket(){ 
		//Initialize weosocket
		//const that = this;
		const wsuri = SocketServer; //ws address, this needs to be negotiated and defined with the backend
		// console.log(wsuri);
		//establish connection
		
		this.websock = new WebSocket(wsuri);
		//connection succeeded
		this.websock.onopen = this.websocketonopen;
		//connection error
		this.websock.onerror = this.websocketonerror;
		//Receive information
		this.websock.onmessage = this.websocketonmessage;
		//The connection is closed
		this.websock.onclose = this.websocketclose;
		
    },
	
	reconnect() {//Reconnect
      var that = this;
      if(that.lockReconnect) {
		console.log("WebSocket lockReconnect is true then return");
        return;
      }
      that.lockReconnect = true;
      //If it is not connected, it will always reconnect, set a delay to avoid too many requests
      that.timeoutnum && clearTimeout(that.timeoutnum);
      that.timeoutnum = setTimeout(function () {
      //New connection
      that.initWebSocket();
      that.lockReconnect = false;
      },5000);
    },
	
	websocketonmessage(){
		//console.log("-----WebSocket websocketonmessage");
	},
	
	websocketonopen() {//connection success event
		//console.log("-----WebSocket connection is successful");
        //Turn on the heartbeat
		this.getServerTemperatureData()
    },
    websocketonerror(e) {//connection failure event
        //Error
        console.log("------WebSocket connection error occurred:"+e.toString());
        //Reconnection
		this.reconnect();
    },
    websocketclose(e) {//connection closed event
        //shut down
       // console.log("------WebSocket closed");
		this.websock = null;
		console.log("connection closed (" + e.code + ")",+e.toString());
        //Reconnection
		this.reconnect();
    },
   
	//reset the heartbeat
	reset(){
      // console.log('Reset heartbeat');
      var that = this;
      //Clear time
      clearTimeout(that.timeoutObj);
      clearTimeout(that.serverTimeoutObj);
      //Restart the heartbeat
      that.startHeartBeat();
    },
	
	connect() {
	
	//console.log(" ------- start connect ");
    const self = this;
	
    self.websocket = new WebSocket(SocketServer);
	
	


    self.websocket.onclose = () => {
		//console.log(" ------- monitor connect onclose");
      self.lockReconnect = false
      setTimeout(() => {
		//console.log(" -------re connect ");
		self.websocket = null
        self.connect();

      }, self.reconnectInterval);
    };
	
  },


	
	
	//turn of the heartbeat
	startHeartBeat(){    
    },
	
	drawBackground(){	
        canvasWidth = Number(ctx.canvas.width) / dpr;
        canvasHeight = Number(ctx.canvas.height) / dpr;
		ctx.fillStyle = 'white';
		ctx.fillRect(0,0,canvasWidth,canvasHeight);
	},
	
	
    drawFourthSurfaceBezCurve(  ){
		

	},
	
	drawSurfaceBezCurve(color, x){
	//dyzdyz

		ctx.beginPath();
		ctx.lineWidth = 4
		ctx.fillStyle = 'white'
		// ctx.strokeStyle = 'red';
ctx.moveTo(0, canvasHeight/2 + 60 );
ctx.quadraticCurveTo(x, canvasHeight/2 -30 , 0, 42);
// ctx.quadraticCurveTo(canvasHeight*4 - 160, canvasHeight/2 -30 , 0, 42);

ctx.stroke();



	
	},
	
	drawThirdSurfaceBezCurve( solidusPosX ){
		//var spacingX = parseFloat((realX / rowDataCounter ).toFixed(1));
		ctx.beginPath();
		ctx.moveTo(10, 10 + driftY );  
		ctx.quadraticCurveTo( solidusPosX*2 - 40, canvasHeight/4  + 20, 10, canvasHeight/2 + 6 );
		ctx.strokeStyle = '#ef7941';
		ctx.fillStyle = '#ef7941';//
		ctx.fill();
		ctx.stroke();
	},
	
	drawFSolidusBezCurve( solidusPosX ){
		//var spacingX = parseFloat((realX / rowDataCounter ).toFixed(1));
		ctx.beginPath();
		ctx.moveTo(0,10 + driftY );  
		ctx.quadraticCurveTo( solidusPosX*2 - 180, canvasHeight/4  + 20, 0, canvasHeight/2  + 4 );
		ctx.strokeStyle = '#f5b84e';
		ctx.fillStyle = '#f5b84e';//
		ctx.fill();
		ctx.stroke();
	},
	drawFliquidusBezCurve( liquidusPosX  ){
		var curveEndx = liquidusPosX*2  - 180
		//patch
		if ( curveEndx < 150 ) {
			curveEndx =  curveEndx - 80
	}
    //var startpoint = this.topLayertemperatureData[0];
	//var endpoint = this.topLayertemperatureData[48*rowDataCounter + 1];
    ctx.beginPath();	  
	ctx.moveTo(0,10 + driftY );
	ctx.quadraticCurveTo( curveEndx, realY /4 + 20,-4, canvasHeight/2  + 8);
	//ctx.moveTo(0, 87);
    //ctx.quadraticCurveTo( 344, 223, 0, 279);
    ctx.strokeStyle = '#fad955';
    ctx.fillStyle = '#fad955';
    ctx.fill();
    ctx.stroke();
	},
	
	getServerTemperatureData() {
		return new Promise((resolve) =>{
			var messagetype = 'undefined'
			//var socket = new WebSocket( SocketServer );
		this.websock.onmessage = (event) => {
		if(typeof event.data === 'string') {
		//console.log(" ------- event.data:",event.data)
		var tmpData = JSON.parse(event.data)			
		messagetype = tmpData.messagetype
		}

		var data = JSON.parse(event.data )
		if ( 'fIsotherm' === messagetype ) {
			//console.log(" ------- event.data:",event.data)
			if ( this.reverseFlag === 0 ) {
				data = JSON.parse(event.data )
				this.fIsothermData = data

				this.topLayertemperatureData = []
				this.temperatureColorData = []
				this.temperatureData = []
				this.rowsElementsCoordinateInfo = []
				
				this.FirstBezierEndx = 0 
				this.SecondBezierEndx = 0 
				this.ThirdBezierEndx = 0 


				this.convertTemperatrueToColor()
				this.initiateParameter()
				this.drawBackground()
				this.drawFiostherm( )
				//this.drawFliquidusBezCurve()
			}
		}
		else if ( messagetype === "fTempCurve") {
			data = JSON.parse(event.data )
			this.fLiquidus = data.fLiquidus
			this.fSolidus = data.fSolidus
        }
		else if ( messagetype === "ModelResult") {
			data = JSON.parse(event.data )
			const ModelResultData = JSON.parse( data.ModelResultData )
			this.fSoliLength = ModelResultData.fSoliLength/1000
			this.fSoliLength = this.fSoliLength.toFixed(0)
			this.BottomfSoliLength = ModelResultData.fSoliLength.toFixed(0)
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

		
    drawRowTemperatureColor(  ) {
	
		ctx.beginPath();
			
		//var rows = this.temperatureColorData.length ;
		
		//var Colordata = this.temperatureColorData[0];
	
		//var elementWidth = parseFloat(( canvasWidth / rowDataCounter).toFixed(4))
		//elementWidth = elementWidth.toFixed(0); 		
		//var elementHeight = parseFloat(( canvasHeight  /rows ).toFixed(4));
       // elementHeight = elementHeight.toFixed(0);
		//elementHeight = 10 ;

		//console.log("________dyzdyz drawRowTemperatureColor elementWidth:"+ elementWidth)
		// ctx.beginPath();
		//var mIndex = 0 ;

		
		var currentLinex = 40
		var currentLiney = temperatureBarHeight
		var prevElementx = 0
		var prevElementy = 0


		var tempColor = 'yellow'
		var currentLine = 0 

		var layer2Temperature = 1113
		var coordinateInfo = []
		var currentElementx = 0
		var currentElementy = 0


		

		this.rowsElementsCoordinateInfo.forEach((row,index)=>{		
			var rowColors = this.temperatureColorData[ index ];
			var rowTemperatures = this.temperatureData[ index ];

			for (var i = 0; i < rowDataCounter  ; i++) {             
				var temperature = rowTemperatures[ i  ].toFixed(0) ;
				// ctx.strokeStyle = rowColors[ i  ]
				ctx.fillStyle = rowColors[  i  ]

				prevElementx = row.startx + row.elementWidth*i
				prevElementy = row.starty + row.elementHeight	

				if ( temperature >= layer2Temperature  ) {
					currentElementx = row.startx + row.elementWidth*i
					currentElementy = row.starty + row.elementHeight
					tempColor = rowColors[ i  ]
			
				}
				

				if ( i === ( rowDataCounter -1 )  ) {			
					coordinateInfo.push({
						index: index,
						startx: currentElementx,
						starty: currentElementy, 	
						color: tempColor			
						}
					)
				}

				
		
				if ( temperature === 0 ) {
					ctx.strokeStyle = "#ffffff"
					ctx.fillStyle = "#ffffff"
				}
				
				var temperatureList = this.temperatureData[index]				
				temperature = temperatureList[i]
				if ( temperature > firstLayerBezCurveTemperature ) {
					this.FirstBezierEndx = row.startx + row.elementWidth*i  +  row.elementWidth
				}
				if ( temperature > secondLayerBezCurveTemperature && temperature < firstLayerBezCurveTemperature ) {//900 700
					this.SecondBezierEndx = row.startx + row.elementWidth*i  +  row.elementWidth
									
				}
				if ( temperature < Number(this.fSolidus) && temperature >= 985 ) {
					this.ThirdBezierEndx = row.startx + row.elementWidth*i  +  row.elementWidth
				}
				
				
				ctx.fillRect( row.startx + row.elementWidth*i  , row.starty + row.elementHeight  , row.elementWidth + 1, row.elementHeight + 1 )
				ctx.stroke()
				//ctx.strokeRect(row.startx + row.elementWidth*i , row.starty + row.elementHeight  , row.elementWidth + 1, row.elementHeight  )
				//ctx.stroke()
				if ( i === rowDataCounter -1 ) {
					this.temperatureBarIndicatorStartx = row.startx + row.elementWidth*i +  row.elementWidth
				}
			}



			currentLine ++ 
			//draw			
			if ( index > 1 ) {
			// ctx.fillStyle = coordinateInfo[index].color
			ctx.fillStyle = '#ffffff'
			ctx.lineWidth = 6
			var info = coordinateInfo[index - 1 ]
			var info1 = coordinateInfo[ index ]
			
			ctx.moveTo( info.startx , info.starty  )
			ctx.lineTo( info1.startx, info1.starty  )
			ctx.stroke()
			}

			//mIndex ++ 			
		});
		

		
		// this.drawSurfaceBezCurve('#ef7941',this.ThirdBezierEndx)
		// this.drawSurfaceBezCurve('#f5b84e',this.SecondBezierEndx)
		// this.drawSurfaceBezCurve('#fad955',this.FirstBezierEndx)

	},


	getCoordinate(){     
		var i = 0 
		for ( i = 0; i < this.temperatureColorData.length ; i++) { // this.temperatureColorData.length -1
			this.getElementInformation(this.temperatureColorData[i], i )
		}				
		for ( i = 0; i < this.temperatureColorData.length  ; i++) { 
			this.getRowElementsInformation( i  )
		}
		
	},
	drawFiostherm( ) {	//
		var indicatorStartx = 0
        // canvasWidth = Number(ctx.canvas.width) / dpr;
        // canvasHeight = Number(ctx.canvas.height) / dpr;
        originX = canvasWidth - driftX - originxOffset;
        originY = canvasHeight - driftY;
        endY = 30;
        //var spacingX = parseFloat(( realX / rowDataCounter ).toFixed(1))
		
		
		// 2.计算X轴终点的位置
		endX = 0;
		// 3.x，y实际长度
		realX = originX;
		realY = originY;
		
		// 4.绘制X轴		
		//var xlineOffset = 20 ;
		//var startX = 0 ;
		//var startY = originY ;
		var i = 0 
		for ( i = 0; i < this.temperatureColorData.length ; i++) { // this.temperatureColorData.length -1
			this.getElementInformation(this.temperatureColorData[i], i )
		}
		
		
		for ( i = 0; i < this.temperatureColorData.length  ; i++) { 
			this.getRowElementsInformation( i  )
		}
		
		
		
	
		var result = 0 
		if ( result === 0 ){
		return
		}
		
		
		//draw top layer
		//ctx.beginPath();
		ctx.strokeStyle = this.rgbToHex(242,42,36 );
		ctx.strokeStyle = this.topLayerColor
		let temArray = []	   
		let tempItem ;
		
		var lineWidth = ctx.lineWidth.toFixed(2) 
		
		var mIndex = 0 
		var colorIndex = 0
		//var row = 0 
		
		var itemOffset = 0  // 24	 
		//var tempPosx = 0 ;
		//var spanY = 0 ;
		this.topLayertemperatureData.forEach((item,index)=>{	
		
			if ( index > 0 &&  ( index % rowDataCounter ) === 0 ) {
				//row ++ 			
				colorIndex ++ 
				mIndex = 0 
			}				
			ctx.beginPath();
			ctx.lineWidth = lineWidth 
			
			var color = this.temperatureColorData[colorIndex]
			ctx.strokeStyle = color[mIndex]
			ctx.fillStyle = color[mIndex]
			if ( item.value === 0 ) {
				ctx.strokeStyle = "#ffffff"
				ctx.fillStyle = "#ffffff"
				
			}
			
			
			if ( item.value  > 0 ) { // 1250
				ctx.strokeStyle = color[mIndex]
				ctx.fillStyle = color[mIndex]
				
				ctx.moveTo( item.startx , item.starty - itemOffset );
				//ctx.lineTo( item.endx, item.endy  - itemOffset );	
				ctx.fillRect(item.startx , item.starty - itemOffset,ctx.lineWidth,8 )

             
				ctx.closePath()
				ctx.stroke()
				
				
				//fLiquidus: "1518.0",FSolidus: "1468.0",
				if ( Number( item.value ) >= Number( this.fLiquidus)  ) {
					if ( item.endx >= this.fLiquidusPositionx ) {
						this.fLiquidusPositionx = item.endx 	
						//ctx.save()	
						//////	this.drawThirdSurfaceBezCurve( this.fSolidusPositionx )
						/////	this.drawFSolidusBezCurve( this.fSolidusPositionx )
						/////	this.drawFliquidusBezCurve( item.endx )
						//ctx.restore()
					}	
				}
				// else if ( Number( item.value)< Number( this.fSolidus) && Number( item.value ) >= Number( this.fLiquidus)  ) {
				// 	//spacingX = parseFloat((realX / rowDataCounter).toFixed(1));
				// 	if ( this.fSolidusMinPositionx === 0 ) {
				// 		this.fSolidusMinPositionx = item.endx 
				// 	}
					
				// 	if ( item.endx >= this.fSolidusPositionx   ) {
				// 	//ctx.save()	
				// 	this.fSolidusPositionx = item.endx 
					
				// 	///////this.drawThirdSurfaceBezCurve( this.fSolidusPositionx )

				// 	////////this.drawFSolidusBezCurve( this.fSolidusPositionx )
		
				// 	//this.fSolidusPositionx  = item.startx 	
					
				// 	///////this.drawFliquidusBezCurve( this.fLiquidusPositionx )
					
					
				// 	//ctx.restore()
				// 	}
				// 	else if ( this.fSolidusPositionx != 0 ) {					
				// 		if ( item.endx < this.fSolidusPositionx ) {
				// 			//this.fSolidusPositionx = item.endx 
				// 		}
				// 	}
				// }
				else {
					//////this.drawThirdSurfaceBezCurve( this.fSolidusPositionx )
					/////this.drawFSolidusBezCurve( this.fSolidusPositionx )
					//////this.drawFliquidusBezCurve( this.fLiquidusPositionx )
				}
			}
			
			//record first element
	
			if ( mIndex ===  0 ) {
				tempItem = item
			}
			else {	
				
				if ( color[mIndex] !== "#ffd42d"  
					&& color[mIndex -1] === "#ffd42d" 
					&& item.index === tempItem.index ){
					temArray.push( tempItem )
					// if ( row >= 0 ){
					// 	console.log(" aaa ------- tempItem："+tempItem)
					// 	temArray.push( tempItem )
					// }
				}
				// if ( item.index !== tempItem.index ) {				
				// 	temArray.push( tempItem )
				// 	if ( index === this.topLayertemperatureData.length -1 ) {
				// 		temArray.push( item )
				// 	}
				// }
				tempItem = item
			}
			mIndex ++ 
			
			
			//mark last posx
			if ( mIndex === 120 ){
				indicatorStartx = item.endx
			}
				
		});
		
		
		this.drawIndicatorOfTemperature( indicatorStartx )
		this.drawHorizontalAxisTicks()
	
		//draw indicator of temperature
		// this.drawIndicatorOfTemperature()
		// this.drawOutline(temArray)
		

    },
	drawOutline(temArray){
		ctx.beginPath();
		ctx.strokeStyle = this.rgbToHex(255,255,255 );
		ctx.strokeStyle = this.rgbToHex(255,212,45 ); //218
		ctx.lineWidth = 1
	
		//drawing outline
		let middleValue = (temArray.length/2).toFixed(0)
		let lineOffset = 2
		if ( temArray.length%2 !== 0 ){ //odd
			middleValue  = middleValue -1 
		}	
		for ( let i = 0 ; i < temArray.length - 1 ; i ++ ) {
			if ( i === ( middleValue -1 ) ) {
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
			// if ( temArray[i+1].endy > temArray[i].endy ) {
			for ( let ii = 0 ; ii < 8 ; ii++ ){
				ctx.moveTo( temArray[i].endx , temArray[i].endy + ctx.lineWidth   + 1*ii - 4);
				ctx.lineTo( temArray[i + 1 ].endx, temArray[i+1].endy    + 1*ii - 4);
			}
			// }	   
			ctx.stroke()
			}
		}
	
	},

	
	drawIndicatorOfTemperature1(  ){
		var yoffset = 0 		
		var indicatorStartx = 0 ;
		var indicatorStarty = 0;
		
		ctx.beginPath()
		ctx.lineWidth = 1
		ctx.fillStyle = "#ffffff";
		ctx.strokeStyle ="grey"
		ctx.strokeRect( indicatorStartx  , indicatorStarty  , originY - 32,36);
		ctx.strokeRect( indicatorStartx  , indicatorStarty  , originY - 32,36);

		
		//color bar
		//const gnt = ctx.createLinearGradient( indicatorStartx + 8, endY + 10 - yoffset , indicatorStartx + 62 , endY + 10 + originY  );
		const gnt = ctx.createLinearGradient( indicatorStartx + 2 , indicatorStarty ,  endY + 10 + originY,indicatorStartx + 62  );
		gnt.addColorStop(0, '#FFC92D');
		gnt.addColorStop(0.3, '#fe902d');
		gnt.addColorStop(0.6, '#ff0000');
		gnt.addColorStop(1, '#707070');
		
		ctx.fillStyle = gnt;
		ctx.strokeStyle = gnt
		ctx.fillRect(indicatorStartx + 2, indicatorStarty + 2, originY - 36 - yoffset,12  );
		
		
		ctx.fillStyle = "grey"
		ctx.strokeStyle = "grey"
		ctx.font = "12px Calibri";
	
		
		//ctx.fillText("T/°C",indicatorStartx + 10 , 30);
		ctx.font = "12px Calibri";
		ctx.fillText(1500,indicatorStartx + 2 , 30 );
		ctx.fillText(1100, (originY - 32)/2 , 30 );		
		ctx.fillText(600,originY - 52, 30 );
		ctx.font = "15px Calibri";
		ctx.fillText("T/°C",originY - 30, 30);
		
	},



	drawIndicatorOfTemperature( ){
	var indicatorStartx = 0 ;
	var indicatorStarty = 0 ;
	var rectWidth = canvasWidth / 5 
	var rectHeight = temperatureBarHeight 

	ctx.beginPath()
	ctx.lineWidth = 1
	ctx.fillStyle = colorWhite
	ctx.strokeStyle = colorGrey
	ctx.strokeRect( indicatorStartx  , indicatorStarty  , rectWidth + 12 , rectHeight );
	
		
	//color bar
	const gnt = ctx.createLinearGradient( indicatorStartx + 2 , indicatorStarty ,  rectWidth, rectHeight/2  );
	gnt.addColorStop(0, '#FFC92D');
	gnt.addColorStop(0.3, '#fe902d');
	gnt.addColorStop(0.6, '#ff0000');
	// gnt.addColorStop(1, '#ff3030');
	// gnt.addColorStop(1, '#707070');
		
	ctx.fillStyle = gnt;
	ctx.strokeStyle = gnt
	ctx.fillRect(indicatorStartx + 2, indicatorStarty + 2, rectWidth + 8 , rectHeight/2  );
		
	ctx.fillStyle = colorGrey
	ctx.strokeStyle = colorGrey
	ctx.font = fontSize12

	var textY = temperatureBarHeight - 4
	ctx.fillText( "1500", indicatorStartx + 2 , textY );
	ctx.fillText( "1100", rectWidth/2 , textY );	
	ctx.fillText( "600",  rectWidth - 10, textY );	
	ctx.fillText( "T/°C", rectWidth + 15 , textY);
	// ctx.fillText( "1100", (originY - 32)/2 , textY );		
	// ctx.fillText( "600",originY - 52, textY );
	// ctx.fillText( "T/°C", originY - textY, textY);

	},


	drawText( text , scaleStartx , index ,spacingX , scaleStarty) {
		var textDeltax = 22;
		ctx.font = "12px Calibri";
		if( text > 9) {
			ctx.fillText( text ,scaleStartx + ( index - 0.6) * spacingX, scaleStarty + textDeltax );
		}
		else if( text === 0) {
			ctx.fillText( text ,scaleStartx + ( index - 0) * spacingX, scaleStarty+ textDeltax );
		}
		else {
			ctx.fillText( text ,scaleStartx + ( index - 0.2) * spacingX, scaleStarty+ textDeltax );
		}
	},

	drawAxistext( num , index , span ){
		var starty = canvasHeight/2 + 78 ;

		starty = canvasHeight
		ctx.font = fontSize12
		if( num > 9) {
			ctx.fillText( num ,endX + ( index - 0.8) * span - 2, starty );
		}
		else if( num === 0 ) {
			ctx.fillText( num ,endX + ( index - 0 ) * span, starty);
		}
		else {
			ctx.fillText( num ,endX + ( index - 0.2) * span - 2, starty);
		}
	},
	drawHorizontalAxisTicks(){
		//var slabLength = 35 
		var num = 0, spacingX = parseFloat((canvasWidth / rowDataCounter ).toFixed(1));
		var perScaleLength = canvasWidth/slabLength
		var span = (perScaleLength/10).toFixed(2)
		spacingX = span 

		var deltaY = 8 
		var startx = 0 
		var starty = canvasHeight - BottomOffset  	
		starty = canvasHeight/2 + 60 ;


		// starty = canvasHeight - BottomOffset;
		starty = canvasHeight - BottomOffset

		for ( var i = 0;i < slabLength*10 ; i++ ){ 
			ctx.lineWidth = 1
			ctx.beginPath();
			if(i % 20 === 0){
				ctx.strokeStyle = colorBlack
				ctx.fillStyle = colorBlack
				//text();
				this.drawAxistext( num , i , spacingX );
				num += 2;				
				ctx.moveTo( startx + i * spacingX, starty );
				ctx.lineTo( startx + i * spacingX, starty + deltaY );
			}else {
				if ( i % 2 === 0 )
				{
					ctx.strokeStyle = colorGrey
					ctx.fillStyle = colorGrey
					ctx.moveTo( startx + i * spacingX, starty );
					ctx.lineTo( startx + i * spacingX, starty + 4 );
				}
			}		
			ctx.stroke();
			
		}
	},  


	getRowElementsInformation( rowNo ){

		// var elementWidth = parseFloat(( canvasWidth / rowDataCounter).toFixed(1))
		// elementWidth = elementWidth.toFixed(1); 	
		// elementWidth = elementWidth - 0.1

		//计算每个元素的刻度长度：
		//calcalute width of element
		var perScaleLength = canvasWidth / slabLength 
		var elementLength = slabLength / rowDataCounter 
		var span  = elementLength*perScaleLength
		var elementWidth = span.toFixed(1) - 0.01
		
		var Colordata = this.temperatureColorData;	
		var rows = Colordata.length

		//calcalute height of element
		var elementHeight = parseFloat(( canvasHeight  /rows ).toFixed(1));
        elementHeight = elementHeight.toFixed(1);
		elementHeight = 10 ;			
		var startx = 0
		var starty = 0
		starty = rowNo*elementHeight + 30 ;

		
		//new calculation
		var deltaY = graphicHeightOffset
		starty = temperatureBarHeight
		var graphicHeight = canvasHeight - ( BottomOffset + temperatureBarHeight  + deltaY )
		elementHeight = parseFloat(( graphicHeight/rows ).toFixed(1));
		starty = starty + rowNo*elementHeight




		this.rowsElementsCoordinateInfo.push(
		{
			index: rowNo,
			startx: startx,
			starty: starty, 
			elementWidth: elementWidth ,
			elementHeight: elementHeight  				
			}
		)

				
	},

	getElementInformation( Colordata ,index ){
	
		//var startyOffset = 60 // 44  115 
		var xlineOffset = 20
		var height = realY - 126 //70
		

		height = height /2 - 30
		//height += 30 
	
		//console.log(" ______ dyz getElementInformation index:" + index + " Colordata.length:" + Colordata.length)
		var spacingX = parseFloat(( (realX - xlineOffset) / rowDataCounter).toFixed(1))
		var spacingY = parseFloat(( height  / columnCounter).toFixed(4));
		//spacingY = parseFloat(( height  / Colordata.length ).toFixed(4));
		spacingY = spacingY + 2
	
		
		var offset_width = 6 ; //change 1 to 6 2024.11.22
		ctx.lineWidth = parseInt(spacingX) + offset_width;
		
		spacingX = spacingX.toFixed(0); 
		spacingY = spacingY.toFixed(0); 
		this.spacingY = spacingY 

		

		var perScaleLength = canvasWidth / slabLength 
		//计算每个元素的刻度长度：如果需要绘制120个元素，且总长度为35米，则每个元素在实际长度上占的米数为：
		var elementLength = slabLength / rowDataCounter 
		var span  = elementLength*perScaleLength
		spacingX = span - 0.3
	
		// console.log(" ______ dyz ctx.lineWidth:" + ctx.lineWidth)
		
		
		var secondAxisX = 0 
		//var tmpSolidus = this.FSolidus 		
		var temperatureData = this.temperatureData[index]
		
		
		var offsetx = 8 		
		// ctx.beginPath();
	
	
	
		var startx = 0
		var starty = 0
		var endx = 0
		var endy = 0
		
		for (var i = 0; i < Colordata.length  ; i++) {
	
			// ctx.beginPath();
			// ctx.lineWidth = spacingY;
					
			ctx.strokeStyle = Colordata[i];
			
		
			startx = i*spacingX 		
			startx = Number(startx.toFixed(1))						
			starty = originY -  index*spacingY 
			

			
			starty = originY - height*2 -  index*spacingY 				
			//starty = originY -  index*spacingY - elementStartyOffset 
			
		
		
			endx += parseFloat(spacingX)
			endy = starty	
			var tmpx = 0 , tmpy = 0
			var x = 0 
			if ( Number( temperatureData[i] ) >= Number( this.fLiquidus)  ) {
				
				this.topLayertemperatureData.push(
					{
					index: index,
					startx: startx, starty: starty, 
					endx: endx , endy: endy , 
					value:  Number( temperatureData[i] )					
					}
				)
				
		
				
				//remember last position
				if ( endx > this.fLiquidusPositionx ){
					//this.fLiquidusPositionx = endx
					//this.fLiquidusPositiony = endy
					//console.log(" ______ dyz last fLiquidusPositionx:" + endx )
				}
					
				
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
		
				
			else if (  Number( temperatureData[i] ) >= Number( this.FSolidus) && Number( temperatureData[i] ) < Number( this.fLiquidus) ) {
				//
				this.topLayertemperatureData.push(
						{
						index: index,
						startx: startx, starty: starty, 
						endx: endx , endy: endy , 
						value:  Number( temperatureData[i] )					
						}
					)
					//console.log(" aaa**************i:" + i  + " last fSolidusPositiony:" + endx )
					//remember last position
					if ( endx >= this.fSolidusPositionx ){
						this.fSolidusPositionx = endx
						
						//this.fSolidusPositiony = endy
						//console.log(" ______ dyz index:" + index + " i:" + i + " last fSolidusPositiony:" + endx   )
						//console.log(" ______ dyz index:" + index + " i:" + i + " temperature:" + Number( temperatureData[i]) + " temperature:" + " fLiquidus:" + this.fLiquidus + " FSolidus:" + this.FSolidus )
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
					
				tmpx = endX + (i + 1) * spacingX  -offsetx
				tmpy = originY - spacingY * index - spacingX
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
			
				x = endX + (i + 1) * spacingX  - offsetx 
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
				tmpx = endX + (i + 1) * spacingX - offsetx
				tmpy = originY - spacingY * index - spacingX
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
			
			x = endX + (i + 1) * spacingX - offsetx 
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
				
			tmpx = endX + (i + 1) * spacingX - offsetx
			tmpy = originY - spacingY * index - spacingX
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
		
			x = endX + (i + 1) * spacingX - offsetx 
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
				
				
			tmpx = endX + (i + 1) * spacingX - offsetx
			tmpy = originY - spacingY * index - spacingX
			
		
		
			x = endX + (i + 1) * spacingX - offsetx 
			
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

	drawSecondBackgroud(color, num ){
		var spacingX = parseFloat((realX / rowDataCounter).toFixed(1));
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



	getSourceTemperatureData(  ){
		var sourceData = [] 
		//for( var i = 1 ; i <= 50 ; i++ ) {			
			//var mIndex = "fIsotherm_" + i 
			//var mData = JSON.parse(this.fIsothermData[mIndex])
			//var baseIndex = index*rowDataCounter + 1 
							
			//for(var j = 0 ; j < rowDataCounter  ; j++ ) {				
			//sourceData.push(mData[ baseIndex +  j ])
			//}		
		//}	

		
		var mIndex = "fIsotherm_50"
		var mData = JSON.parse(this.fIsothermData[mIndex])	
		
		
		mData.forEach((elem,index)=>{ 
		if ( index > 0 ) {
			sourceData.push(elem)
		}
		})
		
		//for( var i = 0 ; i <= mData.length ; i++ ) {	
			//sourceData.push(mData[ i +  1 ])
		//}		
		console.log("--- len:" + sourceData.length + " mData.length:" + mData.length +  " sourceData:" + sourceData)
		return sourceData	
	},
	convertTemperatrueToColor(){
		var temperature = []
		var colors = []
		// var temperatureSourceData = this.getSourceTemperatureData(  )

		var mIndex = "fIsotherm_50"
		var temperatureSourceData = JSON.parse(this.fIsothermData[mIndex])		
		temperatureSourceData.forEach((element,index)=>{ 
			if ( index > 0 ) {
			
				var item = 0 
				if(typeof element === 'string') {
					item = Number(element)
				}
				else 
				item = element						
				// item = item.toFixed(2)
				temperature.push(item)						
				item = Number(item)											
				var rgb = this.temperature2Color(index,item,600,1600,"half")	//550 1800					
				colors.push(this.rgbToHex(rgb.r,rgb.g,rgb.b))
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
		
		arr = []
		list = []
		
		//var counter = 0 ;
		temperature.forEach(t => {
			arr.push(t)
			//counter ++ 
			if ( arr.length >= rowDataCounter ) {	
				list.push(arr)
				arr = []
				//counter = 0 
			}		
		})
		this.temperatureData = [...list]
		//console.log("bbb---temperature:" + this.temperatureData)
	},	
	decToHex(dec) {
		return dec.toString(16)
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
		//screen width and height
		canvasWidth = Number(ctx.canvas.width) / dpr;
		canvasHeight = Number(ctx.canvas.height) / dpr;
			
		// The position of the origin of the coordinate system
		originX = canvasWidth - driftX;
		originY = canvasHeight - driftY;
		
		//  X-axis ends
		endX = 0;
		
		//real with and height
		realX = originX;
		realY = originY;
		
		
		
		originX = driftX;
		originY = driftY
		realX =  canvasWidth;
		realY = canvasHeight;
		
		
		this.fSolidusPositionx = 0 
		this.fSolidusPositiony = 0 
		this.fLiquidusPositionx = 0 
		this.fLiquidusPositiony = 0 
		this.fSolidusMinPositionx = 0
		this.fIsothermData = []

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
		
		var temperature = kelvin / 10.0;
		var red, green, blue;
		
		if (temperature <= 66.0) {
			red = 0;
		} 
		else {
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
		} 
		else {
			green = temperature - 60.0;
			green = 288.1221695283 * Math.pow(green, -0.0755148492);
			if (green < 0) green = 0;
			if (green > 255) green = 255;
		}
		
		/* Calculate blue */
		
		if (temperature >= 66.0) {
			blue = 255;
		} 
		else {
			if (temperature <= 19.0) {
				blue = 0; 
			} 
			else {
				blue = temperature - 10;
				blue = 138.5177312231 * Math.log(blue) - 305.0447927307;

				if (blue < 0) blue = 0;
				if (blue > 255) blue = 255;
			}
		}
		
		return {red: Math.round(red), blue: Math.round(blue), green: Math.round(green)};
		// return this.rgbToHex(red,green,blue)	
		},
		
	
		temperature2Color (index,t, min, max, mode) {
			var rValue = 255
			var gValue = 255;
			var bValue = 255;
			if (!Number.isFinite(t) || !Number.isFinite(min) || !Number.isFinite(max)) {
				console.log("--temperature beyond range t:" + t + " index:"+index)
			}
			
		
			if ( t === 0  ) {
				//set white as default color
				return {
					r: this.checkRange(Math.trunc(255)),
					g: this.checkRange(Math.trunc(255)),
					b: this.checkRange(Math.trunc(255))
				}
			}

			//console.log("index:" +index +" -- t:" + t + " this.fSolidus:"+this.fSolidus +" this.fLiquidus:"+this.fLiquidus )
			// if ( t > this.fSolidus ) {
			// 	rValue = 128 + 4 * (1 - 0.838) * 127.999;
			// 	gValue = 0;
			// 	bValue = 0;

			// 	rValue = 255
			// 	gValue = 212
			// 	bValue = 45;

				
			// 	//set white as default color
			// 	return {
			// 		r: this.checkRange(Math.trunc(rValue)),
			// 		g: this.checkRange(Math.trunc(gValue)),
			// 		b: this.checkRange(Math.trunc(bValue))
			// 	}
			// }			
			// if ( t < Number(this.fSolidus) && t >= 985 ) {
			// 	return {
			// 		r: this.checkRange(Math.trunc(255)),
			// 		g: this.checkRange(Math.trunc(201)),
			// 		b: this.checkRange(Math.trunc(14))
			// 	}
			// }	
			

			if (t < min) {
				t = min;
			} else if (t > max) {
				t = max;
			}
			//console.log("-- t:" + t + " index:"+index)
			const nT = (t - min) / (max - min);
			
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
					}
					else if (nT > regions[1] && nT <= regions[2]) {
						rValue = 0;
						gValue = 255;
						bValue = 768 - 6 * nT * 255.999;

				
						// console.log(" -------------nT:"+ nT + " t:" + t + " regions[0]:"+regions[1] + " regions[1]:"+regions[2])
						// if ( t > 980 ) {
						// 	rValue = 255
						// gValue = 200
						// bValue = 40;
						// }
					

					
					} else if (nT > regions[2] && nT <= regions[3]) {
						rValue = 768 - 6 * (1 - nT) * 255.999;
						gValue = 255;
						bValue = 0;

						// if ( t > 1120 ){
						// rValue = 255
						// gValue =  220;
						// bValue = 45
						// }
						
					
					

					} else if (nT > regions[3] && nT <= regions[4]) {
						rValue = 255;
						gValue = 1280 - 6 * nT * 255.999;
						bValue = 0;

						// console.log(" -------------nT:"+ nT + " t:" + t + " regions[0]:"+regions[1] + " regions[1]:"+regions[2])
						// if ( t > 1250 && t < 1400 ){
						// 	rValue = 255
						// gValue =  220;
						// bValue = 45
						// }

					} else {
						rValue = 255;
						gValue = 0;
						bValue = 128 - 6 * (1 - nT) * 127.999;
					
					// rValue = 255
					// gValue = 508 - 2 * (1 - nT) * 255.999 - 100;
					// bValue = 10;	
					
					
				}
				break;
			}
			
			case 'half': {
	
				const regions = [1 / 4, (1 / 4) * 2, (1 / 4) * 3];
				if (nT <= regions[0]) {
					rValue = 0;
					gValue = 128 + 4 * nT * 127.999;
					bValue = 0;
					
					rValue = 255;
					gValue = 508 - 2 * (1 - nT) * 255.999;
					bValue = 30;

					

				} else if (nT > regions[0] && nT <= regions[1]) {
					// console.log(" -------------nT:"+ nT + " t:" + t + " regions[0]:"+regions[0] + " regions[1]:"+regions[1])
					//500---900
					var factor = nT 
					rValue = 768 - 4 * (1 - nT) * 255.999;
					gValue = 255;
					bValue = 0;
					
					rValue = 255;
					gValue = 508 - 2 * (1 - nT) * 255.999 ;
					gValue = 508 - 2 * (1 - nT) * 255.999 - 70;
					bValue = 0;
					if ( factor > 0.25 ) {
						gValue = 508 - 2 * 0.74 * 255.999 ;
					}
				

					if ( t > 980 && t < 1200 ){					
						//layer3
						// gValue = 508 - 2 * (1 - nT) * 255.999 
						// gValue = 180	
						// bValue = 40;

						gValue = 508 - 2 * 0.678 * 255.999 ;


					}
				
				} else if (nT > regions[1] && nT <= regions[2]) {
					
					// 900 < t < 1200
					rValue = 255;
					gValue = 768 - 4 * nT * 255.999;
					bValue = 0;
					// 255 183  45
					bValue = 45;
					bValue = 25;
					gValue = 183
					// console.log(" -------------nT:"+ nT + " t:" + t + " regions[0]:"+regions[0] + " regions[1]:"+regions[1])			
					//layer2
					if ( t < 1350 && t > 1118 ){
						

						rValue = 255
						gValue = 408 - 2 * (1 - nT) * 255.999 
						// gValue = 200	
						bValue = 10;
						
						// console.log(" -------------nT:"+ nT)
						var factor1 = nT
						if ( factor > 0.58 ) {
							factor1 = 0.56
						}
						else {
							factor1 = 0.6
						}
						rValue = 255;
						gValue = 408 - 2 * (1 - factor1) * 255.999 
						bValue = 10;
					}
					else if ( t < 1118 ){
						gValue = 508 - 2 * 0.678 * 255.999 ;
					
						
					}

					
					
					
				} 
				else {				 
					//liquid  1500
					// ///console.log(" -------------nT:"+ nT + " t:" + t + " regions[0]:"+regions[0] + " regions[1]:"+regions[1])
					rValue = 128 + 4 * (1 - nT) * 127.999;
					gValue = 0;
					bValue = 0;
					
					//liquid the highest temperature
					rValue = 255
					gValue = 508 - 2 * (1 - nT) * 255.999 - 100;
					bValue = 10;


					rValue = 255
					gValue = 220
					bValue = 10;


					
				}
				break;
			}
			
			default: {
				const regions = [1 / 4, (1 / 4) * 2, (1 / 4) * 3];
				// const regions = [1 / 5, (1 / 5) * 2, (1 / 5) * 3,4/5];
				if (nT <= regions[0]) {					  
				rValue = 0;
				gValue = 4 * nT * 255.999;
				bValue = 255;
					
					// adjust
					// rValue = 255;
					// gValue = 2.5 * nT * 255.999;
					// bValue = 30;
				} else if ( nT > regions[0] && nT <= regions[1] ) {
					rValue = 0;
					gValue = 255;
					bValue = 512 - 4 * nT * 255.999;
					//adjust
					// rValue = 255;
					// gValue = 2.5 * nT * 255.999;
					// bValue = 30;

				} else if (nT > regions[1] && nT <= regions[2]) {
					rValue = 512 - 4 * (1 - nT) * 255.999;
					gValue = 255;
					bValue = 0;  
					// rValue = 255
					// gValue = 512 - 3 * (1 - nT) * 255.999;
					// bValue = 0;  
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
		},
		
		
		TemperatuerToColortempToColor (temperature, mintemperature, maxtemperature)
		{
			var Red = this.rgbToHex(255, 0, 0)
			var Yellow = this.rgbToHex(255, 255, 0)
			var Blue = this.rgbToHex(0, 255, 255)
			
			var ratio = temperature / (maxtemperature - mintemperature)
			var result1 = this.Compute(Blue, Yellow, ratio)
			var result2 = this.Compute(Yellow, Red, ratio)
			var result =  this.Compute(result1, result2, ratio)

			// return new SolidColorBrush(result);
			return result
		},
		Compute( fromColor,  toColor,  ratio)
		{
			var rValue = (fromColor.R * (1.0 - ratio) + toColor.R * ratio);
			var gValue = (fromColor.G * (1.0 - ratio) + toColor.G * ratio);
			var bValue = (fromColor.B * (1.0 - ratio) + toColor.B * ratio);
			
			return {
			r: this.checkRange(Math.trunc(rValue)),
			g: this.checkRange(Math.trunc(gValue)),
			b: this.checkRange(Math.trunc(bValue))
			}
			// return this.rgbToHex(r, g, b);
		},
			
		tempToColor_3 (index,t) {
			
			var Temperature = t
			var Red = 255
			var Green = 255
			var Blue = 255
			
			if ( Temperature < 1000)
			Temperature = 1000
			
			Temperature = Temperature /10
			
			//console.log("----dyzdyzdyz t:"+Temperature)
			if ( Temperature <= 66 ){
				Red = 255
			}
			else {
				Red = Temperature - 60 
				Red = 329.698727446*Math.pow(Red,-0.1332047592)
				if ( Red < 0 ) {
					Red = 0 
				}
				if (Red > 255 ) {
					Red = 255
				}
			}
			
			
			if ( Temperature <= 66 ){
				Green = Temperature
				Green = 99.4708025861*Math.log(Green) - 161.1195681661
				if ( Green < 0 ) {
					Green = 0 
				}
				if (Green > 255 ) {
					Green = 255
				}
			}
			else {
				Green = Temperature - 60
				Green = 288.1221695283*Math.pow(Green,-0.0755148492)
				if ( Green < 0 ) {
					Green = 0 
				}
				if (Green > 255 ) {
					Green = 255
				}
			}
			
			
			if ( Temperature >= 66 ){
				Blue = 255
			}
			else if ( Temperature <= 19  ){
				Blue = 0
			}
			else {
				Blue = Temperature - 10 
				Blue = 138.5177312231*Math.log(Blue) - 305.0447927307
				if ( Blue < 0 ) {
					Blue = 0 
				}
				if (Blue > 255 ) {
					Blue = 255
				}
			}
			
			return {
				r: Red,
				g: Green,
				b: Blue
			}
			
		},
		
		tempToColor2 (index,t) {
			// HSL(t, 1, 0.5, R, G, B);
			// H S L
			var temperature = t/ 1000;
			var H = temperature
			var S = 1
			var L = 0.5
			
			var var_1;
			var var_2;
			var Hu= H + 0.53;
			var Hd= H - 0.53;
			
			var Rval = 255
			var Gval = 255
			var Bval = 255
			
			if ( S == 0 )                       //HSL from 0 to 1
			{
			Rval = L * 255;                      //RGB results from 0 to 255
			Gval = L * 255;
			Bval = L * 255;
			}
			else
			{
			if ( L < 0.5 ) 
			var_2 = L * ( 1 + S );
			else           
			var_2 = ( L + S ) - ( S * L );
					
			var_1 = 2 * L - var_2;
					
			Rval = 255 * this.Hue_2_RGB( var_1, var_2, Hu );
			Gval = 255 * this.Hue_2_RGB( var_1, var_2, H );
			Bval = 255 * this.Hue_2_RGB( var_1, var_2, Hd );
			}
			
			
			return {
			r: Rval,
			g: Gval,
			b: Bval
			}
		},
		
				
		
		Hue_2_RGB( v1, v2, vH )             //Function Hue_2_RGB
		{
		if ( vH < 0 ) 
			vH += 1;
		if ( vH > 1 ) 
			vH -= 1;
		if ( ( 6 * vH ) < 1 ) 
			return ( v1 + ( v2 - v1 ) * 6 * vH );
		if ( ( 2 * vH ) < 1 ) 
			return ( v2 );
		if ( ( 3 * vH ) < 2 ) 
			return ( v1 + ( v2 - v1 ) * (.66-vH) * 6 );
			return ( v1 );
		}
    }
  }



</script>

<style scoped>
  #fiosthermsDynamic{
   /* width: 80vw;
    height: 12.63vw; */
	width: 80%;
	height: 360px;
  }
</style>
