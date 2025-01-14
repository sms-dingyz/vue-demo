<template>
  <div class="dashboard">
    <!-- <el-row :gutter="24">
      <el-col :span="6" v-for="(val, key, i) in totalCardList">
        <el-card class="total-card" :style="{backgroundColor: val.backgroundColor}">
          <div class="total-card-title">
            <span>{{val.title}}</span>
            <el-tooltip :content="val.desc" placement="top" effect="dark">
              <el-icon><InfoFilled /></el-icon>
            </el-tooltip>
          </div>
          <div class="total-card-number">{{val.text}}</div>
          <div class="total-card-footer" v-if="i % 2 === 0">
            <span :class="{actived: val.type === 'day'}">日</span>
            <span :class="{actived: val.type === 'week'}">周</span>
            <span :class="{actived: val.type === 'month'}">月</span>
          </div>
        </el-card>
      </el-col>
    </el-row> -->
    <h4 class="dashboard-title" style="margin-top: 0;padding-top: 0">炉次信息</h4>
		<div class="heat-box">
		<template v-for="(item) in heatList" :key="item.id">
		<div class="heat-box-col">
			<p
			v-for="(sub) in item.list"
			:key="sub.name"
			:class="sub.visited ? 'visited' : ''">
			<span class="heat-box-label">{{sub.text}}</span>
			{{sub.data}}
			</p>
		</div>
		</template>
    </div>

	<h4 class="dashboard-title">分析结果</h4>
<!--	<el-table :data="AnalysisData.data" style="width: 100%;margin-bottom: 40px">-->

<!--	  <template v-for="key in AnalysisData.keys" :key="key" >-->
<!--	    <el-table-column :prop="key" :label="key" />-->
<!--    </template>-->
<!--	</el-table>-->
    <el-descriptions
      title=""
      direction="vertical"
      :column="11"
      border
      style="margin-bottom: 40px"
    >
      <template v-for="(value, key) in AnalysisData" :key="key" >
        <el-descriptions-item :label="key">{{value}}</el-descriptions-item>
      </template>
    </el-descriptions>

    <!-- <el-divider/> -->
    <el-row :gutter="48" class="task-box">
      <template v-for="(item ) in parameterList" :key="item.id">
        <el-col :span="8" class="border-right ">
          <h4>
            <span :style="{backgroundColor: item.iconBackgroundColor}"><el-icon color="#fff"><component :is="item.icon"></component></el-icon></span>
            {{item.title}}</h4>
          <div class="task-box-col">
            <p
              v-for="(sub ) in item.list"
              :key="sub.name"
              :class="[sub.visited ? 'visited' : '']">
              <span class="task-box-label">{{sub.text}}</span>
              {{sub.data}}
            </p>
          </div>

        </el-col>
      </template>
    </el-row>

	<h4 class="dashboard-title">模型结果</h4>
	<el-divider/>
	<el-table :data="tableData" style="width: 100%;margin-bottom: 40px;color:#333;font-size:15px;">
	<el-table-column prop="name" label="Name" width="120" align='center' />

	<el-table-column prop="fSprayWaterFlow1" label="" width="120" align='center'/>
	<el-table-column prop="fSprayWaterFlow2" label="" width="120" align='center'/>
	<el-table-column prop="fSprayWaterFlow3" label="" width="120" align='center'/>
	
	<el-table-column prop="fSprayWaterFlow4" label="" width="120" align='center'/>
	<el-table-column prop="fSprayWaterFlow5" label="" width="120" align='center'/>
	<el-table-column prop="fSprayWaterFlow6" label="" width="120" align='center'/>



	<el-table-column prop="Shell_loose" label="Shell_loose(mm)" width="160"  align='center' />
	<el-table-column prop="Shell_fix" label="Shell_fix(mm)" width="140" align='center'/>
	<el-table-column prop="Core" label="Core(°C)" width="100" align='center'/>
	<el-table-column prop="Mean" label="Mean(°C)" width="100" align='center'/>
	<el-table-column prop="loose" label="Loose" width="100" align='center'/>
	<el-table-column prop="fix" label="Fix" width="100" align='center'/>
	<el-table-column prop="set" label="Set" />
	</el-table>


	<!-- <h4 class="dashboard-title">厚度曲线</h4>
	<el-divider/>
	<ShellThickness></ShellThickness>
    <el-divider/> -->

    <h4 class="dashboard-title">温度曲线</h4>
	<el-divider/>
    <!-- <div id="line1" style="height: 300px; width: 100%"></div> -->
	<TemperatureCurve></TemperatureCurve>
	
	<!--
	<div id="temperaturecurve" style="height: 600px; width: 100%"></div>
		<div style="margin-top: -100px;font-weight: bold">Spay Schedule No.{{this.Schedule}}</div>
	-->
    <el-divider/>
	
	<!--
	<h4 class="dashboard-title">温度图形</h4>
	<el-divider/>
	<Fiostherms3></Fiostherms3>
	-->
	
	<h4 class="dashboard-title">温度侧视图</h4>
	<el-divider/>
	<Fiostherms3Dynamic></Fiostherms3Dynamic>
	
	
	
	
    <el-divider/>

	<h4 class="dashboard-title">温度俯视图</h4>
	<el-divider/>
	
	
	<FiosthermsCutView></FiosthermsCutView>
    <el-divider/>


	<h4 class="dashboard-title">图形列表</h4>
	<el-divider/>
	<el-row :gutter="24">
	<el-col :span="12" class="border-right">
		<div class="y-desc">ShellThickness[mm]</div>
	<div id="pie1" style="height: 600px; width: 100%"></div>
		<div style="margin-top: -100px;font-weight: bold">Distance from mold level[m]</div>
	</el-col>
	<!-- <el-col :span="8" class="border-right">
	<div id="pie2" style="height: 200px; width: 100%"></div>
	</el-col> -->
	<el-col :span="12">
	<!-- <div id="pie3" style="height: 200px; width: 100%"></div> -->		
		<div id="pie3" style="height: 600px; width: 100%"></div>
		<div style="margin-top: -100px;font-weight: bold">Spay Schedule No.{{this.Schedule}}</div>
	</el-col>
	</el-row>
  </div>
