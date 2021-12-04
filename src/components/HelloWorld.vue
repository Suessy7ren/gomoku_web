<template>
  <div class="playingChess">
    <div class="hello">
      <input type="text" v-model="msgSend">
      <button @click="send">send info</button>
    </div>
  </div>
</template>

<script>
import config from "../config"
export default {
  data () {
    return {
      path:"ws://"+config.address,
      socket:"",
      msgSend:""
    }
  },
  mounted () {
    // 初始化
    this.init()
  },
  methods: {
    init: function () {
      if(typeof(WebSocket) === "undefined"){
        alert("您的浏览器不支持socket")
      }else{
        // 实例化socket
        this.socket = new WebSocket(this.path)
        // 监听socket连接
        this.socket.onopen = this.open
        // 监听socket错误信息
        this.socket.onerror = this.error
        // 监听socket消息
        this.socket.onmessage = this.getMessage
      }
    },
    open: function () {
      console.log("socket连接成功")
    },
    error: function () {
      console.log("连接错误")
    },
    getMessage: function (msg) {
      console.log("收到" + msg.data)
    },
    send: function () {
      this.socket.send(this.msgSend)
    },
    close: function () {
      console.log("socket已经关闭")
    }
  },
  destroyed () {
    // 销毁监听
    this.socket.onclose = this.close
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
