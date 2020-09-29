<template>
  <v-skeleton-loader
    :loading="starting"
    class="mx-auto, mt-5 mb-3"
    min-width="300"
    transition="scale-transition"
    type="chip"
  >
    <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
      <template v-slot:activator="{ on, attrs }">
        <v-badge
          bordered
          overlap
          mode="in-out"
          :content="amount"
          color="error"
          icon="mdi-lock"
        >
          <span class="caption">Minhas conversões</span>
          <v-btn
            icon
            color="success"
            v-bind="attrs"
            v-on="on"
            @click="getConversions()"
          >
            <v-icon>mdi-sack</v-icon>
          </v-btn>
        </v-badge>
      </template>
      <v-card>
        <v-toolbar dark short flat tile color="teal">
          <v-badge
            inline
            mode="in-out"
            :content="amount"
            color="error"
            icon="mdi-lock"
          >
            <v-icon
              @click="dialog = false"
            >
              mdi-sack
            </v-icon>
          </v-badge>
          <v-toolbar-title
            class="subtitle-1"
          >
            Conversões
          </v-toolbar-title>
          <v-spacer />
          <v-toolbar-items>
            <v-btn
              dark
              icon
              @click="dialog = false"
            >
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </v-toolbar-items>
        </v-toolbar>
        <v-container>
          <v-card outlined tile class="mt-5 pl-2 pr-2">
            <v-toolbar tile flat dense>
              My coversions
              <v-divider vertical inset class="mx-3" />
              <v-icon
                v-if="selected.length"
                @click="confirmShow()"
              >
                mdi-delete
              </v-icon>
              <v-progress-circular
                v-if="selected.length"
                :size="40"
                :width="3"
                :value="(selected.length / amount) * 100"
                color="error"
              >
                {{ selected.length }}
              </v-progress-circular>
              <v-spacer />
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
              />
            </v-toolbar>
            <v-data-table
              v-model="selected"
              :headers="headers"
              :items="conversions"
              :search="search"
              class="elevation-0"
              locale="pt-BR"
              show-select
            >
              <template v-slot:item.fromCurrencyValue="{ item }">
                <v-icon left color="success">
                  mdi-cash-multiple
                </v-icon>
                <span v-if="item.currentInput === 'fromCurrency'">{{ formatCurrency(item.fromCurrencyValue, item.fromCurrencyId, 3, 6) }}</span>
                <span v-else>{{ formatCurrency(item.toCurrencyValue, item.toCurrencyId, 3, 6) }}</span>
              </template>
              <template v-slot:item.fromCurrencyId="{ item }">
                <v-tooltip top color="orange">
                  <template v-slot:activator="{ on, attrs }">
                    <v-avatar
                      size="22"
                      tile
                      v-bind="attrs"
                      v-on="on"
                    >
                      <v-img v-if="item.currentInput === 'fromCurrency'" :src="`flags/${item.fromCurrencyCountryId.toLowerCase()}.svg`" />
                      <v-img v-else :src="`flags/${item.toCurrencyCountryId.toLowerCase()}.svg`" />
                    </v-avatar>
                    {{ item.currentInput === 'fromCurrency' ? item.fromCurrencyId : item.toCurrencyId }}
                  </template>
                  <span>
                    {{ item.currentInput === 'fromCurrency' ? item.fromCurrencyName : item.toCurrencyName }}
                  </span>
                </v-tooltip>
              </template>
              <template v-slot:item.toCurrencyId="{ item }">
                <v-tooltip top color="orange">
                  <template v-slot:activator="{ on, attrs }">
                    <v-avatar
                      size="22"
                      tile
                      v-bind="attrs"
                      v-on="on"
                    >
                      <v-img v-if="item.currentInput === 'fromCurrency'" :src="`flags/${item.toCurrencyCountryId.toLowerCase()}.svg`" />
                      <v-img v-else :src="`flags/${item.fromCurrencyCountryId.toLowerCase()}.svg`" />
                    </v-avatar>
                    {{ item.currentInput === 'fromCurrency' ? item.toCurrencyId : item.fromCurrencyId }}
                  </template>
                    <span>
                      {{ item.currentInput === 'fromCurrency' ? item.toCurrencyName : item.fromCurrencyName }}
                    </span>
                </v-tooltip>
              </template>
              <template v-slot:item.toCurrencyValue="{ item }">
                <v-icon left color="success">
                  mdi-cash-multiple
                </v-icon>
                <span v-if="item.currentInput === 'fromCurrency'">{{ formatCurrency(item.toCurrencyValue, item.toCurrencyId, 2, 2) }}</span>
                <span v-else>{{ formatCurrency(item.fromCurrencyValue, item.fromCurrencyId, 2, 4) }}</span>
              </template>
              <template v-slot:item.convertedAt="{ item }">
                <v-icon left color="primary">
                  mdi-update
                </v-icon>
                {{ item.convertedAt }}
              </template>
              <template v-slot:item.actions="{ item }">
                <v-icon
                  @click="confirmShow(item)"
                >
                  mdi-delete-sweep-outline
                </v-icon>
              </template>
            </v-data-table>
          </v-card>
          <v-snackbar
            v-model="confirmDestroy"
            outlined
            timeout="-1"
            dark
          >
            Excluir {{ selected.length }} {{ selected.length > 1 ? 'selecionados' : 'selecionado' }}?
            <template v-slot:action="{ attrs }">
              <v-btn
                color="indigo"
                icon
                v-bind="attrs"
                @click="destroy()"
              >
                <v-icon
                  color="success"
                >
                  mdi-check
                </v-icon>
              </v-btn>
              <v-btn
                color="indigo"
                icon
                v-bind="attrs"
                @click="confirmDestroy = false, selected = []"
              >
                <v-icon
                  color="error"
                >
                  mdi-close
                </v-icon>
              </v-btn>
            </template>
          </v-snackbar>
        </v-container>
      </v-card>
    </v-dialog>
  </v-skeleton-loader>
