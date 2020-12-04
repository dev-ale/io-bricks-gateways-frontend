
<template>
  <v-app>
    <Navbar :broker = brokerOnline />
    <v-main>

      <v-container fluid class="app">
        <v-row>
          <v-col align="center">
            <div v-if="this.gateways.length === 0">
              <p>no Gateways connected</p>
            </div>
            <div v-if="this.gateways.length > 0">
              <p>Gateways</p>
            </div>
          </v-col>
        </v-row>
        <v-row>
          <v-card
              class="mx-auto mb-6"
              width="300"
              v-for="gateway in gateways"
              :key="gateway.name"

          >
            <v-card-text>
              <p v-if="!gateway.loading" class="green--text">Online </p>
              <p v-if="gateway.loading" class="orange--text">Searching </p>
              <p class="display-1 text--primary">
                {{ gateway.name }}
              </p>
              <p v-if="gateway.connectedDevices.length === 0 &&  !gateway.loading">No connected devices</p>
              <div v-if="!gateway.loading && gateway.connectedDevices.length > 0">
                <p>Connected Devices:</p>
                <v-list
                    v-for="device in gateway.connectedDevices"
                    :key="device"
                    dense
                    disabled
                >
                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon color="primary">mdi-bluetooth</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content style="font-size: large">Brick {{ device }}</v-list-item-content>
                  </v-list-item>
                </v-list>
              </div>
              <div v-if="gateway.loading">
                <p>Scanning for Devices</p>
                <v-progress-linear
                    indeterminate
                    color="blue darken-2"
                ></v-progress-linear>
              </div>

            </v-card-text>
            <v-card-actions>
              <span v-if="gateway.lastScan" class="grey--text">Last Scan: {{ gateway.lastScan }}</span>
              <v-spacer></v-spacer>
              <v-btn
                  dark
                  color="red accent-2"
                  @click="scan(gateway)"
                  v-if="!gateway.loading"
              >
                Scan
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-row>
      </v-container>
      <Footer/>
    </v-main>
  </v-app>
</template>

<script>
/* eslint-disable no-use-before-define */
import Navbar from "@/components/Navbar";
import Footer from "@/components/Footer";

export default {
  name: 'App',

  components: {
    Footer,
    Navbar
  },

  data: () => ({
    client: null,
    brokerOnline: false,
    gateways: []
  }),
  created() {
// Create a client instance
    this.client = new Paho.MQTT.Client('mqtt.brick.li', 9001, 'webclient' + parseInt(Math.random() * 100, 10))
// set callback handlers
    this.client.onConnectionLost = this.onConnectionLost;
    this.client.onMessageArrived = this.onMessageArrived;

// connect the client
    this.client.connect({onSuccess: this.onConnect});

  },
  methods: {
// called when the client loses its connection
    onConnectionLost(responseObject) {
      if (responseObject.errorCode !== 0) {
        console.log('onConnectionLost:' + responseObject.errorMessage)
      }
    },

// called when a message arrives
    onMessageArrived(message) {

      if (message.destinationName.endsWith("/scan")) {
//Scan Topic wurde gepublished
      }

      if (message.destinationName.endsWith("/state")) {
//State Topic wurde gepublished


        let state = JSON.parse(message.payloadString)
        console.log('New State for ' + state.name)
        console.log(state)

        if (this.gateways.some(gateway => gateway.name === state.name)) {

// Check if online --> remove gateway
          console.log(state.online)
          if (state.online === 'false') {
            console.log('offline called')
            let i = this.gateways.findIndex(gateway => gateway.name === state.name);
            this.gateways.splice(i, 1);
            console.log(this.gateways)
          }
// if online --> update values
          else if (state.online === 'true') {
            console.log("matched and updatet gateway: " + state.name + state.online)
            let foundIndex = this.gateways.findIndex(gateway => gateway.name === state.name);
            state.loading = false
            this.$set(this.gateways, foundIndex, state)
//this.gateways[foundIndex+1].loading = false;
          }

// if gateway not in list --> add it
        } else {

          if (state.online === 'false') {
            console.log("nothing to delete")
          }
// if gateway not in list --> add it
          else {
            console.log("new gateway: " + state.name)
            //let j = this.gateways.findIndex(gateway => gateway.name === state.name);
            state.loading = false
            this.gateways.push(state)
          }

        }
      }
    },
    onConnect() {
      console.log('ONCONNECT ausgeführt')
      this.client.subscribe('devices/#')
      this.client.subscribe('gateways/#')
      this.brokerOnline = true

    },
    subscribe(topic) {
      console.log('SUBSCRIBE auf: ' + topic)
      console.log('subscribed to ' + topic)
    },
    publish(message, topic) {
      console.log('PUBLISH auf: ' + topic + ' Message: ' + message)
      let msg = new Paho.MQTT.Message(message);
      msg.destinationName = topic;
      this.client.send(msg);
    },
    scan(gateway) {
      console.log(this.client.name)
      console.log('SCAN ausgeführt')
      let name = gateway.name
      let time = new Date().toJSON().substring(10, 17).replace('T', ' ');
      this.publish(time, 'gateways/' + name + '/scan')
      gateway.loading = true
      console.log(gateway.connectedDevices)
      setTimeout(() => {
        gateway.loading = false
        gateway.lastScan = time
      }, 8000)
    }
  }
}
</script>

<style>
.app {
  font-family: Montserrat, sans-serif;
}
</style>
