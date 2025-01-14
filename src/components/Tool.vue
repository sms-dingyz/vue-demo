<template>
  <div class="tool">
    <select v-model="selectedColor" @change="setColor">
      <option value="black">黑色</option>
      <option value="blue">蓝色</option>
      <option value="red">红色</option>
    </select>
    <select v-model="selectedSize" @change="setSize">
      <option value="1">细线</option>
      <option value="5">中等线</option>
      <option value="10">粗线</option>
    </select>
	<button type="button" style="margin-left:10px;margin-top:22px;" @click="request">request</button>
  </div>
</template>

<script>
export default {
	
  data() {
    return {
      selectedColor: "black",
      selectedSize: "1",
    };
  },
  methods: {
	request() {
		console.log(" _____ request")
		 this.$emit("request");
	},
    setColor() {
	  console.log(" _____ tool change color")
      this.$emit("change-color", this.selectedColor);
    },
    setSize() {
		console.log(" _____ tool change size")
      this.$emit("change-size", this.selectedSize);
    },
  }
}
</script>

<style scoped>

</style>