</template>

<script>
export default {
  data () {
    return {
      confirmDestroy: false,
      search: '',
      starting: true,
      dialog: false,
      notifications: false,
      sound: true,
      widgets: false,
      amount: 0,
      conversions: [],
      selected: [],
      value: 0,
      headers: [
        { text: 'Coverter', value: 'fromCurrencyValue' },
        {
          text: 'From Currency',
          align: 'start',
          value: 'fromCurrencyId'
        },
        { text: 'To Currency', value: 'toCurrencyId' },
        { text: 'Coverted', value: 'toCurrencyValue' },
        { text: 'Converted At', value: 'convertedAt' },
        { text: 'Actions', value: 'actions', sortable: false }
      ]
    }
  },

  created () {
    this.getConversions()
  },

  mounted () {
    this.starting = false
  },

  methods: {
    getConversions () {
      if (process.browser) {
        const conversions = JSON.parse(localStorage.getItem('conversions'))
        this.conversions = conversions
        this.amount = conversions.length
      }
    },

    formatCurrency (value, currencyId, minimumSignificantDigits = 1, maximumSignificantDigits = 2) {
      return Intl.NumberFormat('pt-BR', {
        style: 'currency',
        currency: currencyId,
        minimumSignificantDigits,
        maximumSignificantDigits
      }).format(value)
    },

    deleteItem (item) {
      if (process.browser) {
        const conversions = this.conversions.filter(conversion => conversion.id !== item.id)
        this.conversions = conversions
        this.amount = conversions.length
        localStorage.setItem('conversions', JSON.stringify(conversions))
      }
    },

    confirmShow (item = null) {
      this.confirmDestroy = true
      if (item) {
        this.selected.push(item)
      }
    },

    destroy () {
      if (process.browser) {
        this.selected.forEach((selected, index) => {
          this.deleteItem(selected)
        })
        this.selected = []
        this.confirmDestroy = false
      }
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  margin: 1rem
</style>