</template>
<script>
  import * as echarts from 'echarts';
  import { shallowRef } from 'vue'
  import dayjs from 'dayjs'
  import { DocumentCopy, List, SoldOut, Van } from '@element-plus/icons-vue'
  import TemperatureCurve from './TemperatureCurve.vue'
  import ShellThickness from './ShellThickness.vue'
  import FiosthermsCutView from './FiosthermsCutView.vue'

  import Fiostherms3 from './Fiostherms3.vue'
  import Fiostherms3Dynamic from './Fiostherms3Dynamic.vue'

  
  import { getData, delZero } from './dataCollectedPlcDataBag'
  import { getShellThicknessData } from './ShellThickness'

  const isLocalData = 1

  export default {
	components: {
		TemperatureCurve,
		ShellThickness,
		Fiostherms3,
		Fiostherms3Dynamic,
		FiosthermsCutView,
	},
    data() {
        return {
			rawThicknessData: {} ,
			Schedule: '4',
			iDscMode: '1',
			maxList: [],
			minList: [],
			
			rawfTempCurveData:{},
			
			
			

			//water flow echart bar data
			waterFlowBar1Data:[],
			waterFlowBar2Data:[],
			waterFlowBar3Data:[],

			timer: null,
			webScoketData: {},
			heatData: [],
			moldPara: [],
			moldWater: [],
			spayWater:[],
			castTemperature: "",
			castSpeed: "",
			//moldwater
			fSoliLength: "6073",

			//fcurve
			fLiquidus: "1518.0",
			FSolidus: "1468.0",

			totalCardList: {
				borrowBook: { title: 'aa次', desc: 'xxxx', text: '0', backgroundColor: '#106abc', type: 'day'},
				borrowBookTotal: { title: 'bbb', desc: 'bbbb', text: '0', backgroundColor: '#12b43a' },
				user: { title: 'ccc', desc: 'cccc', text: '0', backgroundColor: '#e68e08', type: 'day' },
				userTotal: { title: 'ddd', desc: 'ddddd', text: '0', backgroundColor: '#6530b0', type: 'day'}
          },
          heatList: [
            {
              id: 'order',
              title: '炉次信息',
              icon: shallowRef(DocumentCopy),
              iconBackgroundColor: '#a894f7',
              list: [
				{ visited: false, text: '炉号：',  data: 'B223Y38925'},
				{ visited: false, text: '钢种：',  data: ' SPHC-A'},
                { visited: false, text: '当前日期：',  data: dayjs(new Date().getTime()).format('YYYY-MM-DD')},
                { visited: false, text: '当前时间：',  data: dayjs(new Date().getTime()).format('HH:mm')}
              ]
            }
          ],

          parameterList: [
            {
              id: 'order',
              title: '结晶器参数',
              icon: shallowRef(DocumentCopy),
              iconBackgroundColor: '#a894f7',
              // list: [],
			list: [
				{text: 'Cast Temperature(°C)：', data:'1552.05'},
				{text: 'Cast Speed(m/min)：', data:'4.8'},
				{text: 'FLiquidus(°C)：', data:'1518.0'},
				{text: 'FSolidus(°C)：', data:'1468.0'},
				{text: 'Thickness(mm)：',  data:'52.00-61.23'},
				{text: 'Width(mm)：',  data:'1536.90'}
			]

            },
            {
              id: 'logisitc',
              title: '结晶器冷却水参数',
              icon: shallowRef(Van),
              iconBackgroundColor: '#4ecccb',
              list: [
				{text: 'Entry Temperature(°C)：', data:'28.47'},
				{text: 'Delta Temperature(°C)：', data: '3.80'},
				{text: 'Mold Level(mm)：', data:'22.03'},
				{text: 'Solidification(mm)：', data: '5300.20' },
				{text: 'Total Water：', data:''}
			],

            },
            {
              id: 'package',
              title: '二冷水参数',
              icon: shallowRef(SoldOut),
              iconBackgroundColor: '#009eff',
              list: [
				{text: 'Entry Temperature(°C)：', data: '20.0'},
				{text: 'Schedule：', data: '4'},
			]

            }
          ],

          AnalysisData: {},

          tableData: [
			{
				name: 'Mold',
				fSprayWaterFlow: '',
				temperature2: '',
				temperature3: '',

				Shell_loose:'5.94',
				Shell_fix:'5.94',
				Core: '1555',
				Mean: '1533',
				loose: '810',
				fix: '120',
				set: '',

			},
            {
				name: 'RING',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				temperature2: '',
				temperature3: '',

				Shell_loose:'5.92',
				Shell_fix:'5.92',
				Core: '1555',
				Mean: '1533',
				loose: '810',
				fix: '120',
				set: '',
            },
            {
				name: 'Seg0o',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',

				Shell_loose:'9.53',
				Shell_fix:'9.53',
				Core: '1554',
				Mean: '1530',
				loose: '839',
				fix: '120',
				set: '',
            },
            {
				name: 'Seg0m',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',

				Shell_loose:'17.03',
				Shell_fix:'17.03',
				Core: '1553',
				Mean: '1533',
				loose: '854',
				fix: '120',
				set: '',
            },
            {
				name: 'Seg0u',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				Shell_loose:'24.98',
				Shell_fix:'24.98',
				Core: '1555',
				Mean: '1533',
				loose: '806',
				fix: '120',
				set: '',
            },
            {
				name: 'Seg1',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				fSprayWaterFlow4: '',
				fSprayWaterFlow5: '',
				fSprayWaterFlow6: '',

				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
            },
			{
				name: 'Seg2',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				fSprayWaterFlow4: '',
				fSprayWaterFlow5: '',
				fSprayWaterFlow6: '',
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			},
			{
				name: 'S3&4',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				fSprayWaterFlow4: '',
				fSprayWaterFlow5: '',
				fSprayWaterFlow6: '',
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			},
			{
				name: 'S5&6',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				fSprayWaterFlow4: '',
				fSprayWaterFlow5: '',
				fSprayWaterFlow6: '',
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			},
			{
				name: 'S7&8',
				fSprayWaterFlow1: '',
				fSprayWaterFlow2: '',
				fSprayWaterFlow3: '',
				fSprayWaterFlow4: '',
				fSprayWaterFlow5: '',
				fSprayWaterFlow6: '',
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			},
			{
				name: 'S9-11',
				fSprayWaterFlow1: '',
			
				fSprayWaterFlow3: '',
			
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			},
			{
				name: 'S12-1',
				fSprayWaterFlow1: '',
			
				fSprayWaterFlow3: '',
			
				Shell_loose:'30.31',
				Shell_fix:'30.31',
				Core: '1556',
				Mean: '1533',
				loose: '907',
				fix: '120',
				set: '',
			}
          ]
        }
    },
    mounted() {

		if ( isLocalData === 1 ) {
			var rawData = getShellThicknessData()
			this.rawThicknessData.fElementPositionArray = rawData.fElementPositionArray ;

			
			this.rawThicknessData.fShellUp_3DArray = rawData.fShellUp_3DArray ;
		
		//console.log(" _____ dyz fShellThickness fElementPositionArray",this.rawThicknessData.fElementPositionArray)
		///console.log(" _____ dyz fShellThickness fShellUp_3DArray",this.rawThicknessData.fShellUp_3DArray)

		
			this.setThicknessEchartData()
			return
		}
		this.timer = setInterval(() => {
		try {
			const index1 = this.heatList[0].list.findIndex(item => item.text === '当前日期：');
			this.heatList[0].list[index1].data = dayjs(new Date().getTime()).format('YYYY-MM-DD');
			const index2 = this.heatList[0].list.findIndex(item => item.text === '当前时间：');
			this.heatList[0].list[index2].data = dayjs(new Date().getTime()).format('HH:mm:ss');
			// this.webScoketData = getData();
			// this.webDataParser()
		// eslint-disable-next-line no-empty
		} catch (err) {}
		},  1000)
		// this.initiateData()
		// this.webScoketData = getData();
		// this.webDataParser()

		var messagetype = 'undefined'
		this.socket = new WebSocket('ws://127.0.0.1:50000');
		this.socket.onmessage = (event) => {

		if(typeof event.data === 'string') {
			// this.webScoketData = JSON.parse(event.data);
			// messagetype = this.webScoketData.messagetype

			var tmpData = JSON.parse(event.data)
			messagetype = tmpData.messagetype
		}

       this.webScoketData = JSON.parse(event.data);
       this.webDataParser( messagetype )

		//this.messages.push(event.data);
		}
		//this.webScoketData = getData();
		// this.setThicknessEchartData()
      // this.getPie2()
      this.setWaterFlowEchartData()
      this.getLine1()
    },
    onUnmounted() {
      clearInterval(this.timer)
      this.timer = null
    },
    methods: {

      initiateData(){
		this.webScoketData = getData();
        this.webDataParser()
      },
		getAnalysisData( analysisData ) {
			// console.log(" _____ analysisData:",analysisData)
			this.AnalysisData = JSON.parse( analysisData )
      },
      webDataParser( _messagetype ){

		var arr = []
		var i = 0
		var threeDArray = []		
		var messagetype = this.webScoketData.messagetype
		messagetype = _messagetype ;

        if ( messagetype === "CollectedPlcDataBag") {

			//console.log(" _____ messagetype is CollectedPlcDataBag")
			this.heatData  = []
            this.heatData = [
             {text: '钢种：', data: this.webScoketData.Steelgrade},
             {text: '炉号：', data: this.webScoketData.heatNo}
           ]


			this.heatList[0].list[0].data = this.webScoketData.heatNo
			this.heatList[0].list[1].data = this.webScoketData.Steelgrade

			this.getAnalysisData( this.webScoketData.AnalysisData )



            //moldpara
            const CollectedPlcData = JSON.parse( this.webScoketData.CollectedPlcData )
			console.log(" _____ CollectedPlcData:",CollectedPlcData)
			//console.log("_________aaa  CollectedPlcData:",this.iDscMode)
            this.moldPara = [
              {text: 'Cast Temperature(°C)：', data: CollectedPlcData.fTundTemp.toFixed(2)},
              {text: 'Cast Speed(m/min)：', data: CollectedPlcData.fCastSpeed.toFixed(2)},
              {text: 'FLiquidus(°C)：', data: this.fLiquidus},
              {text: 'FSolidus(°C)：', data:this.fSolidus},
              {text: 'Thickness(mm)：', data:CollectedPlcData.fThickness1.toFixed(2) + "-" + CollectedPlcData.fThickness4.toFixed(2)},
              {text: 'Width(mm)：', data:CollectedPlcData.fWidth.toFixed(2)}
            ]
			
			this.Schedule = CollectedPlcData.iCoolCurveNo
			this.iDscMode = CollectedPlcData.iDscMode.toFixed(0)
			
			this.waterFlowBar1Data = []
			this.waterFlowBar2Data = []
			this.waterFlowBar3Data = []
			// this.tableData[0].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow1.toFixed(0)
			this.tableData[1].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow1.toFixed(0)
			this.tableData[1].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow2.toFixed(0)
			
			this.tableData[2].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow3.toFixed(0)
			this.tableData[2].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow4.toFixed(0)
			this.tableData[2].fSprayWaterFlow3 = CollectedPlcData.fSprayWaterFlow5.toFixed(0)

			this.tableData[3].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow6.toFixed(0)
			this.tableData[3].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow7.toFixed(0)
			this.tableData[3].fSprayWaterFlow3 = CollectedPlcData.fSprayWaterFlow8.toFixed(0)

			this.tableData[4].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow9.toFixed(0)
			this.tableData[4].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow10.toFixed(0)
			this.tableData[4].fSprayWaterFlow3 = CollectedPlcData.fSprayWaterFlow11.toFixed(0)

			this.tableData[5].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow12.toFixed(0)
			this.tableData[5].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow13.toFixed(0)
			this.tableData[5].fSprayWaterFlow3 = CollectedPlcData.fSprayWaterFlow14.toFixed(0)
			this.tableData[5].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow15.toFixed(0)
			this.tableData[5].fSprayWaterFlow5 = CollectedPlcData.fSprayWaterFlow16.toFixed(0)
			this.tableData[5].fSprayWaterFlow6 = CollectedPlcData.fSprayWaterFlow17.toFixed(0)

			this.tableData[6].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow18.toFixed(0)
			this.tableData[6].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow19.toFixed(0)
			this.tableData[6].fSprayWaterFlow3 = CollectedPlcData.fSprayWaterFlow20.toFixed(0)
			this.tableData[6].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow21.toFixed(0)
			this.tableData[6].fSprayWaterFlow5 = CollectedPlcData.fSprayWaterFlow22.toFixed(0)
			this.tableData[6].fSprayWaterFlow6 = CollectedPlcData.fSprayWaterFlow23.toFixed(0)
			
			
			this.tableData[7].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow24.toFixed(0)
			this.tableData[7].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow25.toFixed(0)
			this.tableData[7].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow26.toFixed(0)
			this.tableData[7].fSprayWaterFlow5 = CollectedPlcData.fSprayWaterFlow27.toFixed(0)
			
			this.tableData[8].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow28.toFixed(0)
			this.tableData[8].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow29.toFixed(0)
			this.tableData[8].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow30.toFixed(0)
			this.tableData[8].fSprayWaterFlow5 = CollectedPlcData.fSprayWaterFlow31.toFixed(0)
			
			
			this.tableData[9].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow32.toFixed(0)
			this.tableData[9].fSprayWaterFlow2 = CollectedPlcData.fSprayWaterFlow33.toFixed(0)
			
			this.tableData[9].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow34.toFixed(0)
			this.tableData[9].fSprayWaterFlow5 = CollectedPlcData.fSprayWaterFlow35.toFixed(0)
			
			
			this.tableData[10].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow36.toFixed(0)
			this.tableData[10].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow37.toFixed(0)
			
			
			this.tableData[11].fSprayWaterFlow1 = CollectedPlcData.fSprayWaterFlow38.toFixed(0)
			this.tableData[11].fSprayWaterFlow4 = CollectedPlcData.fSprayWaterFlow39.toFixed(0)
			
			
			//
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow1_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow2_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow3_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow4_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow5_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow6_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow7_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow8_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow9_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow10_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow11_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow12_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow13_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow14_1)
			
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow15_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow16_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow17_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow18_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow19_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow20_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow21_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow22_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow23_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow24_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow25_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow26_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow27_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow28_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow29_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow30_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow31_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow32_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow33_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow34_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow35_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow36_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow37_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow38_1)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow39_1)
			
			
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow1_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow2_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow3_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow4_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow5_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow6_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow7_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow8_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow9_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow10_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow11_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow12_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow13_2)
			this.maxList.push(CollectedPlcData.fMinMaxWaterFlow14_2)
			
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow15_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow16_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow17_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow18_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow19_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow20_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow21_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow22_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow23_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow24_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow25_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow26_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow27_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow28_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow29_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow30_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow31_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow32_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow33_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow34_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow35_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow36_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow37_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow38_2)
			this.minList.push(CollectedPlcData.fMinMaxWaterFlow39_2)
			
			console.log("_________ iDscMode:",this.iDscMode)
			if ( this.iDscMode === '3' ) {
				this.tableData[1].set = CollectedPlcData.nTempCurve1.toFixed(0)
				this.tableData[2].set = CollectedPlcData.nTempCurve2.toFixed(0)
				this.tableData[3].set = CollectedPlcData.nTempCurve3.toFixed(0)
				this.tableData[4].set = CollectedPlcData.nTempCurve4.toFixed(0)
				this.tableData[5].set = CollectedPlcData.nTempCurve5.toFixed(0)
				this.tableData[6].set = CollectedPlcData.nTempCurve6.toFixed(0)
			}
			
			//bar data
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow1.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow3.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow6.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow9.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow12.toFixed(0))		
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow15.toFixed(0))	
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow18.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow21.toFixed(0))		
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow24.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow26.toFixed(0))			
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow28.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow30.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow32.toFixed(0))			
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow34.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow36.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow37.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow38.toFixed(0))
			this.waterFlowBar1Data.push(CollectedPlcData.fSprayWaterFlow39.toFixed(0))
		
		
			this.waterFlowBar2Data.push(0)
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow4.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow7.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow10.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow13.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow16.toFixed(0))

			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow19.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow22.toFixed(0))

			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow25.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow27.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow29.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow31.toFixed(0))
			
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow33.toFixed(0))
			this.waterFlowBar2Data.push(CollectedPlcData.fSprayWaterFlow35.toFixed(0))
			this.waterFlowBar2Data.push(0)
			this.waterFlowBar2Data.push(0)
		
			
			
			
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow5.toFixed(0))
			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow8.toFixed(0))
			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow11.toFixed(0))
			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow14.toFixed(0))
			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow17.toFixed(0))

			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow20.toFixed(0))

			this.waterFlowBar3Data.push(CollectedPlcData.fSprayWaterFlow23.toFixed(0))
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
			this.waterFlowBar3Data.push(0)
		
			
		
			this.setWaterFlowEchartData()



			this.castSpeed = CollectedPlcData.fCastSpeed.toFixed(2)
			this.castTemperature = CollectedPlcData.fTundTemp.toFixed(2)
			this.parameterList[0].list = this.moldPara

            //moldWater
            this.moldWater = [
              {text: 'Entry Temperature(°C)：', data: CollectedPlcData.fWaterTemp.toFixed(2)},
              {text: 'Delta Temperature(°C)：', data: CollectedPlcData.fCastSpeed},
              {text: 'Mold Level(mm)：', data:CollectedPlcData.fLevel.toFixed(2)},
              {text: 'Solidification(mm)：', data: this.fSoliLength }, //fSoliLength
              {text: 'Total Water：', data:''}
            ]
			this.parameterList[1].list = this.moldWater


            //spayWater
            this.spayWater = [
              {text: 'Entry Temperature(°C)：', data: CollectedPlcData.fSprayWaterTemp.toFixed(1)},
              {text: 'Schedule：', data: CollectedPlcData.iCoolCurveNo},
            ]
			
			this.parameterList[2].list = this.spayWater


          } else if ( messagetype === "fTempCurve") {
			console.log(" _____ messagetype is fTempCurve:",this.webScoketData)
			//console.log(" _____ messagetype is fTempCoreArray:",this.webScoketData.fTempCoreArray)			
			this.fLiquidus = this.webScoketData.fLiquidus
			this.fSolidus = this.webScoketData.fSolidus
			this.rawfTempCurveData = this.webScoketData		
			var data = JSON.parse(this.webScoketData.fTempCurvPosArray)
			for ( i = 0; i < data.length; i++) {
			
				arr.push(data[i])
			
			}
			this.rawfTempCurveData.fElementPositionArray = arr ;

			threeDArray = []		
			data = JSON.parse(this.webScoketData.fTempCoreArray)
			for ( i = 0; i < data.length; i++) {
			
				if ( ( data[i] - 600) > -600 ) {
				threeDArray.push(data[i])
				}
			
			}
			this.rawfTempCurveData.fTempCoreArray = threeDArray ;
			
			threeDArray = []
			
			data = JSON.parse(this.webScoketData.fTempMeanArray)
			for ( i = 0; i < data.length; i++) {
			if ( ( data[i] - 600) > -600 ) {
				threeDArray.push(data[i])
				}
			
			}			
			this.rawfTempCurveData.fTempMeanArray = threeDArray ;
			
			
			threeDArray = []
			data = JSON.parse(this.webScoketData.fTempSideArray)
			for ( i = 0; i < data.length; i++) {
				threeDArray.push(data[i])
			}			
			this.rawfTempCurveData.fTempSideArray = threeDArray ;
			
			
			// this.setTemperatureCurveEchartData()


          } else if (  messagetype === "ModelResult" ) {
			// console.log(" _____ messagetype is ModelResult:",this.webScoketData.ModelResultData)
			const ModelResultData = JSON.parse( this.webScoketData.ModelResultData )
			//console.log(" _____ messagetype is ModelResultData:",ModelResultData)
			
			const ModelResultDataSegmentTemp = JSON.parse(this.webScoketData.ModelResultDataSegmentTemp )
			
	
					
			this.fSoliLength = ModelResultData.fSoliLength
			this.tableData.forEach( function( item, index )  {
				const key = `fShellThickness1_${index + 1}`;
				
				if(Object.keys(ModelResultData).includes(key)) {
					item.Shell_loose = Number(ModelResultData[key]).toFixed(0) // 2--> 0
				
				}

				const key2 = `fShellThickness2_${index + 1}`;			
				if(Object.keys(ModelResultData).includes(key2)) {
				item.Shell_fix = Number(ModelResultData[key2]).toFixed(0) // 2--> 0
				}
				
			
				
				const key3 = `fTemp${index +1 }`
				if(Object.keys(ModelResultDataSegmentTemp).includes(key3)) {
				item.Core = Number(ModelResultDataSegmentTemp[key3]).toFixed(0) // 2--> 0
				}
				const key4 = `fTemp${index +13 }`
				if(Object.keys(ModelResultDataSegmentTemp).includes(key4)) {
				item.Mean = Number(ModelResultDataSegmentTemp[key4]).toFixed(0) // 2--> 0
				}
				
				const key5 = `fTemp${index +25 }`
				if(Object.keys(ModelResultDataSegmentTemp).includes(key5)) {
				item.loose = Number(ModelResultDataSegmentTemp[key5]).toFixed(0) // 2--> 0
				}
				
				const key6 = `fTemp${index +27 }`
				if(Object.keys(ModelResultDataSegmentTemp).includes(key6)) {
				item.fix = Number(ModelResultDataSegmentTemp[key6]).toFixed(0) // 2--> 0
				}
			
		
			})
			
			if ( this.iDscMode === '3' ) {
				//this.tableData[1].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut1.toFixed(0)
				this.tableData[1].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut1.toFixed(0)
				this.tableData[1].fSprayWaterFlow2 = ModelResultData.fSprayWaterFlowOut2.toFixed(0)
				
				
				
				this.tableData[2].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut2.toFixed(0)
				
				this.tableData[3].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut3.toFixed(0)
				this.tableData[3].fSprayWaterFlow2 = ModelResultData.fSprayWaterFlowOut4.toFixed(0)
				this.tableData[3].fSprayWaterFlow3 = ModelResultData.fSprayWaterFlowOut5.toFixed(0)
				
				this.tableData[4].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut6.toFixed(0)
				this.tableData[4].fSprayWaterFlow2 = ModelResultData.fSprayWaterFlowOut7.toFixed(0)
				this.tableData[4].fSprayWaterFlow3 = ModelResultData.fSprayWaterFlowOut8.toFixed(0)
				
				this.tableData[5].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut9.toFixed(0)
				this.tableData[5].fSprayWaterFlow2 = ModelResultData.fSprayWaterFlowOut10.toFixed(0)
				this.tableData[5].fSprayWaterFlow3 = ModelResultData.fSprayWaterFlowOut11.toFixed(0)
				
				this.tableData[6].fSprayWaterFlow1 = ModelResultData.fSprayWaterFlowOut12.toFixed(0)
				this.tableData[6].fSprayWaterFlow2 = ModelResultData.fSprayWaterFlowOut3.toFixed(0)
				this.tableData[6].fSprayWaterFlow3 = ModelResultData.fSprayWaterFlowOut14.toFixed(0)
			}
			
		}
		else if ( messagetype === "fShellThickness" ){
			this.rawThicknessData = this.webScoketData
			arr = []
			var fElementPositionArray = JSON.parse(this.webScoketData.fElementPositionArray)
			for ( i = 0; i < fElementPositionArray.length; i++) {
				if(i === 0 || i === 1 || i === 2 || fElementPositionArray[i] !== 0.0) {
				arr.push( fElementPositionArray[i] )
				} else {
				break;
				}
			}
			this.rawThicknessData.fElementPositionArray = arr ;

			threeDArray = []
			var fShellUp_3DArray = JSON.parse(this.webScoketData.fShellUp_3DArray)
			for ( i = 0; i < fShellUp_3DArray.length; i++) {
				if(i === 0 || i === 1 || i === 2 || fShellUp_3DArray[i] !== 0.0) {
				threeDArray.push(fShellUp_3DArray[i])
				} else {
				break;
				}
			}
			this.rawThicknessData.fShellUp_3DArray = threeDArray ;
		
		//console.log(" _____ dyz fShellThickness fElementPositionArray",this.rawThicknessData.fElementPositionArray)
		///console.log(" _____ dyz fShellThickness fShellUp_3DArray",this.rawThicknessData.fShellUp_3DArray)

		
			this.setThicknessEchartData()

		}
		
		else {
			//console.log(" _____ messagetype is undefined")
		}


      this.getHeatTask()
      this.getMoldParameter()
      this.getMoldWater()
      this.getSpayWater()
		// this.getAnalysisData( getData().AnalysisData )
      },
	
    getHeatTask() {
		//this.heatData.forEach((item) => {
			//if ( this.parameterList[0].list.length === 0 ){
				//  this.heatList[0].list.unshift(
				// { visited: false, text: item.text,  data: item.data},
				//  )
			//}


			//});
    },

	getMoldParameter() {
		if ( this.parameterList[0].list.length === 0 ){
			this.moldPara.forEach((item) => {
			//this.moldPara.forEach((item, index) => {
				this.parameterList[0].list.push(
				{ visited: false, text: item.text,  data: item.data},
				)
			})
		}	
	},

	getMoldWater() {
		if ( this.parameterList[1].list.length === 0 ){
			//this.moldWater.forEach((item, index) => {
			this.moldWater.forEach((item) => {
				this.parameterList[1].list.push(
				{ visited: false, text: item.text,  data: item.data},
				)
			})
		}
	},

	getSpayWater(){

	if ( this.parameterList[2].list.length === 0 ){
		this.spayWater.forEach((item, index) => {
			this.parameterList[2].list.push(
			{ visited: false, text: item.text,  data: item.data},
			)
		})
	}
	else {
		//change value
		// this.spayWater.forEach((item, index) => {
		//   this.parameterList[2].list[index].data = item.data
		// })
	}
	},

      setThicknessEchartData() {
        var chartDom = document.getElementById('pie1');
        var myChart = echarts.init(chartDom);
        var option;
       option = {
         // title: {
         //   left: 'center',
         //   text: 'Strand Temperature'
         // },
         grid: [
           {
             left: 100,
             right: '8%',
             bottom: 150
           }
         ],
         xAxis: {
           type: 'value',
           min: 0,
           //max: 22.5,
			max: 34.5,
           interval: 2,
           axisTick: {
             inside: true
           },
           minorTick: {
             show: true
           }
         },
         yAxis: {
			type: 'value',
			min: 0,
			//max: 35,
			//interval: 5,
			max: 130,
			interval: 10,
			axisTick: {
			inside: true
			},
			minorTick: {
			show: true
			}
         },
         color: ['#EB5B5B', '#C655E9', '#2D53D3'],
         tooltip: {
           trigger: 'axis'
         },
         series: [
		{
			type: 'line',
			showSymbol: false,
			data: this.rawThicknessData.fElementPositionArray.map((item, index) => [
			item,
			this.rawThicknessData.fShellUp_3DArray[index]
			]),
			endLabel: {
			show: true,
			formatter: function (params) {
				return 'solidification \ngauge:' ;
			},
			//color: '#2D53D3',
			color: 'black',
			fontSize: 12
			}
		}
         ]
       };

        option && myChart.setOption(option);
      },
      getPie2() {
      
      },
	setTemperatureCurveEchartData() {
        var chartDom = document.getElementById('temperaturecurve');
        var myChart = echarts.init(chartDom);
        var option;
       option = {
         // title: {
         //   left: 'center',
         //   text: 'Strand Temperature'
         // },
         grid: [
           {
             left: 100,
             right: '8%',
             bottom: 150
           }
         ],
         xAxis: {
           type: 'value',
           min: 0,
		   max: 34,
           interval: 2,
           axisTick: {
             inside: true
           },
           minorTick: {
             show: true
           }
         },
         yAxis: {
           type: 'value',
           min: 600,
		   interval: 100,
		   max: 1600,
           axisTick: {
             inside: true
           },
           minorTick: {
             show: true
           }
         },
         color: ['#EB5B5B', '#C655E9', '#2D53D3'],
         tooltip: {
           trigger: 'axis'
         },
         series: [
		{
			type: 'line',
			showSymbol: false,
			data: this.rawfTempCurveData.fElementPositionArray.map((item, index) => [
			item,
			this.rawfTempCurveData.fTempCoreArray[index]
			]),
			endLabel: {
			show: true,
			formatter: function (params) {
				return 'Core' ;
			},
			//color: '#2D53D3',
			color: 'black',
			fontSize: 12
			}
		},
		{
			type: 'line',
			showSymbol: false,
			data: this.rawfTempCurveData.fElementPositionArray.map((item, index) => [
			item,
			this.rawfTempCurveData.fTempMeanArray[index]
			]),
			endLabel: {
			show: true,
			formatter: function (params) {
				return 'Average' ;
			},
			//color: '#2D53D3',
			color: 'black',
			fontSize: 12
			}
		},
		
		{
			type: 'line',
			showSymbol: false,
			data: this.rawfTempCurveData.fElementPositionArray.map((item, index) => [
			item,
			this.rawfTempCurveData.fTempSideArray[index]
			]),
			endLabel: {
			show: true,
			formatter: function (params) {
				return 'center loose' ;
			},
			//color: '#2D53D3',
			color: 'black',
			fontSize: 12
			}
		}
		
		
         ]
       };

        option && myChart.setOption(option);
      },

      setWaterFlowEchartData() {
        var chartDom = document.getElementById('pie3');
        var myChart = echarts.init(chartDom);
        var option;

        //var xData = ['RING', 'GRID', 'SEG1', 'SEG2','SEG3','SEG4'];
        //var maxList = [440, 1600, 1600, 1500, 800,700, 900,700,700,700,700]
        //var minList = [100, 620, 300, 200, 80, 30, 50,50,50,50,50]
		
		//var maxList = [39,18,31,20,20,26,17,17,17,13,15,10,12,8,8,12,12,8,10,8,6,20,30,20,20,15,20,10,15,15,15,15,15,20,25,25,30]
        //var minList = [630,252,677,521,521,443,369,369,390,312,157,157,165,165,165,116,116,116,145,145,145,352,176,439,220,271,136,339,170,199,100,248,125,324,405,298,73]




		
		var xData = ['RING', 'SEG0O','SEG0m', 'SEG0U','SEG1','SEG2','S3&S4','S5&6','S7&8','S9-11','S12-1'];

        option = {
           title: {
             left: 'center',
             text: 'Absolute Water Flow Rate'
           },
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            },
            formatter: (params) => {
              const chartData = params[0]
              const index = xData.findIndex(item => item === chartData.axisValue);
              if(index >= 0 ) {
                return `<div>max:${this.maxList[index]}</div><div>min:${this.minList[index]}</div>`
              } else {
                return ''
              }

            }
          },
          grid: [
            {
              left: 100,
              right: '8%',
              bottom: 150
            }
          ],
          xAxis: {
            type: 'category',
            axisTick: {
              show: false
            },
            data: xData
          },
          yAxis: {
            type: 'value',
            min: 0,
            interval: 200,
            axisLine: {
              show: true
            },
            axisTick: {
              show: true
            }
          },
          series: [
            {
				name: '',
				//data: [352, 1554, 1455, 1450, 648, 601, 501,501,501,501,501],
				data: this.waterFlowBar1Data,
				type: 'bar',
				itemStyle: {
				color: 'rgba(41, 97, 177, 1)'
				},
				barGap:'0%',
				label: {
				show: true,
				position: 'top',
				formatter: params => {
					if(params.value == 0) {
						return ''
					} else {
						return params.value
					}
				}
				}
            },
            {
              name: '',
              //data: [0, 0, 653, 655, 257, 190,180,180,180,180,180],
			  data: this.waterFlowBar2Data,
              type: 'bar',
              itemStyle: {
                color: 'rgb(86, 192, 198)'
              },
              barGap:'0%',
              label: {
                show: true,
                position: 'top',
                formatter: params => {
                  if(params.value == 0) {
                    return ''
                  } else {
                    return params.value
                  }
                }
              }
            },
            {
              name: '',
              //data: [0, 0, 306, 345, 133, 107 ,102,102,102,102,102],
			  data: this.waterFlowBar3Data,
              type: 'bar',
              itemStyle: {
                color: 'rgb(86, 192, 198)'
              },
              barGap:'0%',
              label: {
                show: true,
                position: 'top',
                formatter: params => {
                  if(params.value == 0) {
                    return ''
                  } else {
                    return params.value
                  }
                }
              }
            },
           
          ]
        };

        option && myChart.setOption(option);
      },
      getLine1() {
     
      }
    }
  }
