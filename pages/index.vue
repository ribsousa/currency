<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-flex
      xs12
      sm8
      md6
    >
      <currency :dialog="showModal" @show="update" />
      <conversions :dialog="showModal" @show="update" />
      <install />
    </v-flex>
  </v-layout>
</template>

<script>
import Currency from '~/components/Currency'
import Install from '~/components/Install'
import Conversions from '~/components/Conversions'

export default {
  components: {
    Conversions,
    Currency,
    Install
  },

  data () {
    return {
      showModal: false
    }
  },

  methods: {
    update (showModal) {
      this.showModal = showModal
    }
  }
}
</script>
