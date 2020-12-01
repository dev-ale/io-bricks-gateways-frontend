<template>
  <div>
    <nav class="blue-grey">
      <div class="nav-wrapper">
        <a href="#" class="brand-logo center">iot bricks ble - gateways</a>
      </div>
    </nav>
    <div id="app" style="align-content: center">

      <div style="align-content: center">
        <div class="row">
          <div class="col s12 m6 offset-m3">
            <div v-for="gateway in gateways" :key="gateway.name" class="card blue-grey darken-1 hoverable">
              <div class="card-content white-text">
                <span class="card-title"><span style="font-weight: bold" class="orange-text">{{ gateway.name }}</span></span>
                <div v-if="gateway.loading" class="progress orange accent-2">
                  <div class="indeterminate orange accent-1"></div>
                </div>
                <p v-if="!gateway.loading" style="font-size: x-large">Connected Devices:</p>
                <p v-if="gateway.loading"style="font-size: x-large">Scanning for new Devices</p>
                <ul v-for="device in gateway.connectedDevices" :key="device">
                  <li style="font-size: large">{{ device }}</li>
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
import Publish from './components/Publish'
import Sub1 from './components/Sub1'
import Sub2 from './components/Sub2'
import Sub3 from './components/Sub3'
export default {
  name: 'app',
  data () {
    return {
      gateways: [
        {
          name: 'Gateway-001',
          loading: false,
          connectedDevices: ['Brickli 0000-0005']
        },
        {
          name: 'Gateway-002',
          loading: false,
          connectedDevices: ['Brickli 0000-0001', 'Brickli 0000-0002', 'Brickli 0000-0004', 'Brickli 0000-0005']
        }
      ]
    }
  },
  components: {
    Publish, Sub1, Sub2, Sub3
  },
  methods: {
    scan (gateway) {
      console.log('scan')
      gateway.loading = true
      gateway.connectedDevices = []
      setTimeout(() => {
        gateway.loading = false
        gateway.connectedDevices = [
          'Brickli 0000-0001', 'Brickli 0000-0002', 'Brickli 0000-0004'
        ]
      }, 3000)
    }
  },
  mounted () {
    this.$mqtt.subscribe('VueMqtt/#')
  }
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
