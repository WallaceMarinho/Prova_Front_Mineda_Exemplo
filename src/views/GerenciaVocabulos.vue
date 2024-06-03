<template>
    <div>
      <h1>Gerência de Vocábulos</h1>
  
      <!-- Formulário de Cadastro -->
      <form @submit.prevent="addVocabulo">
        <div>
          <label for="termo">Termo:</label>
          <input type="text" v-model="newVocabulo.termo" required>
        </div>
        <div>
          <label for="significado">Significado:</label>
          <input type="text" v-model="newVocabulo.significado" required>
        </div>
        <div>
          <label for="versao">Versão:</label>
          <input type="number" v-model="newVocabulo.versao" required>
        </div>
        <button type="submit">Cadastrar</button>
      </form>
  
      <!-- Formulário de Consulta -->
      <form @submit.prevent="buscarPorTermoEVersao">
        <div>
          <label for="termoConsulta">Termo:</label>
          <input type="text" v-model="termoConsulta" required>
        </div>
        <div>
          <label for="versaoConsulta">Versão:</label>
          <input type="number" v-model.number="versaoConsulta" required>
        </div>
        <button type="submit">Consultar</button>
      </form>
  
      <!-- Tabela de Vocábulos -->
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Termo</th>
            <th>Significado</th>
            <th>Versão</th>
            <th>Situação</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="vocabulo in vocabuloList" :key="vocabulo.id">
            <td>{{ vocabulo.id }}</td>
            <td>{{ vocabulo.termo }}</td>
            <td>{{ vocabulo.significado }}</td>
            <td>{{ vocabulo.versao }}</td>
            <td>{{ vocabulo.situacao }}</td>
          </tr>
          <tr v-if="buscou && vocabuloList.length === 0">
            <td colspan="5">Nenhum vocábulo encontrado para os critérios fornecidos.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        vocabuloList: [],
        newVocabulo: {
          termo: '',
          significado: '',
          versao: ''
        },
        termoConsulta: '',
        versaoConsulta: null,
        buscou: false
      };
    },
    methods: {
      fetchVocabulos() {
        axios.get('http://localhost:8080/vocabulo')
          .then(response => {
            console.log(response)
            this.vocabuloList = response.data.map(vocabulo => {
              return {
                ...vocabulo,
                situacao: vocabulo.dataHoraDesativacao ? 'Desativado' : 'Ativo'
              };
            });
          })
          .catch(error => {
            console.error('Erro ao buscar vocábulos:', error);
          });
      },
      addVocabulo() {
        axios.post('http://localhost:8080/vocabulo', this.newVocabulo)
          .then(() => {
            this.fetchVocabulos();
            this.clearForm();
          })
          .catch(error => {
            console.error('Erro ao cadastrar vocábulo:', error);
          });
      },
      buscarPorTermoEVersao() {
        axios.get(`http://localhost:8080/vocabulo/termo/${this.termoConsulta}/versao/${this.versaoConsulta}`)
          .then(response => {
            if (response.data) {
              this.vocabuloList = response.data.map(vocabulo => {
              return {
                ...vocabulo,
                situacao: vocabulo.dataHoraDesativacao ? 'Desativado' : 'Ativo'
              };
            });
            }
            else {
              this.vocabuloList = []
            }
            this.buscou = true;
            this.clearConsultaForm();
          })
          .catch(error => {
            console.error('Erro ao buscar vocábulos por termo e versão:', error);
          });
      },
      clearForm() {
        this.newVocabulo.termo = '';
        this.newVocabulo.significado = '';
        this.newVocabulo.versao = '';
      },
      clearConsultaForm() {
        this.termoConsulta = '';
        this.versaoConsulta = null;
      }
    },
    created() {
      this.fetchVocabulos();
    }
  };
  </script>
  
  <style>
  /* Adicione estilos conforme necessário */
  form {
    margin-bottom: 20px;
  }
  
  form div {
    margin-bottom: 10px;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
  }
  
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
  }
  
  th {
    background-color: #f2f2f2;
  }
  </style>
  