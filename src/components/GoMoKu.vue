<template>
  <div id="container">
    <div id="navigator" v-show="!playingWindow" style="margin-top:100px">
      <div class="menu" v-if="navigator">
        <div class="btn" @click="createRoomButton">我要创建房间</div>
        <div class="btn" @click="enterRoomButton">我要加入房间</div>
      </div>
      <div class="menu" v-else-if="choice">
        <h2>{{this.roomNumber?"等待对手进入房间...":"请输入你的昵称"}}</h2>
        <input v-model="nickname" />
        <h3>房间号为：{{ this.roomNumber || "创建房间后获取" }}</h3>
        <div class="btn" @click="initGame('host')">{{this.roomNumber?"点此复制房间号":"创建房间"}}</div>
        <div @click="back" class="backbtn">back</div>
      </div>
      <div class="menu" v-else>
        <h3>请输入你的昵称</h3>
        <input v-model="nickname" />

        <h3>请输入房间号</h3>
        <input
          id="roomNumber"
          v-model="roomNumber"
          placeholder="输入昵称后创建房间即可获得房间号"
        />
        <p @click="initGame('guest')" class="btn">加入房间</p>
        <div @click="back" class="backbtn">back</div>
      </div>
    </div>
    <div v-show="playingWindow" id="boardContainer">
      <div class="userinfo">
        <div class="player" :class="{color1: host}">host</div>
        <div class="player" :class="{color2: host}">{{ playerInfo.host.nickname }}</div>
        <div class="player" :class="{color1: !host}">guest</div>
        <div class="player" :class="{color2: !host}">{{ playerInfo.guest.nickname }}</div>
      </div>
      <div class="tipBar" :class="tipBar=='你的回合！'?'color3':'color4'">
        {{ this.tipBar }}
      </div>
      <!--用事件修饰符来阻止点击，算了也不一定这样，还没想好具体功能毕竟-->
      <!--可以用侦听器watch来搞，内部多加参数就是了-->
      <div v-for="j in rows" :key="j" style="margin-bottom:-3px">
        <tableUnit
          v-for="i in column"
          :row="j"
          :key="i"
          :column="i"
          ref="unit"
          @choose="takeOneStep(j, i)"
        ></tableUnit>
      </div>
    </div>
  </div>
</template>

<script>
import tableUnit from "@/components/tableUnit";
import config from "../config";

