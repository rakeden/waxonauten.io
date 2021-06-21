<template>
  <v-row justify="center">
    <v-col cols="12" sm="4" md="4">
      <v-card class="elevation-0" :loading="assetsLoading">
        <v-card-title>
          <h2 class="text-h5">
            Snaking Power calculator
          </h2>
        </v-card-title>
        <v-card-text>
          <p class="d-block caption">
            Unofficial community tool made with &lt;3 and nuxt.js
            Read the <a href="https://www.notion.so/Nova-Rally-Whitepaper-4f3956cb4edf404ea10a8a1f94dfb507" target="_blank">Whitepaper</a>
          </p>

          <v-text-field
            v-model="walletId"
            label="Wallet"
            placeholder="e.g. novarallywax"
            required
            v-on:keyup.enter="fetchSnaking"
          ></v-text-field>

        </v-card-text>
        <v-card-actions>
          <v-btn
            color="warning"
            nuxt
            text
            small
            :disabled="!snakingAssets"
            @click="walletId = null; snakingAssets = null"
          >
            Clear
          </v-btn>
          <v-spacer />
          <v-btn
            color="success"
            nuxt
            text
            @click="fetchSnaking"
            :disabled="!walletId"
          >
            Check now
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
    <v-col
      v-if="snakingAssets"
      cols="12"
      sm="8"
      md="8"
    >
      <v-card class="elevation-0">
        <v-card-title>
          <h2 class="text-h5">
            Snaking Power: <strong>{{ totalSnaking.toLocaleString() }}</strong> 🏁 🐍
          </h2>
        </v-card-title>
        <v-card-text>
          <p class="mb-10">
            Below find your snaking value and more details.
          </p>

          <v-row>
            <v-col cols="12">
              <v-row>
                <v-col cols="6">
                  <h3 class="text-h5 mt-5">Asset Overview</h3>
                  <v-data-table
                    :headers="overviewTableHeaders"
                    :items="overviewTableRows"
                    :hide-default-footer="true"
                  >
                  </v-data-table>
                </v-col>

                <v-col cols="6">
                  <h3 class="text-h5 mt-5">Vehicle-Drivers-Ratio</h3>

                  <VueApexCharts
                    :options="ratioOptions"
                    :series="ratioRows"
                  ></VueApexCharts>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="6">
              <h3 class="text-h5 mt-5">👾 Drivers - <strong>{{ totalSnakingByEntity('drivers') }}</strong></h3>
              <v-data-table
                :headers="driversTableHeaders"
                :items="driversTableRows"
                :hide-default-footer="true"
              >
              </v-data-table>
            </v-col>

            <v-col cols="6">
              <h3 class="text-h5 mt-5">🚗  Vehicles - <strong>{{ totalSnakingByEntity('vehicles') }}</strong></h3>

              <v-data-table
                :headers="vehiclesTableHeaders"
                :items="vehiclesTableRows"
                :hide-default-footer="true"
              >
              </v-data-table>
            </v-col>
            <v-col cols="6">
              <h3 class="text-h5 mt-5">Vehicle Rarities</h3>

              <VueApexCharts
                :options="vehiclesRaritiesOptions"
                :series="vehiclesRaritiesRatio"
              ></VueApexCharts>
            </v-col>
            <v-col cols="6">
              <h3 class="text-h5 mt-5">Drivers Rarities</h3>

              <VueApexCharts
                :options="driversRaritiesOptions"
                :series="driversRaritiesRatio"
              ></VueApexCharts>
            </v-col>
          </v-row>

        </v-card-text>
        <v-card-actions>
          <v-spacer />
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import axios from 'axios';

const RARITYVALUE = {
  'Common': 1000,
  'Uncommon': 3500,
  'Rare': 12500,
  'Classic': 35000,
  'Sketch': 100000,
}

