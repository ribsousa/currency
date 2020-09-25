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
      <v-skeleton-loader
        :loading="starting"
        class="mx-auto, mt-5 mb-3"
        min-width="300"
        transition="scale-transition"
        type="chip"
      >
        <v-switch
          v-model="themeDark"
          :prepend-icon="themeDark ? 'mdi-lightbulb-off-outline' : 'mdi-lightbulb-on-outline'"
          @click="changeTheme()"
        />
      </v-skeleton-loader>
      <v-skeleton-loader
        :loading="starting"
        class="mx-auto"
        min-width="300"
        transition="scale-transition"
        type="image"
      >
        <v-card
          outlined
          :loading="loading"
          :disabled="disabled"
        >
          <v-toolbar flat dense class="caption">
            <v-spacer />
            <span class="mr-1">
              ({{ fromCurrency.currencyId }}) {{ fromCurrency.alpha3 }}
            </span>
            <gb-flag
              :code="`${fromCurrency.id.toLowerCase()}`"
            />
            <v-btn icon :loading="loading">
              <v-icon
                v-if="currentInput === 'fromCurrency' && loading === false"
                color="success"
                @click="changeCurrency(currentInput, true, true)"
              >
                mdi-arrow-right-thin-circle-outline
              </v-icon>
              <v-icon
                v-if="currentInput === 'toCurrency' && loading === false"
                color="success"
                @click="changeCurrency(currentInput, true, true)"
              >
                mdi-arrow-left-thin-circle-outline
              </v-icon>
            </v-btn>
            <gb-flag
              :code="`${toCurrency.id.toLowerCase()}`"
            />
            <span class="ml-1">
              {{ toCurrency.alpha3 }} ({{ toCurrency.currencyId }})
            </span>
            <v-spacer />
          </v-toolbar>
          <v-card-title class="headline">
            <div>
              <span
                v-if="currentInput === 'fromCurrency'"
                class="subtitle-1"
              >
                {{ formatCurrency(fromCurrencyValue, fromCurrency.currencyId, 1, 6) }}
                {{ fromCurrency.currencyName }} equals
              </span>
              <span
                v-else
                class="subtitle-1"
              >
                {{ formatCurrency(toCurrencyValue, toCurrency.currencyId, 1, 6) }}
                {{ toCurrency.currencyName }} equals
              </span>
              <h4
                v-if="currentInput === 'fromCurrency'"
                class="font-weight-regular"
              >
                {{ formatCurrency(toCurrencyValue, toCurrency.currencyId, 2, 3) }}
                {{ toCurrency.currencyName }}
              </h4>
              <h4
                v-else
                class="font-weight-regular"
              >
                {{ formatCurrency(fromCurrencyValue, fromCurrency.currencyId, 2, 3) }}
                {{ fromCurrency.currencyName }}
              </h4>
              <span class="caption">
                {{ now }} ·
                <a
                  :class="`${themeDark ? 'white--text' : 'black--text'}`"
                  class="apilink"
                  href="https://www.currencyconverterapi.com/"
                >
                  Fontes
                </a>
              </span>
            </div>
          </v-card-title>
          <v-card-text>
            <v-form ref="form">
              <v-container>
                <v-row dense align="center" justify="center">
                  <v-col
                    cols="4"
                  >
                    <v-text-field
                      v-model="fromCurrencyValue"
                      type="number"
                      min="1"
                      step="1"
                      required
                      outlined
                      dense
                      @change="changeCurrency('fromCurrency', true)"
                    />
                  </v-col>
                  <v-col cols="4">
                    <v-autocomplete
                      v-model="fromCurrency"
                      :items="items"
                      item-text="currencyName"
                      item-value="currencyId"
                      return-object
                      outlined
                      dense
                      @change="changeCurrency('fromCurrency', false)"
                    />
                  </v-col>
                </v-row>
                <v-row dense align="center" justify="center">
                  <v-col
                    cols="4"
                  >
                    <v-text-field
                      v-model="toCurrencyValue"
                      type="number"
                      min="1"
                      required
                      outlined
                      dense
                      @change="changeCurrency('toCurrency', true)"
                    />
                  </v-col>
                  <v-col cols="4">
                    <v-autocomplete
                      v-model="toCurrency"
                      :items="items"
                      item-text="currencyName"
                      item-value="currencyId"
                      return-object
                      outlined
                      dense
                      @change="changeCurrency('toCurrency', false)"
                    />
                  </v-col>
                </v-row>
              </v-container>
            </v-form>
            <a
              href="https://www.currencyconverterapi.com/"
              class="caption apilink"
              target="_blanck"
              :class="`${themeDark ? 'white--text' : 'black--text'}`"
            >
              Dados de câmbio disponibilizados por Currency Converter
            </a>
          </v-card-text>
        </v-card>
      </v-skeleton-loader>
      <v-snackbar
        v-model="showError"
        color="error"
        timeout="8000"
        outlined
        center
      >
        Serviço indisponível, volte mais tarde!
        <template v-slot:action="{ attrs }">
          <v-btn
            icon
            v-bind="attrs"
            @click="showError = false"
          >
            <v-icon>mdi-close-circle-outline</v-icon>
          </v-btn>
        </template>
      </v-snackbar>
    </v-flex>
  </v-layout>
