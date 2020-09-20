<template>
  <div
    class="text-center"
  >
    <v-bottom-sheet
      v-if="deferredPrompt"
      v-model="sheet"
      max-width="480"
      hide-overlay
      open-on-focus
    >
      <v-sheet
        class="text-center"
        height="64px"
      >
        <v-toolbar
          flat
          outlined
        >
          <v-btn
            icon
            color="success"
            @click="promptInstall()"
          >
            <v-icon>mdi-checkbox-marked-circle-outline</v-icon>
          </v-btn>

          <v-toolbar-title
            class="overline"
          >
            Instal this app
          </v-toolbar-title>

          <v-spacer />

          <v-btn
            icon
            @click="cancelInstall()"
          >
            <v-icon>mdi-progress-close</v-icon>
          </v-btn>
        </v-toolbar>
      </v-sheet>
    </v-bottom-sheet>
  </div>
</template>

<script>
export default {
  data: () => ({
    sheet: false,
    deferredPrompt: undefined
  }),

  created () {
    this.show()

    this.$on('canInstall', (e) => {
      e.preventDefault()
      this.deferredPrompt = e
    })
  },

  methods: {
    show () {
      this.sheet = true
    },

    promptInstall () {
      this.deferredPrompt.prompt()

      this.deferredPrompt.userChoice.then((choiceResult) => {
        if (choiceResult.outcome === 'accepted') {
          console.log('User accepted the install prompt')
        } else {
          this.sheet = !this.sheet
        }

        this.deferredPrompt = null
      })
    },

    cancelInstall () {
      this.deferredPrompt = undefined
      this.sheet = !this.sheet
    }
  }
}
</script>
