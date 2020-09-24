<template>
  <v-app>
    <v-app-bar
      color="amber"
      fixed
      short
      tile
      flat
      app
    >
      <v-spacer />
      <v-avatar class="mr-1" size="35">
        <img
          src="icon.png"
          alt="Currency"
        >
      </v-avatar>
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
    <v-bottom-navigation
      app
      horizontal
      color="teal"
    >
      <v-btn
        href="https://nuxtjs.org/"
        target="_blank"
      >
        <span>Nuxt</span>
        <v-icon>mdi-nuxt</v-icon>
      </v-btn>

      <v-btn
        href="https://gleebe.com"
        target="_blank"
      >
        <span class="ml-3">Developer</span>
        <v-avatar size="28">
          <img
            src="https://gleebe.com/images/0.jpg"
            alt="Ronaldo"
          >
        </v-avatar>
      </v-btn>

      <v-btn
        href="https://vuetifyjs.com/"
        target="_blank"
      >
        <span>Vuetify</span>
        <v-icon>mdi-vuetify</v-icon>
      </v-btn>
    </v-bottom-navigation>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      title: 'Currency'
    }
  }
}
</script>