export default {
  components: {
    VueApexCharts: () => import('vue-apexcharts')
  },
  data() {
    return {
      walletId: 'rakedenxriva',
      snakingAssets: null,
      totalSnaking: null,
      assetsLoading: false,

      series: [44, 55, 13, 43, 22],
    }
  },
  watch: {
    snakingAssets(assets) {
      if (!assets) {
        this.totalSnaking = 0
        return
      }

      this.totalSnaking = assets
        .filter(asset => asset.schema.schema_name !== 'packs')
        .map((asset) => RARITYVALUE[asset.data.Rarity])
        .reduce((a, b) => (a + b), 0)
    }
  },
  computed: {
    overviewTableHeaders() {
      return [{
        text: 'Vehicles',
        value: 'vehicles',
        sortable: false,
      },{
        text: 'Drivers',
        value: 'drivers',
        sortable: false,
      },{
        text: 'Packs',
        value: 'packs',
        sortable: false,
      }]
    },
    overviewTableRows() {
      return [{
        vehicles: this.getTotalVehicles(),
        drivers: this.getTotalDrivers(),
        packs: this.getTotalPacks(),
      }]
    },
    driversTableHeaders() {
      return [{
        text: 'Rarity',
        value: 'rarity',
        sortable: false,
      },{
        text: 'Count',
        value: 'count',
        sortable: false,
      },{
        text: 'Snaking Value',
        value: 'snaking',
        sortable: false,
      }]
    },
    driversTableRows() {
      const rarities = Object.keys(RARITYVALUE)

      const allDrivers = this.snakingAssets
          .filter(asset => asset.schema.schema_name === 'drivers')

      return rarities.map(rarity => {
        return {
          rarity,
          count: allDrivers ? allDrivers.filter(asset => asset.data.Rarity === rarity).length : 0,
          snaking: allDrivers ?
            allDrivers
              .filter(asset => asset.data.Rarity === rarity)
              .map(asset => RARITYVALUE[rarity])
              .reduce((a, b) => (a + b), 0)
              .toLocaleString()
            : 0
        }
      })
    },
    vehiclesTableHeaders() {
      return [{
        text: 'Rarity',
        value: 'rarity',
        sortable: false,
      },{
        text: 'Count',
        value: 'count',
        sortable: false,
      },{
        text: 'Snaking Value',
        value: 'snaking',
        sortable: false,
      }]
    },
    vehiclesTableRows() {
      const rarities = Object.keys(RARITYVALUE)

      const allVehicles = this.snakingAssets
          .filter(asset => asset.schema.schema_name === 'vehicles')

      return rarities.map(rarity => {
        return {
          rarity,
          count: allVehicles ? allVehicles.filter(asset => asset.data.Rarity === rarity).length : 0,
          snaking: allVehicles ?
            allVehicles
              .filter(asset => asset.data.Rarity === rarity)
              .map(asset => RARITYVALUE[rarity])
              .reduce((a, b) => (a + b), 0)
              .toLocaleString()
            : 0
        }
      })
    },
    ratioOptions() {
      return {
        chart: {
          width: 380,
          type: 'pie',
        },
        stroke: {
          show: false,
          width:0
        },
        colors: ['#BDC3C7', '#3498DB', '#27AE60', '#E74C3C', '#EA4C88'],
        labels: ['Vehicles', 'Drivers'],
        responsive: [{
          breakpoint: 480,
          options: {
            chart: {
              width: 200
            },
            legend: {
              position: 'bottom'
            }
          }
        }]
      }
    },
    ratioRows() {
      const total = this.getTotalSnakingAssets();

      if(!total)
        return [0, 0]

      return [
        this.getTotalVehicles(),
        this.getTotalDrivers()
      ];
    },
    vehiclesRaritiesOptions() {
      return {
        chart: {
          width: 380,
          type: 'pie',
        },
        stroke: {
          show: false,
          width:0
        },
        colors: ['#BDC3C7', '#3498DB', '#27AE60', '#E74C3C', '#EA4C88'],
        labels: Object.keys(RARITYVALUE),
        responsive: [{
          breakpoint: 480,
          options: {
            chart: {
              width: 200
            },
            legend: {
              position: 'bottom'
            }
          }
        }]
      }
    },
    vehiclesRaritiesRatio() {
      const allRelevantAssets = this.snakingAssets.filter(asset => asset.schema.schema_name === 'vehicles');

      const total = this.getTotalSnakingAssets();

      if(!total)
        return [0, 0, 0, 0, 0]

      return [
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Common').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Uncommon').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Rare').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Classic').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Sketch').length,
      ]
    },
    driversRaritiesOptions() {
      return {
        chart: {
          width: 380,
          type: 'pie',
        },
        stroke: {
          show: false,
          width:0
        },
        colors: ['#BDC3C7', '#3498DB', '#27AE60', '#E74C3C', '#EA4C88'],
        labels: Object.keys(RARITYVALUE),
        responsive: [{
          breakpoint: 480,
          options: {
            chart: {
              width: 200,
            },
            legend: {
              position: 'bottom'
            },
          }
        }]
      }
    },
    driversRaritiesRatio() {
      const allRelevantAssets = this.snakingAssets.filter(asset => asset.schema.schema_name === 'drivers');

      const total = this.getTotalSnakingAssets();

      if(!total)
        return [0, 0, 0, 0, 0]

      return [
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Common').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Uncommon').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Rare').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Classic').length,
        allRelevantAssets.filter(asset => asset.data.Rarity === 'Sketch').length,
      ]
    },
  },
  methods: {
    fetchSnaking() {
      if (!this.walletId)
        return

      this.assetsLoading = true

      const url = encodeURI(`https://wax.api.atomicassets.io/atomicassets/v1/assets?page=1&limit=9999&owner=${this.walletId}&collection_name=novarallywax`)

      axios.get(url)
        .then(({data}) => {
          this.snakingAssets = data.data
          this.assetsLoading = false
        })
        .catch((err) => console.log(err))
    },
    totalSnakingByEntity(entity) {
      const allAssets = this.snakingAssets
          .filter(asset => asset.schema.schema_name === entity)

      if (!allAssets)
        return 0

      return allAssets
        .map((asset) => {
          if(asset.schema.schema_name !== 'packs') {
            return RARITYVALUE[asset.data.Rarity]
          }
        })
        .filter(asset => asset !== undefined)
        .reduce((a, b) => (a + b), 0)
        .toLocaleString()
    },
    getTotalSnakingAssets() {
        return this.snakingAssets.filter(asset => asset.schema.schema_name !== 'packs').length || 0
    },
    getTotalDrivers() {
        return this.snakingAssets.filter(asset => asset.schema.schema_name === 'drivers').length || 0
    },
    getTotalVehicles() {
        return this.snakingAssets.filter(asset => asset.schema.schema_name === 'vehicles').length || 0
    },
    getTotalPacks() {
        return this.snakingAssets.filter(asset => asset.schema.schema_name === 'packs').length || 0
    },
  },
}
</script>
