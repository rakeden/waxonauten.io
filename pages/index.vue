<template>
  <v-row justify="center">
    <v-col cols="4" sm="4" md="4">
      <v-card class="elevation-0" :loading="assetsLoading">
        <v-card-title>
          <h2 class="text-h5">
            Snaking Power calculator
          </h2>
        </v-card-title>
        <v-card-text>
          <p class="d-block caption">
            Unofficial community tool made with &lt;3 and nuxt.js
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
      cols="8"
      sm="8"
      md="8"
    >
      <v-card class="elevation-0">
        <v-card-title>
          <h2 class="text-h5">
            Snaking Power: <strong>{{ totalSnaking.toLocaleString() }}</strong> 🐍
          </h2>
        </v-card-title>
        <v-card-text>
          <p class="mb-10">
            Below find your snaking value and more details.
          </p>

          <h3>Asset Overview</h3>

          <v-data-table
            :headers="overviewTableHeaders"
            :items="overviewTableRows"
            :hide-default-footer="true"
          >
          </v-data-table>


          <h3 class="mt-5">Drivers Overview ({{ totalSnakingByEntity('drivers') }})</h3>

          <v-data-table
            :headers="driversTableHeaders"
            :items="driversTableRows"
            :hide-default-footer="true"
          >
          </v-data-table>


          <h3 class="mt-5">Vehicles Overview ({{ totalSnakingByEntity('vehicles') }})</h3>

          <v-data-table
            :headers="vehiclesTableHeaders"
            :items="vehiclesTableRows"
            :hide-default-footer="true"
          >
          </v-data-table>

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
  data() {
    return {
      walletId: 'rakedenxriva',
      snakingAssets: null,
      totalSnaking: null,
      assetsLoading: false,
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
        console.log(rarity)
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
        console.log(rarity)
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
  },
  methods: {
    fetchSnaking() {
      if (!this.walletId)
        return

      this.assetsLoading = true

      const url = encodeURI(`https://wax.api.atomicassets.io/atomicassets/v1/assets?page=1&limit=100000&owner=${encodeURIComponent(this.walletId).replace(/\./g, '%2E')}&collection_name=novarallywax`)

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
