<template>
  <form :action="sendMessage" @click.prevent="onSubmit">
    <label for="tableID">TableID: </label>
    <input v-model="tableID" type="text" > <br>
    <label for="userID">UserID:  </label>
    <input v-model="userID" type="text" > <br>
    <label for="seatID">SeatID:  </label>
    <input v-model="seatID" type="text" > <br>
    <label for="betvol">BetVol:   </label>
    <input v-model="betvol" type="text" > <br><br>
    <input type="submit" value="Send" @click="sendMessage">
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
  "connType": 'JOIN',
  "status": 'WAITING',
  "betvol": 200,
  "greeting": 'Hi',
}

export default {
  name: 'App',
  data() {
    return {
      betvol: 50,
      socket: null,
      rcvMessage: "",
      showMsg: true,
      tableID: 0,
      userID: "c63p432n1fdk5k0aeta1",
      seatID: 0,
      userStatus: ""
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
      msg.connType = "JOIN"
      msg.status = "WAITING"
      this.socket.send(JSON.stringify(msg))
    }
    this.socket.onmessage = (evt) => {
        this.acceptMsg(evt)
    }
  },
  methods: {
    sendMessage() {
      msg.tableID = parseInt(this.tableID) 
      msg.userID = this.userID
      msg.seatID = parseInt(this.seatID)
      msg.connType = "ONLINE"
      msg.status = "BETDONE"
      msg.betvol = parseInt(this.betvol)
      msg.greeting = "Hello!"
      this.socket.send(JSON.stringify(msg))
    },
    acceptMsg(evt) {
      let rcvJson
      this.rcvMessage = evt.data
      try {
        rcvJson = JSON.parse(evt.data)
        // console.log(rcvJson)
        console.log(rcvJson.tableID, rcvJson.userID)
      } catch(e) {
        console.log(e.message)
      }
    },
    WebSocketClose() {
      msg.tableID = parseInt(this.tableID) 
      msg.userID = this.userID
      msg.connType = "CLOSE"
      this.socket.send(JSON.stringify(msg))
      this.socket.close()
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
</style>