export default {
  name: "GoMoKu",
  components: {
    tableUnit,
  },
  data() {
    return {
      host:false,
      socket: null,
      path: "ws://"+config.address,
      msg: {
        sender: "client",
        name: "嘉琳今天吃浩睿",
        content: [],
      },

      playingWindow: false,
      navigator: true,
      choice: true,
      rows: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
      column: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
      tipBar: "游戏开始！",

      nickname: "",
      roomNumber: "",

      role: {
        color: "",
        turn: false,
      },

      playerInfo: {
        host: {
          nickname: "睿睿公主",
        },
        guest: {
          nickname: "嘉琳今天吃什么",
        },
      },
    };
  },
  methods: {
    copyRoomNumber(){
        var save = (e) => {
          e.clipboardData.setData("text/plain", this.roomNumber);
          console.log(this.roomNumber);
          console.log(e.clipboardData);
          e.preventDefault();
        };
        document.addEventListener("copy", save);
        document.execCommand("copy");
        document.removeEventListener("copy", save);
    },
    stateInit() {
      if (this.socket) this.socket.close();
      this.host = false
      this.socket = null;
      this.msg = {
        sender: "client",
        name: "嘉琳今天吃浩睿",
        content: [],
      };
      this.roomNumber = "";
      this.nickname = "";
      this.role = {
        color: "",
        turn: false,
      };
      this.playingWindow = false;
      this.navigator = true;
      this.choice = true;
      this.tipBar = "游戏开始！";
      for(let element in this.$refs.unit){
        this.$refs.unit[element].init()
      }
    },
    back() {
      if (this.socket) this.socket.close();
      this.navigator = true;
    },
    createRoomButton() {
      this.stateInit()
      this.navigator = false;
      this.choice = true;
    },
    enterRoomButton() {
      this.stateInit()
      this.navigator = false;
      this.choice = false;
    },
    initGame(role) {
      if (this.roomNumber&&role === "host") {
        this.copyRoomNumber()
        alert("复制成功!");
        return
      }
      if (this.nickname === "") {
        //    这里得改一下判断
        alert("请先输入昵称");
      } else {
        this.msg.content.push(this.nickname);
        if (role === "host") {
          this.init("/createroom");
        } else if (role === "guest") {
          if (this.roomNumber === "") {
            alert("请输入房间号");
          } else {
            this.init("/enterroom");
          }
        }
      }
    },
    init(role) {
      if (typeof WebSocket === "undefined") {
        alert("您的浏览器不支持socket");
      } else {
        // 实例化socket
        this.socket = new WebSocket(this.path + role);
        // 监听socket连接
        if (role === "/createroom") {
          this.socket.onopen = () => {
            this.host = true
            this.msg.name = "createRoom";
            this.playerInfo.host.nickname = this.nickname;
            console.log("socket连接成功");
            console.log(JSON.stringify(this.msg));
            this.socket.send(JSON.stringify(this.msg));
          };
        } else if (role === "/enterroom") {
          this.socket.onopen = () => {
            this.host = false
            this.msg.name = "enterRoom";
            this.playerInfo.guest.nickname = this.nickname;
            this.msg.content.push(this.roomNumber);
            console.log("socket连接成功");
            this.socket.send(JSON.stringify(this.msg));
          };
        }
        // 监听socket错误信息
        this.socket.onerror = this.error;
        // 监听socket消息
        this.socket.onmessage = this.getMessage;
      }
    },
    error() {
      console.log("连接错误");
    },
    getMessage(msg) {
      console.log("收到" + msg.data);
      let messageBody = JSON.parse(msg.data);
      let msgName = messageBody.name;
      if (msgName === "roomNumber") {
        this.roomNumber = messageBody.content[0];
        this.copyRoomNumber()
        alert(
          "房间创建成功，房间号为\n" +
            this.roomNumber +
            "\n房间号已复制到粘贴板！\n请等待玩家进入房间..."
        );
      }
      // if(msgName === "hostNickname"){
      //   this.playerInfo.host.nickname = messageBody.content[0];
      // }
      if (msgName === "hostStartGame") {
        this.playerInfo.guest.nickname = messageBody.content[0];
        this.role.color = messageBody.content[1];
        this.playingWindow = true;
        if (this.role.color === "black") {
          this.role.turn = true;
          this.tipBar = "你的回合！"
          alert("你是先手");
        } else {
          this.role.turn = false;
          this.tipBar = "正在等待对方落子..."
          alert("你是后手");
        }
      }
      if (msgName === "guestStartGame") {
        let _alert = "已经加入" + messageBody.content[0] + "的房间，开始游戏！"
        this.playerInfo.guest.nickname = this.nickname;
        this.playerInfo.host.nickname = messageBody.content[0];
        this.role.color = messageBody.content[1];
        this.playingWindow = true;
        setTimeout(()=>{
          if (this.role.color === "black") {
            this.role.turn = true;
            _alert+="\n你是先手!"
            this.tipBar = "你的回合！"
            alert(_alert);
          } else {
            this.role.turn = false;
            _alert+="\n你是后手!"
            this.tipBar = "正在等待对方落子..."
            alert(_alert);
          }
        },0)
      }
      if (msgName === "opponentStep") {
        if (messageBody.content[1] === "victory") {
          let x = parseInt(messageBody.content[0]);
          if (this.role.color === "white") this.$refs.unit[x].turnBlack();
          else if (this.role.color === "black") this.$refs.unit[x].turnWhite();
          alert("你输了！");
          this.msg.name = "gameFinish";
          this.msg.content = [];
          this.socket.send(JSON.stringify(this.msg));
          this.gameFinish()
        } else {
          let x = parseInt(messageBody.content[0]);
          if (this.role.color === "white") this.$refs.unit[x].turnBlack();
          else if (this.role.color === "black") {
            this.$refs.unit[x].turnWhite();
          }
          this.role.turn = true;
          this.tipBar = "你的回合！"
        }
      }
      if (msgName === "victory") {
        alert("你赢了！");
        this.msg.name = "gameFinish";
        this.msg.content = [];
        this.socket.send(JSON.stringify(this.msg));
        this.gameFinish()
      }
    },
    close() {
      console.log("socket已经关闭");
    },
    takeOneStep(i, j) {
      console.log("click once");
      if (this.role.turn === true) {
        let location = i * 10 + j;
        console.log(this.$refs);
        if (this.role.color === "black") this.$refs.unit[location].turnBlack();
        else if (this.role.color === "white")
          this.$refs.unit[location].turnWhite();
        this.msg.name = "step";
        this.msg.content = [];
        this.msg.content.push("" + location);
        console.log(JSON.stringify(this.msg));
        this.socket.send(JSON.stringify(this.msg));
        this.role.turn = false;
        this.tipBar = "正在等待对方落子..."
      } else {
        alert("现在轮到另一玩家落子，请等待");
      }
    },
    gameFinish(){
      this.stateInit()
    }
  },
};
</script>

<style scoped>

#boardContainer{
  display: flex;
  flex-direction: column;
  align-items: center;
}

#navigator {
  height: 60%;
  width: 80%;
  margin: 0 auto;
}
input {
  height: 30px;
}
.menu {
  height: 100%;
  width: 100%;
  border: solid;
  border-radius: 10%;
}
.btn {
  border: solid;
  border-radius: 10%;
  width: 50%;
  height: 25%;
  margin: 50px auto;
  line-height: 120px;
  cursor: pointer;
}
.userinfo {
  width: 464px;
  margin: auto;
  padding: 0;
  display: flex;

}
.tipBar {
  height: 40px;
  width: 458px;
  line-height: 40px;
  padding: 0;
  border: solid;
  text-align: center;
  margin-bottom: 2px;
  font-weight: bold;

}
.player {
  font-size: 15px;
  height: 30px;
  width: 116px;
  margin: 0;
  text-align: center;
  padding: 0px 5px;
  border: solid;
  display: inline-block;
  line-height: 30px;
  font-weight: bold;
}

.backbtn {
  cursor: pointer;
  display: inline-block;
  border: 2px solid black;
  padding: 10px 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}
input {
  width: 300px;
}

.color1{
  background-color: #85FFBD;
  background-image: linear-gradient(45deg, #85FFBD 0%, #FFFB7D 100%);
}

.color2{
  background-color: #FFDEE9;
  background-image: linear-gradient(0deg, #FFDEE9 0%, #B5FFFC 100%);
}
.color3{
  background-color: #FFE53B;
  background-image: linear-gradient(45deg, #FFE53B 0%, #FF2525 74%);
}
.color4{
  background-color: #8BC6EC;
  background-image: linear-gradient(135deg, #8BC6EC 0%, #9599E2 100%);
}
</style>
