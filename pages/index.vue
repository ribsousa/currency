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
      <install />
    </v-flex>
  </v-layout>
</template>

<script>
import Currency from '~/components/Currency'
import Install from '~/components/Install'

export default {
  components: {
    Currency,
    Install
  }
}
</script>
