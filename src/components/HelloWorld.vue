<template>
  <div class="app">
    <!-- <Tool @change-color="changeColor" @change-size="changeSize" @reuqest="reuqest"/> -->
    <!-- <Canvas ref="canvas" :color="color" :size="size"/> -->

	 <!-- <Echarts></Echarts> -->
	<!-- <StrandTemperature2></StrandTemperature2> -->
<!--		<Fiostherms3></Fiostherms3>-->
<!--    <Fiostherms></Fiostherms>-->
    <Home></Home>
	<!--  <FiosthermsCutView></FiosthermsCutView>-->


  </div>

</template>

<script>
import Tool from './Tool.vue';
import Canvas from './Canvas.vue';
import Echarts from './Echarts.vue';
// import StrandTemperature2 from './StrandTemperature2.vue';
// import HeatMap from './HeatMap.vue';
import Fiostherms3 from './Fiostherms3.vue';
// import Fiostherms from './Fiostherms.vue';
import FiosthermsCutView from './FiosthermsCutView.vue'

import Home from './home.vue'
import axios from 'axios';
export default {
  components: {
    Tool,
    Canvas,
	Echarts,
    // StrandTemperature2,
	Fiostherms3,
    // Fiostherms,
    Home,
	FiosthermsCutView,

  },
  data() {
    return {}
  },

  methods: {
	request( ) {
		 console.log(" _____ helloWorld change color",color)
		 const url = `/public/api/reports/heats`
		 axios.get(url).then(res => {
		   const  { data } = res
		   this.dataSource = data.content


		 }).catch(() => {

		 })
	},
    changeColor( color) {
	  console.log(" _____ helloWorld change color",color)

	  this.$refs.canvas.setColor(color);
    },
    changeSize( size ) {
	  console.log(" _____ helloWorld change size",size)

	  this.$refs.canvas.setSize(size);
    },
	request(){
		console.log(" _____ request")
	}
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
