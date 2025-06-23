<template>
  <div>
    <h1>Importar XML CT-e</h1>
    <input type="file" accept=".xml" class="imput-type" multiple @change="handleFileUpload" />
    <br><br>

    <table v-if="tableData.length > 0" class="cte-table">
      <thead>
        <tr>
          <th v-for="(value, key) in tableData[0]" :key="key">{{ key }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, rowIndex) in tableData" :key="rowIndex">
          <td v-for="(value, key) in row" :key="key">{{ value }}</td>
        </tr>
      </tbody>
    </table>

    <br>
    <button v-if="tableData.length > 0" @click="downloadExcel" class="button-download">Baixar Excel</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as XLSX from 'xlsx'

const tableData = ref([])

const handleFileUpload = async (event) => {
  const files = event.target.files
  if (!files.length) return

  tableData.value = [] 

  for (const file of files) {
    await new Promise((resolve) => {
      const reader = new FileReader()
      reader.onload = (e) => {
        const parser = new DOMParser()
        const xml = parser.parseFromString(e.target.result, 'application/xml')

        // Captura os dados do XML
        const get = (tag) => {
          const el = xml.getElementsByTagName(tag)[0]
          return el ? el.textContent : ''
        }

        const row = {
          "Número do CT-e": get('nCT'),
          "Data de Emissão": get('dhEmi'),
          "Emitente": get('xNome'),
          "Remetente": get('xNome'),
          "Destinatário": get('xNome'),
          "Valor Total": get('vTPrest'),
          "Tipo de produto": get('proPred'),
        }

        tableData.value.push(row)
        resolve()
      }
      reader.readAsText(file)
    })
  }
}

const downloadExcel = () => {
  const worksheet = XLSX.utils.json_to_sheet(tableData.value)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, 'CTe')
  XLSX.writeFile(workbook, 'XML CTE.xlsx')
}
</script>



<style scoped>

.imput-type {
  border: 1px solid;
  padding: 10px;
  border-radius: 10px;
}

.cte-table {
  border-spacing: 10px;
  border: 2px solid;
  border-radius: 10px;
}

.cte-table td {
  padding: 8px;
  border: 1px solid;
  font-weight: lighter;
  border-radius: 10px;
}
.cte-table th {
  padding: 8px;
  border: 1px solid;
  font-weight: lighter;
  border-radius: 10px;
}

.button-download {
  padding: 10px 20px;
  background-color: #007bff;
  border: none;
  font-weight: lighter;
  border-radius: 10px;
}
</style>