<template>
  <v-app-bar
      app
      color="teal"
      dark
  >
    <v-btn
        href="https://github.com/"
        target="_blank"
        text
    >
      <span class="mr-2">v0.4</span>
    </v-btn>
    <v-spacer></v-spacer>
    <div>
      <v-toolbar-title><strong>iot bricks <span class="blue--text text--darken-4">ble</span></strong></v-toolbar-title>
    </div>

    <v-spacer></v-spacer>

    <v-btn
        v-if="broker"
        href="http://mqtt.brick.li:8888/status"
        target="_blank"
        class="green"
    >
      <span class="mr-2">Online</span>
    </v-btn>
    <v-btn
        v-if="!broker"
        href="http://mqtt.brick.li:8888/status"
        target="_blank"
        class="red"
    >
      <span class="mr-2">Offline</span>
    </v-btn>
  </v-app-bar>
</template>

<script>
export default {
  props: {
    broker: Boolean
  },
  name: "Navbar"
}
</script>

<style scoped>

</style>
