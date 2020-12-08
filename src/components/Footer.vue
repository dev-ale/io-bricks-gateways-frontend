<template>
  <v-footer
      dark
      padless
      fixed
  >
    <v-card
        class="flex"
        flat
        tile
    >
      <v-card-title class="teal">
        {{ new Date().getFullYear() }} — FHNW IP5
        <v-spacer></v-spacer>

        <v-btn
            href="https://github.com/"
            target="_blank"
            class="mx-4"
            dark
            icon
        >
          <v-icon size="24px">
            mdi-github
          </v-icon>
        </v-btn>
      </v-card-title>
    </v-card>
  </v-footer>
</template>
