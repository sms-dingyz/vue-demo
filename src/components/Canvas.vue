<template>
  <canvas :class="{ active: painting }" ref="canvas" @mousedown="startPainting" @mouseup="finishPainting" @mousemove="draw">
  </canvas>
</template>

<script>
export default {
  props: ["color", "size"],
  data() {
    return {
      painting: false,
      context: null,
      mouseX: null,
      mouseY: null,
	  
	 };
  },
  mounted() {
    this.context = this.$refs.canvas.getContext("2d");
    this.context.lineWidth = this.size;
    this.context.strokeStyle = this.color;
    this.context.lineCap = "round";
  },
  methods: {
	  
	 setColor(color) {	    
	      this.context.strokeStyle = color;
	    },
	    setSize(size) {
	    
	      this.context.lineWidth = size;
	    },
		  
    draw(e) {
      if (!this.painting) {
        return;
      }
      this.context.beginPath();
      this.context.moveTo(this.mouseX, this.mouseY);
      this.mouseX = e.pageX - this.$refs.canvas.offsetLeft;
      this.mouseY = e.pageY - this.$refs.canvas.offsetTop;
      this.context.lineTo(this.mouseX, this.mouseY);
      this.context.stroke();
    },
    startPainting(e) {
      this.painting = true;
      this.mouseX = e.pageX - this.$refs.canvas.offsetLeft;
      this.mouseY = e.pageY - this.$refs.canvas.offsetTop;
    },
    finishPainting() {
      this.painting = false;
    }
  }
};
</script>

<style scoped>

</style>
