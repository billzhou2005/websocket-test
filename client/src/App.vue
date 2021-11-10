
<template>
  <div class="join">
    <button class="joinButton" @click="joinMessage(players[0])">Player1 Join</button>
    <button class="joinButton" style="margin-left:10px;" @click="joinMessage(players[1])">Player2 Join</button>
    <button class="joinButton" style="margin-left:10px;" @click="joinMessage(players[2])">Player3 Join</button>
  </div>
  <div id="counter" style="color:red;">
    Counter: {{ counter }}
  </div>

  <form :action="sendMessage" @click.prevent="onSubmit">
    <label for="userID">UserID:  </label>
    <input v-model="players[0].userID" type="text" >
    <label for="seatID">SeatID:  </label>
    <input v-model="players[0].seatID" type="text" >
    <label for="betvol">BetVol:   </label>
    <input v-model="players[0].betvol" type="text" > <br><br>
    <input :disabled="isdisabled" type="submit" value="Player1 Send" @click="sendMessage(players[0])"> <br><br>
  </form>
  <form :action="sendMessage" @click.prevent="onSubmit">
    <label for="userID">UserID:  </label>
    <input v-model="players[1].userID" type="text" >
    <label for="seatID">SeatID:  </label>
    <input v-model="players[1].seatID" type="text" >
    <label for="betvol">BetVol:   </label>
    <input v-model="players[1].betvol" type="text" > <br><br>
    <input type="submit" value="Player2 Send" @click="sendMessage(players[1])"> <br><br>
  </form>
  <form :action="sendMessage" @click.prevent="onSubmit">
    <label for="userID">UserID:  </label>
    <input v-model="players[2].userID" type="text" >
    <label for="seatID">SeatID:  </label>
    <input v-model="players[2].seatID" type="text" >
    <label for="betvol">BetVol:   </label>
    <input v-model="players[2].betvol" type="text" > <br><br>
    <input type="submit" value="Player3 Send" @click="sendMessage(players[2])"> <br><br>
  </form>
  <!-- <p>
    Two way data binding is fun!
  </p>
  <p>
    {{ message }}
  </p> -->
  <div v-if="showMsg">
    <h3>Message in a WebSocket</h3>
    <p>
      {{ rcvMessage }}
    </p>
  </div>
  <div class="closews">
    <button @click="WebSocketClose">CloseWS</button>
  </div>
</template>

<script>
let msg = {
  "tableID": 0,
  "userID": "c63p432n1fdk5k0aeta0",
  "seatID": 0,
  "connType": 'JOINED',
  "status": 'WAITING',
  "betvol": 0,
  "greeting": 'Hi',
}

export default {
  name: 'App',
  data() {
    return {
      players: [
        {"userID": "c63p432n1fdk5k0aeta1", "seatID":1, "betvol":120},
        {"userID": "c63p432n1fdk5k0aeta2", "seatID":2, "betvol":120},
        {"userID": "c63p432n1fdk5k0aeta3", "seatID":3, "betvol":120},
      ],
      betvol: 50,
      socket: null,
      rcvMessage: "",
      showMsg: true,
      tableID: 0,
      userID: "c63p432n1fdk5k0aeta1",
      seatID: 0,
      userStatus: "",
      counter: 0,
      isdisabled: true
    }
  },
  mounted() {
    this.socket = new WebSocket("ws://140.143.149.188:9080/ws")
    this.socket.onclose = () => {
      console.log("Connection closed")
    }
    this.socket.onerror = () => {
      console.log("Connection error")
    }
    this.socket.onopen = () => {
      console.log("Connection success")
      msg.connType = "JOINED"
      msg.status = "WAITING"
      this.socket.send(JSON.stringify(msg))
    }
    this.socket.onmessage = (evt) => {
        this.acceptMsg(evt)
    }
    
    setInterval(() => {
      if (this.counter > 0) {
        this.counter--
        if(this.counter === 0 && this.isdisabled == false) {
          this.isdisabled = true
          msg.userID = this.players[0].userID
          msg.seatID = this.players[0].seatID
          msg.betvol = 0
          msg.connType = "TIMEOUT"
          msg.status = "BDONE"
          this.socket.send(JSON.stringify(msg))
        }
      } 
    }, 1000)
  },
  methods: {
    sendMessage(player) {
      msg.tableID = parseInt(this.tableID) 
      msg.userID = player.userID
      msg.seatID = parseInt(player.seatID)
      msg.connType = "PLAYING"
      msg.status = "BNEXT"
      msg.betvol = parseInt(player.betvol)
      msg.greeting = "test!"
      this.socket.send(JSON.stringify(msg))
      if(player.seatID ===1 ) {
        this.counter = 0
        this.isdisabled = true
      }
    },
    acceptMsg(evt) {
      let rcvJson
      this.rcvMessage = evt.data
      try {
        rcvJson = JSON.parse(evt.data)
        console.log(rcvJson.userID, rcvJson.status, rcvJson.betvol)
        if(rcvJson.seatID === 3) {
          if(rcvJson.status == "BDONE" || rcvJson.status == "BNEXT") {
            this.isdisabled = false
            this.counter = 10
          }
        } 
      } catch(e) {
        console.log("error message", e.message)
      }
    },
    WebSocketClose() {
      msg.tableID = parseInt(this.tableID) 
      msg.userID = this.userID
      msg.connType = "CLOSE"
      this.socket.send(JSON.stringify(msg))
      this.socket.close()
    },
    joinMessage(player) {
      msg.userID = player.userID
      msg.seatID = player.seatID
      msg.connType = "JOINED"
      msg.status = "WAITING"
      msg.betvol = 0
      this.socket.send(JSON.stringify(msg))
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
div {
  padding-top: 15px;
  padding-bottom: 15px;
}

.joinButton {
  color:blue;
}

</style>