</template>
<script>
import axios from 'axios'
import { API_URL, API_KEY } from '~/settings/api'

export default {
  data () {
    return {
      starting: true,
      loading: false,
      fromCurrency: {
        alpha3: 'BRA',
        currencyId: 'BRL',
        currencyName: 'Brazilian real',
        currencySymbol: 'R$',
        id: 'BR',
        name: 'Brazil'
      },
      toCurrency: {
        alpha3: 'USA',
        currencyId: 'USD',
        currencyName: 'United States dollar',
        currencySymbol: '$',
        id: 'US',
        name: 'United States of America'
      },
      fromCurrencyValue: 1,
      toCurrencyValue: 1,
      currentInput: 'fromCurrency',
      items: [],
      errors: null,
      showError: false,
      disabled: false,
      themeDark: false,
      valid: true,
      fromCurrencyValueRules: [
        v => !!v || '',
        v => (v && '^[+-]?[0-9]{1,9}(?:.[0-9]{1,2})?$') || '0'
      ]
    }
  },

  computed: {
    now: () => {
      const options = {
        month: 'long',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        timeZone: 'America/Sao_Paulo',
        timeZoneName: 'short'
      }
      const date = new Date()
      return date.toLocaleDateString('pt-br', options)
    }
  },

  created () {
    this.loadCurrencies()
    this.changeCurrency('fromCurrency', true)
  },

  methods: {
    loadCurrencies () {
      this.starting = true
      axios.get(`${API_URL}countries?apiKey=${API_KEY}`)
        .then((response) => {
          this.generate(response.data.results)
        })
        .catch(error => console.log(error))
        .finally(() => { this.starting = false })
    },

    generate (obj) {
      Object.keys(obj).forEach((key) => {
        this.items.push(obj[key])
      })
    },

    changeCurrency (component, change, reverse = false) {
      if (!this.valid) {
        return false
      }

      if (change) {
        this.currentInput = component
      }

      if (reverse) {
        this.currentInput = (this.currentInput === 'fromCurrency') ? 'toCurrency' : 'fromCurrency'
      }

      let query = null

      switch (component) {
        case 'fromCurrency':
          if (this.currentInput === 'fromCurrency') {
            query = `${this.fromCurrency.currencyId}_${this.toCurrency.currencyId}`
          } else {
            query = `${this.toCurrency.currencyId}_${this.fromCurrency.currencyId}`
          }
          break
        case 'toCurrency':
          if (this.currentInput === 'toCurrency') {
            query = `${this.toCurrency.currencyId}_${this.fromCurrency.currencyId}`
          } else {
            query = `${this.fromCurrency.currencyId}_${this.toCurrency.currencyId}`
          }
          break
      }

      this.converter(component, change, query)
    },

    converter (component, change, query) {
      this.loading = true
      this.disabled = true
      axios.get(`${API_URL}convert?apiKey=${API_KEY}&q=${query}&compact=ultra`)
        .then((response) => {
          let value = 0
          switch (component) {
            case 'fromCurrency':
              if (this.currentInput === 'fromCurrency') {
                value = response.data[query] * this.fromCurrencyValue
                this.toCurrencyValue = this.roundNumber(value, 2)
              } else {
                value = response.data[query] * parseFloat(this.toCurrencyValue)
                this.fromCurrencyValue = this.roundNumber(value, 2)
              }
              break
            case 'toCurrency':
              if (this.currentInput === 'toCurrency') {
                value = response.data[query] * parseFloat(this.toCurrencyValue)
                this.fromCurrencyValue = this.roundNumber(value, 2)
              } else {
                value = response.data[query] * parseFloat(this.fromCurrencyValue)
                this.toCurrencyValue = this.roundNumber(value, 2)
              }
              break
          }
        })
        .catch((error) => {
          this.errors = error
          this.showError = true
        })
        .finally(() => {
          this.loading = false
          this.disabled = false
        })
    },

    formatCurrency (value, currencyId, minimumSignificantDigits = 1, maximumSignificantDigits = 2) {
      return Intl.NumberFormat('pt-BR', {
        style: 'currency',
        currency: currencyId,
        minimumSignificantDigits,
        maximumSignificantDigits
      }).format(value)
    },

    roundNumber (number, decimals) {
      const factorOfTen = Math.pow(10, decimals)
      return Math.round(number * factorOfTen) / factorOfTen
    },

    changeTheme () {
      this.$vuetify.theme.dark = this.themeDark
    },

    validate () {
      this.$refs.form.validate()
    }
  }
}
</script>

<style lang="sass" scoped>
  $input-font-size: 14px !default

  .v-select
    font-size: $input-font-size

  .v-card
    margin-top: 10px

  a.apilink
    text-decoration: none

  a.apilink:hover
    text-decoration: underline
</style>
