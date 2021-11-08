<template>
  <form :action="sendMessage" @click.prevent="onSubmit">
    <input v-model="tableID" type="text" >
    <input v-model="userID" type="text" >
    <input v-model="betvol" type="text" >
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
</template>

<script>
let msg = {
  "tableID": "500001",
  "userID": "c63p432n1fdk5k0aeta0",
  "connType": 'JOIN',
  "status": 'WAITING',
  "betvol": 200,
  "greeting": '',
}

export default {
  name: 'App',
  data() {
    return {
      betvol: 50,
      socket: null,
      rcvMessage: "",
      showMsg: true,
      tableID: "",
      userID: "",
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
      msg.tableID = this.tableID
      msg.userID = this.userID
      msg.connType = "ONLINE"
      msg.status = "BETDONE"
      msg.betvol = parseInt(this.betvol)
      this.socket.send(JSON.stringify(msg))
    },
    acceptMsg(evt) {
      let rcvJson
      this.rcvMessage = evt.data
      rcvJson = JSON.parse(evt.data)
      // console.log(rcvJson)
      console.log(rcvJson.tableID, rcvJson.userID)
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
