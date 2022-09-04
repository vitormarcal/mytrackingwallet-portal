<!-- Please remove this file from your project -->
<template>
  <div class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0">
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.1.2/dist/tailwind.min.css" rel="stylesheet">
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
        <div class="flex justify-center pt-8 sm:pt-0">
          <h2  class="text-2xl leading-7 font-semibold">
            <template v-if="this.showInput">
              Adicione um movimento
            </template>
            <template v-else>
              Movimento criado
            </template>
          </h2>
        </div>

      <div class="mt-8 bg-white overflow-hidden shadow sm:rounded-lg p-6">

        <b-form @submit="onSubmit" @reset="onReset" v-if="this.showInput">

          <b-form-group>
            <b-form-input v-model="movement.name" placeholder="Nome do movimento"></b-form-input>
          </b-form-group>
          <b-form-group>
            <b-form-textarea v-model="movement.description" placeholder="Descrição longa do movimento"></b-form-textarea>
          </b-form-group>
          <b-form-group>
            <b-form-input v-model="movement.value" :state="valueState" type="text" placeholder="Valor do movimento"></b-form-input>
          </b-form-group>
          <b-form-group>
            <b-form-input v-model="movement.numberOfInstallments" type="number" placeholder="Quantidade de parcelas"></b-form-input>
          </b-form-group>
          <b-form-group>
            <b-form-group label="Esse movimento é uma entrada ou saída de dinheiro?" v-slot="{ ariaDescribedby }">
              <b-form-radio v-model="movement.credit" :aria-describedby="ariaDescribedby" name="some-radios" value="true">Entrada</b-form-radio>
              <b-form-radio v-model="movement.credit" :aria-describedby="ariaDescribedby" name="some-radios" value="false">Saída</b-form-radio>
            </b-form-group>
          </b-form-group>
          <b-form-group>
            <b-form-select v-model="movement.categoryId" :options="optionsCategories"></b-form-select>
          </b-form-group>

          <b-button type="submit">Adicionar</b-button>
          <b-button type="reset" variant="danger">Limpar</b-button>
        </b-form>

        <div v-else>


            <b-card
              no-body
              style="max-width: 20rem;"
            >
              <template #header>
                <h4 class="mb-0">{{movement.name}} - {{ movement.descriptionCategory }}</h4>
              </template>

              <b-card-body>
                <b-card-sub-title class="mb-2" v-if="movement.numberOfInstallments > 0">Movimento parcelado discriminado abaixo</b-card-sub-title>
                <b-card-text>
                  Foi registrado com sucesso o movimento {{movement.name}} de {{movement.credit ? 'compra' : 'débito' }}
                  no valor de {{movement.value}}.<br/>
                  {{movement.description}}
                </b-card-text>

              </b-card-body>

              <b-list-group flush v-if="movement.numberOfInstallments > 0" v-for="installment in createdMovements" :key="installment.id">
                <b-list-group-item>{{ installment.name }} - Para o mês: {{installment.settleDate}}</b-list-group-item>
              </b-list-group>

              <b-card-body>
                <a href="#" @click="onReset" variant="light" style="color: blueviolet;" class="card-link">Ir para cadastro de novo movimento</a>
              </b-card-body>

            </b-card>
        </div>



      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AddMovement',
  data() {
    return {
      movement: {},
      createdMovements: [],
      optionsCategories: [],
      showInput: true
    }
  },
  computed: {
    valueState() {
      const str = this.movement.value
      if (typeof str != "string") return false // we only process strings!
      return !isNaN(str) && // use type coercion to parse the _entirety_ of the string (`parseFloat` alone does not do this)...
        !isNaN(parseFloat(str))
    }
  },
  methods: {
    onSubmit(event) {
      event.preventDefault()

      const config = {
        method: 'POST',
        body: JSON.stringify(this.movement),
        headers: {'Content-Type': 'application/json'}
      }
      this.movement.descriptionCategory = this.optionsCategories.find(c => c.value === this.movement.categoryId).text
      fetch('http://localhost:8080/movement', config)
        .then(response => response.json())
        .then(movements => {
          this.createdMovements = movements;
          this.showInput = false
        })

    },
    onReset(event) {
      event.preventDefault()
      this.movement = {}
      this.createdMovements = []
      this.showInput = true
    }
  },
  created() {
    fetch('http://localhost:8080/category')
      .then(i => i.json())
      .then(categories => {
        return this.optionsCategories = categories.map(category => {
          return {
            'value': category.id,
            'text': category.name
          }
        });
      })
  }
}
</script>
