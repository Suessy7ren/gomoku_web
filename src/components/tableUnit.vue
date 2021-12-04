<template>
  <div class="tableUnit">
      <img :src="unit" alt="unit" height="40" width="40" @click="choosing">
  </div>
</template>

<script>
export default {
  name: "tableUnit",
  props:["row","column"],
  data(){
    return{
      unit:"",
      black:require("../assets/blackStone.gif"),
      bg:require("../assets/background.gif"),
      white:require("../assets/whiteStone.gif"),
      location:0
    }
  },
  mounted() {
    this.unit = this.bg;
    this.location = this.row * 10 + this.column;
  },
  emits:['choose'],
  methods:{
    init(){
      this.unit = this.bg;
    },
    turnBlack(){
      this.unit = this.black;
    },
    turnWhite(){
      this.unit = this.white;
    },
    choosing(){
      if(this.unit===this.bg){
        this.$emit('choose')
      }
    }
  },
}
</script>

<style scoped>
.tableUnit{
  height: 40px;
  width: 40px;
  padding: 0;
  border: solid;
  margin: 0;
  display: inline-block;
}

</style>
