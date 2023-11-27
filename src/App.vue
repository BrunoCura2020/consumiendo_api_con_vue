<script setup>
  import { ref, reactive, onMounted, computed } from 'vue'
  import Alerta from './components/Alerta.vue'

  const monedas = ref([
      { codigo: 'USD', texto: 'Dolar de Estados Unidos'},
      { codigo: 'MXN', texto: 'Peso Mexicano'},
      { codigo: 'EUR', texto: 'Euro'},
      { codigo: 'GBP', texto: 'Libra Esterlina'},
  ])

  const criptomonedas = ref([]);
  const error = ref('');

  const cotizar = reactive({
    moneda: '',
    criptomoneda: ''
  })

  const cotizacion = ref([])

  onMounted(() => {
    const url = 'https://min-api.cryptocompare.com/data/top/mktcapfull?limit=20&tsym=USD'
    fetch(url) 
      .then( respuesta => respuesta.json() )
      .then( ({Data}) => criptomonedas.value = Data  );
  })

  const cotizarCripto = () => {
    if(Object.values(cotizar).includes('') ) {
      error.value = 'Todos los campos son obligatorios'
      return
    }
    error.value = ''
    obtenerCotizacion()
  }

  const obtenerCotizacion = async () => {
    const { moneda, criptomoneda} = cotizar
    const url = `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${criptomoneda}&tsyms=${moneda}`

    const respuesta = await fetch(url);
    const data = await respuesta.json();
    console.log(data.DISPLAY[criptomoneda][moneda]);
    cotizacion.value = data.DISPLAY[criptomoneda][moneda];
  }

  const mostrarResultado = computed(() => {
    return cotizacion.value != ''
  });

</script>

<template>
  <div class="contenedor">
    <h1 class="titulo">Cotizador de <span>Criptomoneda</span></h1>
    <div class="contenido">
      <Alerta v-if="error"> 
        {{ error }}
      </Alerta>
      <form class="formulario" @submit.prevent="cotizarCripto">
        <div class="campo">
          <label for="moneda">Moneda: </label>
          <select id="moneda" v-model="cotizar.moneda">
            <option value="">-- Seleciona</option>
            <option v-for="moneda in monedas" :value="moneda.codigo">{{ moneda.texto }}</option>
          </select>
        </div>

        <div class="campo">
          <label for="moneda">Criptomoneda: </label>
          <select id="moneda" v-model="cotizar.criptomoneda">
            <option value="">-- Seleciona</option>
            <option v-for="criptomoneda in criptomonedas" :value="criptomoneda.CoinInfo.Name">{{ criptomoneda.CoinInfo.FullName }}</option>
          </select>
        </div>

        <input type="submit" value="Cotizar">
      </form>

      <div class="contenedor-resultado">
        <h2 >Cotización</h2>
        <div v-if="mostrarResultado" class="resultado">
          <img :src="'https://cryptocompare.com/' + cotizacion.IMAGEURL" alt="Imagen cripto">
          <div>
            <p>El precio es de: <span>{{ cotizacion.PRICE }}</span></p>
            <p>Precio más alto del día: <span>{{ cotizacion.HIGHDAY }}</span></p>
            <p>Precio más bajo del día: <span>{{ cotizacion.LOWDAY }}</span></p>
            <p>Variación últimas 24 horas: <span>{{ cotizacion.CHANGEPCT24HOUR }}%</span></p>
            <p>Última actualización: <span>{{ cotizacion.LASTUPDATE }}</span></p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>
