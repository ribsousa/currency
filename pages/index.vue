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
      <currency />
      <conversions />
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
  }
}
</script>
