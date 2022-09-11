<template>
  <div class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0">
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.1.2/dist/tailwind.min.css" rel="stylesheet">
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
      <b-row>
        <b-col>
          <p>Data ínicio: <b>{{ startDate }}</b></p>
        </b-col>
        <b-col>
          <p>Data fim: <b>{{ endDate }}</b></p>
        </b-col>
      </b-row>
      <b-row>
        <b-col md="auto">
          <b-calendar v-model="startDate" :max="endDate" @input="findConsolidated" locale="pt-BR"></b-calendar>
        </b-col>

        <b-col md="auto">
          <b-calendar v-model="endDate" :min="startDate" @input="findConsolidated" locale="pt-BR"></b-calendar>
        </b-col>

      </b-row>

      <p>Total gasto: {{ consolidated.totalSpent }}</p>
      <p>Total creditado/estornado: {{ consolidated.totalCredit }}</p>
      <p>Diferença gasto e estorno: {{ consolidated.diffSpentAndCredit }}</p>

      <b-tabs content-class="mt-3" justified>

        <b-tab :title="pair.first" v-for="pair in consolidated.data">
          <p>Total gasto: {{ pair.second.totalSpent }}</p>
          <p>Total creditado/estornado: {{ pair.second.totalCredit }}</p>
          <p>Diferença gasto e estorno: {{ pair.second.diffSpentAndCredit }}</p>

          <b-tabs content-class="mt-3">
            <b-tab title="Gastos">
              <b-table striped hover :items="pair.second.spentMovementList"></b-table>
            </b-tab>
            <b-tab title="Estornos/Créditos">
              <b-table striped hover :items="pair.second.creditMovementList"></b-table>
            </b-tab>
          </b-tabs>

        </b-tab>
      </b-tabs>
    </div>

  </div>
</template>

<script>
const ip = 'http://100.68.116.28:8080'
new Date
export default {
  name: "ConsolidatedPage",
  data() {
    return {
      startDate: this.addMonths(0),
      endDate: this.addMonths(1),
      consolidated: {}
    }
  },
  methods: {
    addMonths(numOfMonths, date = new Date()) {
      date.setMonth(date.getMonth() + numOfMonths);

      return date.toISOString().split('T')[0];
    },
    findConsolidated() {
      fetch(`${ip}/movement/consolidated?` + new URLSearchParams({
        startDate: this.startDate,
        endDate: this.endDate
      }))
        .then(i => i.json())
        .then(consolidated => {
          this.consolidated = consolidated
        })
    }
  },
  created() {
    this.findConsolidated()
  }
}
</script>

<style scoped>

</style>
