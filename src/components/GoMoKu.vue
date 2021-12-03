<template>
  <div id="container">
    <div id="navigator" v-show="!playingWindow">
      <div class="menu" v-if="navigator">
        <div class="btn" @click="createRoomButton">我要创建房间</div>
        <div class="btn" @click="enterRoomButton">我要加入房间</div>
      </div>
      <div class="menu" v-else-if="choice">
        <h2>请输入你的昵称</h2>
        <input v-model="nickname">
        <h3>房间号为：{{this.roomNumber}}</h3>
        <div class="btn" @click="initGame('host')" >创建房间</div>
        <div @click="navigator = true">back</div>
      </div>
      <div class="menu" v-else>
        <h3>请输入你的昵称</h3>

        <input v-model="nickname">
        <h3>请输入房间号</h3>

        <input v-model="roomNumber">
        <p @click="initGame('guest')" class="btn" >加入房间</p>
        <div @click="navigator = true">back</div>
      </div>
    </div>
    <div v-show="playingWindow">
      <div class="userinfo">
        <div class="player">host</div>
        <div class="player">{{playerInfo.host.nickname}}</div>
        <div class="player">guest</div>
        <div class="player">{{playerInfo.guest.nickname}}</div>
      </div>
      <div class="tipBar">
        <p>{{ this.tipBar }}</p>
      </div>
      <!--用事件修饰符来阻止点击，算了也不一定这样，还没想好具体功能毕竟-->
      <!--可以用侦听器watch来搞，内部多加参数就是了-->
      <div v-for="j in rows" :key="j">
        <tableUnit v-for="i in column" :row="j" :key="i" :column="i" ref="unit" @choose="takeOneStep(j,i)"></tableUnit>
      </div>
    </div>
  </div>
</template>

<script>
import tableUnit from "@/components/tableUnit";

