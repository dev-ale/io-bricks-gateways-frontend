<template>
  <div>
    <nav class="blue-grey">
      <div class="nav-wrapper">
        <a href="#" class="brand-logo center">iot bricks ble - gateways<span v-if="!brokerOnline" class="badge red black-text">Offline</span></a>
      </div>
    </nav>

    <div id="app">
        <div class="row">
          <div class="col s12 m6 offset-m3">
            <h2 v-if="this.gateways.length === 0">no gateways available</h2>
            <div v-if="this.gateways.length > 0">
              <div v-for="gateway in gateways" :key="gateway.name" class="card blue-grey darken-1 hoverable">
                <div class="card-content white-text">
                  <span class="card-title"><span style="font-weight: bold" class="orange-text">{{ gateway.name }}</span></span>
                  <div v-if="gateway.loading" class="progress orange accent-2">
                    <div class="indeterminate orange accent-1"></div>
                  </div>
                  <p v-if="!gateway.loading" style="font-size: x-large">Connected Devices:</p>
                  <p v-if="gateway.loading"style="font-size: x-large">Scanning for new Devices</p>
                  <ul v-if="!gateway.loading" v-for="device in gateway.connectedDevices" :key="device">
                    <li style="font-size: large">Brickli {{ device }}</li>
                  </ul>
                </div>
                <div class="card-action">
                  <a @click="scan(gateway)" class="waves-effect waves-light btn orange black-text">Scan for new Devices</a>
                </div>
              </div>
            </div>

          </div>
        </div>
      </div>
    </div>

</template>

<script>

export default {
  name: 'app',
  data () {
    return {
      client: null,
      brokerOnline: false,
      gateways: []
    }
  },
  created () {
// Create a client instance
    this.client = new Paho.MQTT.Client('mqtt.brick.li', 9001, 'clientId')

// set callback handlers
    this.client.onConnectionLost = this.onConnectionLost;
    this.client.onMessageArrived = this.onMessageArrived;

// connect the client
    this.client.connect ({onSuccess:this.onConnect});

  },
  methods: {
    // called when the client loses its connection
    onConnectionLost (responseObject) {
      if (responseObject.errorCode !== 0) {
      console.log('onConnectionLost:' + responseObject.errorMessage)
      }
    },

    // called when a message arrives
    onMessageArrived (message) {
    if (message.destinationName.endsWith("/scan")) {
      //console.log(message.destinationName)
      //console.log(message.payloadString)
    }
    if (message.destinationName.endsWith("/state")) {

      console.log('neuer state')
      let obj = JSON.parse(message.payloadString)
      console.log(obj)

      if(this.gateways.some(gateway => gateway.name === obj.name)) {

        // Check if online --> remove gateway
        if (obj.online === false) {
          console.log('offline called')
          let i = this.gateways.findIndex(gateway => gateway.name === obj.name);
          this.gateways.splice(i, 1);
          console.log(this.gateways)
        }
        // if online --> update values
        else {
          console.log("matched and updatet gateway: " + obj.name)
          let foundIndex = this.gateways.findIndex(gateway => gateway.name === obj.name);
          obj.loading = false
          this.$set(this.gateways, foundIndex, obj)
          //this.gateways[foundIndex+1].loading = false;
        }

      // if gateway not in list --> add it
      }else {
        console.log("new gateway: " + obj.name)
        let j = this.gateways.findIndex(gateway => gateway.name === obj.name);
        obj.loading = false
        this.gateways.push(obj)
        //this.gateways[j+1].loading = false;
        //this.$set(this.gateways, this.gateways.length + 1, obj)
      }
    }
    else {

    }
  },

    onConnect () {
      console.log('ONCONNECT ausgeführt')
      this.client.subscribe('devices/#')
      this.client.subscribe('gateways/#')
      this.brokerOnline = true
    },
    subscribe (topic) {
      console.log('SUBSCRIBE auf: ' + topic)
      console.log('subscribed to ' + topic)
    },
    publish (message, topic) {
      console.log('PUBLISH auf: ' + topic + ' Message: ' + message)
      let msg = new Paho.MQTT.Message(message);
      msg.destinationName = topic;
      this.client.send(msg);
    },
    scan (gateway) {
      console.log('SCAN ausgeführt')
      let name = gateway.name
      let time = new Date().toJSON().substring(10,19).replace('T',' ');
      this.publish(time,'gateways/' + name + '/scan')
      gateway.loading = true
      setTimeout(() => {
        gateway.loading = false
      }, 8000)
    }
  },
}
</script>

<style>
#app {
  font-family: Montserrat, serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
button {
  padding: 10px 20px;
}
</style>
