<template>
    <div class="about">
      <h1>Gerencie Encomendas</h1>
      <p><label for="rastreio">Rastreio: </label><input id="rastreio" type="text" v-model="encomenda.rastreio" /></p>
      <p><label for="conteudo">Conteúdo: </label><input id="conteudo" type="text" v-model="encomenda.conteudo" /></p>
      <p><label for="dataHoraPrevista">Data e Hora Prevista:</label><input id="dataHoraPrevista" type="datetime-local" v-model="encomenda.dataHoraPrevista" /></p>
      <p><label for="dataHoraEntrega">Data e Hora Entrega: </label><input id="dataHoraEntrega" type="datetime-local" v-model="encomenda.dataHoraEntrega" /></p>
      <button @click="incluir">Incluir</button>
  
      <h2>Consultar Encomendas</h2>
      <p><label for="consultaConteudo">Conteúdo: </label><input id="consultaConteudo" type="text" v-model="consultaConteudo" /></p>
      <p><label for="consultaDataHoraPrevista">Data e Hora Prevista: </label><input id="consultaDataHoraPrevista" type="datetime-local" v-model="consultaDataHoraPrevista" /></p>
      <button @click="buscarEncomendas">Consultar</button>
      <p>{{ erro }}</p>
  
      <div v-if="encomendas.length === 0 && erro">
        <p>{{ erro }}</p>
      </div>
      <table v-else>
        <thead>
          <tr>
            <th>Rastreio</th>
            <th>Conteúdo</th>
            <th>Data e Hora Prevista</th>
            <th>Data e Hora Entrega</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="encomenda in encomendas" :key="encomenda.rastreio">
            <td>{{ encomenda.rastreio }}</td>
            <td>{{ encomenda.conteudo }}</td>
            <td>{{ formatarData(encomenda.dataHoraPrevista) }}</td>
            <td>{{ encomenda.dataHoraEntrega ? formatarData(encomenda.dataHoraEntrega) : 'N/A' }}</td>
            <td>{{ calcularStatus(encomenda) }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="limparConsulta">Mostrar Todos</button>
    </div>
  </template>
  
  <script setup lang="ts">
  import { onMounted, ref } from 'vue';
  import axios from 'axios';
  
  const encomenda = ref({
    rastreio: '',
    conteudo: '',
    dataHoraPrevista: '',
    dataHoraEntrega: '',
  });
  
  const consultaConteudo = ref('');
  const consultaDataHoraPrevista = ref('');
  const encomendas = ref([]);
  const erro = ref('');

  async function incluir() {
  //erro.value = '';
  try {
    await axios.post('/encomenda', encomenda.value)
    console.log(encomenda.value);
    
    limparCampos(); 
    buscarTodasEncomendas(); 
  } catch (e) {
    erro.value = (e as Error).message;
  }
}
  
  function limparCampos() {
    encomenda.value = {
      rastreio: '',
      conteudo: '',
      dataHoraPrevista: '',
      dataHoraEntrega: '',
    };
  }
  
  async function buscarTodasEncomendas() {
    erro.value = '';
    try {
      const response = await axios.get('/encomenda');
      encomendas.value = response.data;
    } catch (e) {
      erro.value = (e as Error).message;
    }
  }
  
  async function buscarEncomendas() {
    erro.value = '';
    if (consultaConteudo.value && consultaDataHoraPrevista.value) {
      try {
        const url = `/encomenda/${consultaConteudo.value}/${new Date(consultaDataHoraPrevista.value).toISOString()}`;
        const response = await axios.get(url);
        encomendas.value = response.data;
  
        if (encomendas.value.length === 0) {
          erro.value = 'Nenhuma encomenda foi encontrada para os critérios fornecidos';
        } else {
          erro.value = '';
        }
      } catch (e) {
        if (axios.isAxiosError(e) && e.response) {
          erro.value = `Erro: ${e.response.status} - ${e.response.data}`;
        } else {
          erro.value = (e as Error).message;
        }
      }
    } else {
      erro.value = 'Por favor, preencha todos os campos para consulta.';
    }
  }
  
  function limparConsulta() {
    erro.value = '';
    buscarTodasEncomendas();
  }
  
  function calcularStatus(encomenda) {
    if (!encomenda.dataHoraEntrega) {
      return "Pendente";
    }
    const dataHoraEntrega = new Date(encomenda.dataHoraEntrega);
    const dataHoraPrevista = new Date(encomenda.dataHoraPrevista);
    if (dataHoraEntrega > dataHoraPrevista) {
      return "Entregue com atraso";
    } else {
      return "Entregue OK";
    }
  }
  
  function formatarData(data: string) {
    const dataObj = new Date(data);
    return dataObj.toLocaleString();
  }
  
  onMounted(() => {
    buscarTodasEncomendas();
  });
  </script>
  