export default {
  name: "GoMoKu",
  components:{
    tableUnit
  },
  data(){
    return{
      socket:null,
      path:"ws://10.250.169.243:8080",
      msg:{
        sender:"client",
        name:"嘉琳今天吃浩睿",
        content:[],
      },

      playingWindow:false,
      navigator:true,
      choice:true,
      rows:[0,1,2,3,4,5,6,7,8,9],
      column:[0,1,2,3,4,5,6,7,8,9],
      tipBar:"游戏开始",

      nickname:"",
      roomNumber:"输入昵称后创建房间即可获得房间号",

      role:{
        color:"",
        turn:false
      },

      playerInfo:{
        host:{
          nickname:"睿睿公主"
        },
        guest:{
          nickname:"嘉琳今天吃什么"
        }
      }
    }
  },
  methods:{
    createRoomButton(){
      this.navigator = false;
      this.choice = true;
    },
    enterRoomButton(){
      this.navigator = false;
      this.choice = false;
    },
    initGame(role){
      if(this.nickname === ""){
    //    这里得改一下判断
        alert("请先输入昵称");
      }else{
        this.msg.content.push(this.nickname);
        if(role === "host"){
          this.init('/createroom')
        }else if(role === "guest"){
          if(this.roomNumber === ""){
            alert("请输入房间号");
          }else{
            this.init('/enterroom')
          }
        }
      }
    },
    init(role){
      if(typeof(WebSocket) === "undefined"){
        alert("您的浏览器不支持socket")
      }else{
        // 实例化socket
        this.socket = new WebSocket(this.path+role)
        // 监听socket连接
        if(role === '/createroom'){
          this.socket.onopen = () =>{
            this.msg.name="createRoom";
            this.playerInfo.host.nickname = this.nickname;
            console.log("socket连接成功")
            console.log(JSON.stringify(this.msg))
            this.socket.send(JSON.stringify(this.msg));
          }
        }else if(role === '/enterroom'){
          this.socket.onopen = () =>{
            this.msg.name="enterRoom";
            this.playerInfo.guest.nickname = this.nickname;
            this.msg.content.push(this.roomNumber);
            console.log("socket连接成功")
            this.socket.send(JSON.stringify(this.msg));
          }
        }
        // 监听socket错误信息
        this.socket.onerror = this.error
        // 监听socket消息
        this.socket.onmessage = this.getMessage
      }
    },
    error(){
      console.log("连接错误")
    },
    getMessage(msg){
      console.log("收到" + msg.data)
      let messageBody = JSON.parse(msg.data);
      let msgName = messageBody.name;
      if(msgName === "roomNumber"){
        this.roomNumber = messageBody.content[0];
        alert("房间创建成功，房间号为"+this.roomNumber);
        alert("请等待玩家进入房间");
      }
      // if(msgName === "hostNickname"){
      //   this.playerInfo.host.nickname = messageBody.content[0];
      // }
      if(msgName === "hostStartGame"){
        this.playerInfo.guest.nickname = messageBody.content[0];
        this.role.color = messageBody.content[1];
        if(this.role.color === 'black') {
          this.role.turn = true;
          alert("你是先手")
        }
        else {
          this.role.turn = false;
          alert("你是后手")
        }
        this.playingWindow = true;
      }
      if(msgName === "guestStartGame"){
        alert("即将加入"+messageBody.content[0]+"的房间");
        this.playerInfo.guest.nickname = this.nickname;
        this.playerInfo.host.nickname = messageBody.content[0];
        this.role.color = messageBody.content[1];
        if(this.role.color === 'black') {
          this.role.turn = true;
          alert("你是先手")
        }
        else {
          this.role.turn = false;
          alert("你是后手")
        }
        this.playingWindow = true;
      }
      if(msgName === "opponentStep"){
        if(messageBody.content[1] === "victory"){
          let x = parseInt(messageBody.content[0]);
          if(this.role.color === 'white')
            this.$refs.unit[x].turnBlack();
          else if(this.role.color === 'black')
            this.$refs.unit[x].turnWhite();
          alert("对方赢了");
          this.msg.name = "gameFinish";
          this.msg.content = []
          this.socket.send(JSON.stringify(this.msg));
        }else{
          let x = parseInt(messageBody.content[0]);
          if(this.role.color === 'white')
            this.$refs.unit[x].turnBlack();
          else if(this.role.color === 'black'){
            this.$refs.unit[x].turnWhite();
          }
          this.role.turn = true;
        }
      }
      if(msgName === "victory"){
        alert("你赢了")
        this.msg.name = "gameFinish";
        this.msg.content = []
        this.socket.send(JSON.stringify(this.msg));
      }
    },
    close(){
      console.log("socket已经关闭");
    },
    takeOneStep(i,j){
      console.log("click once");
      if(this.role.turn === true){
        let location = i*10+j;
        console.log(this.$refs);
        if(this.role.color === "black")
          this.$refs.unit[location].turnBlack();
        else if(this.role.color === "white")
          this.$refs.unit[location].turnWhite();
        this.msg.name = "step";
        this.msg.content = [];
        this.msg.content.push(""+location)
        console.log(JSON.stringify((this.msg)));
        this.socket.send(JSON.stringify(this.msg));
        this.role.turn = false;
      }else{
        alert("现在轮到另一玩家落子，请等待");
      }
    }
  },

}
</script>

<style scoped>
#container{
  height: 100%;
  width: 100%;
  margin: auto;
  text-align: center;
}
#navigator{
  height: 60%;
  width: 80%;
  margin: 0 auto;
}
input{
  height: 30px;
}
.menu{
  height: 100%;
  width: 100%;
  border: solid;
  border-radius: 10%;
}
.btn{
  border: solid;
  border-radius: 10%;
  width: 50%;
  height: 25%;
  margin: 50px auto;
  line-height: 120px;
}
.userinfo{
  width: 464px;
  margin:auto;
  padding: 0;
  display: flex;
}
.tipBar{
  height: 40px;
  width: 458px;
  margin:auto;
  padding: 0;
  display: flex;
  border: solid;
  text-align: center;
}
.player{
  font-size: 7px;
  height: 30px;
  width: 116px;
  margin: 0;
  text-align: center;
  padding:0px 5px;
  border: solid;
  display: inline-block;
}
</style>
