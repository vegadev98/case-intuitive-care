<template>
  <div>
    <!-- Navbar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
      <div class="container">
        <a class="navbar-brand" href="#">Dashboard Operadoras</a>
      </div>
    </nav>

    <div class="container mt-5">
      <h2 class="text-center mb-4">📊 Maiores Despesas em "EVENTOS/ SINISTROS CONHECIDOS OU
        AVISADOS DE ASSISTÊNCIA A SAÚDE MEDICO HOSPITALAR" — Último Trimestre</h2>

      <!-- Carregando -->
      <div v-if="loading" class="text-center my-4">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Carregando...</span>
        </div>
      </div>

      <!-- Conteúdo -->
      <div v-else>
        <!-- Erro -->
        <div v-if="erro" class="alert alert-danger" role="alert">
          {{ erro }}
        </div>

        <!-- Tabela Trimestre -->
        <div v-else>
          <div class="d-flex justify-content-end mb-2">
            <button class="btn btn-success" @click="exportarParaExcel(trimestre, 'Top_Operadoras_Trimestre')">
              Exportar Trimestre
            </button>
          </div>

          <table class="table table-bordered table-striped text-center">
            <thead class="table-dark">
              <tr>
                <th>REG ANS</th>
                <th>Data</th>
                <th>Despesa (R$)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in trimestre" :key="item.REG_ANS">
                <td>{{ item.REG_ANS }}</td>
                <td>{{ item.DATA }}</td>
                <td class="text-end">{{ formatCurrency(item.VL_SALDO_FINAL) }}</td>
              </tr>
            </tbody>
          </table>

          <hr class="my-5" />

          <!-- Tabela Ano -->
          <h2 class="text-center mb-4">📈 Maiores Despesas — Último Ano</h2>

          <div class="d-flex justify-content-end mb-2">
            <button class="btn btn-success" @click="exportarParaExcel(ano, 'Top_Operadoras_Ano')">
              Exportar Ano
            </button>
          </div>

          <table class="table table-bordered table-striped text-center">
            <thead class="table-dark">
              <tr>
                <th>REG ANS</th>
                <th>Despesa (R$)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in ano" :key="item.REG_ANS">
                <td>{{ item.REG_ANS }}</td>
                <td class="text-end">{{ formatCurrency(item.VL_SALDO_FINAL) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- Rodapé -->
    <footer class="bg-light text-center text-muted py-3 mt-5">
      <div class="container">
        &copy; {{ new Date().getFullYear() }} - vegadev
      </div>
    </footer>
  </div>
</template>

<script>
import * as XLSX from 'xlsx'
import { saveAs } from 'file-saver'

export default {
  data() {
    return {
      loading: true,
      erro: null,
      trimestre: [],
      ano: []
    }
  },
  mounted() {
    this.carregarDados()
  },
  methods: {
    formatCurrency(valor) {
      return Number(valor).toLocaleString('pt-BR', {
        style: 'currency',
        currency: 'BRL',
        minimumFractionDigits: 2
      })
    },
    async carregarDados() {
      try {
        const [resTri, resAno] = await Promise.all([
          fetch('http://localhost:5000/top-operadoras-trimestre'),
          fetch('http://localhost:5000/top-operadoras-ano')
        ])

        const trimestre = await resTri.json()
        const ano = await resAno.json()

        if (trimestre.erro || ano.erro) {
          this.erro = trimestre.erro || ano.erro
        } else {
          this.trimestre = trimestre
          this.ano = ano
        }
      } catch (err) {
        console.error(err)
        this.erro = 'Erro ao buscar dados do servidor.'
      } finally {
        this.loading = false
      }
    },
    exportarParaExcel(dados, nomeArquivo) {
      const worksheet = XLSX.utils.json_to_sheet(dados)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Dados')
      const buffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' })
      const blob = new Blob([buffer], { type: 'application/octet-stream' })
      saveAs(blob, `${nomeArquivo}.xlsx`)
    }
  }
}
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f8f9fa;
}

.table th,
.table td {
  vertical-align: middle;
  border: 2px solid #dee2e6 !important;
}
</style>