</script>
<style>
  .dashboard {
    overflow-x: hidden;
    padding: 0 40px 60px;
  }
  .dashboard .total-card {
    color: #fff;
    height: 100%;
    font-size: 14px;
  }
  .total-card-title{
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .total-card-number{
    font-size: 30px;
    font-weight: 700;
    padding: 15px 0 10px;
  }
  .total-card-footer{
    display: flex;
    justify-content: flex-end;
    align-items: flex-end;
  }
  .total-card-footer span{
    margin-left: 10px;
    cursor: pointer;
    color: rgba(255, 255, 255, .45);
  }
  .total-card-footer span:hover, .total-card-footer .actived{
    color: #fff;
  }
  .dashboard-title{
    padding-top: 40px;
    text-align: left;
  }
  .border-right{
    border-right: 1px solid #eee;
  }
  .task-box h4{
    display: flex;
    justify-content: flex-start;
    align-items: center;
    padding-bottom: 10px;
  }
  .task-box h4 span{
    display: inline-block;;
    border-radius: 50%;
    background-color: plum;
    width: 25px;
    line-height:25px;
    margin-right: 10px;
    text-align: center;
  }
  .task-box-col {
    display: flex;
    justify-content: flex-start;
    align-items: flex-start;
    flex-wrap: wrap;
  }
  .task-box p{
    width: 47%;
    padding: 5px 0;
    font-size: 14px;
    //color: #999;
	 color: #333;
    text-align: left;
  }
  .task-box p:nth-of-type(2n) {
    margin-left: 6%;
  }

  .heat-box .heat-box-col {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    flex-wrap: wrap;
  }
  .heat-box .heat-box-col p{
    padding: 0;
    font-size: 14px;
    //color: #999;
	 color: #333;
  }
  .task-box-label, .heat-box-label{
    color: #333;
    font-weight: 500;
  }

  .y-desc{
    transform: rotate(-90deg);
    transform-origin: left;
    position: absolute;
    left: 40px;
    top: 50%;
    font-size: 17px;
    font-weight: bold;
  }
  
   .y-desc2{
    transform: rotate(-90deg);
    transform-origin: left;
    position: absolute;
    left: 53%;
    top: 50%;
    font-size: 17px;
    font-weight: bold;
  }
</style>
