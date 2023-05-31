<template>
  <div class="table">
    <div
        v-for="row in rowsData"
        :key="row.id"
        class="table-row"
    >
      <div v-for="col in row.cols" :key="col.id" class="table-col">
        <input
          v-model.number="col.value"
          @input="(e) => {e.target.value = e.target.value.replace(/\D/g, '')}"
        />
      </div>
      <div class="table-col table-col-result">{{rowsResult[row.id] as string}} ₽</div>
    </div>

    <div v-if="rowsData.at(0)" class="table-row">
      <div v-for="col in rowsData.at(0).cols" class="table-col table-col-result">{{colsResult[col.id]}} ₽</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {computed, onBeforeMount, ref} from "vue";

type Col = {
  id: number;
  value: number | '';
}


type Row = {
  id: number;
  cols: Col[]
}

const props = defineProps<{
  rows: number;
  cols: number;
}>();

const rowsData = ref<Row[]>([])

const rowsResult = computed<Record<Row["id"], number>>(() => {
  const results = {};

  rowsData.value.forEach((row) => {
    results[row.id as string] = row.cols.reduce((accum, {value}) => accum + (isNaN(Number(value)) ? 0 : Number(value)), <number>0)
  })

  return results;
})

const colsResult = computed<Record<Col['id'], number>>(() => {
  const results = {};

  rowsData.value.forEach((row) => {
    row.cols.forEach((col) => {
      if (results[col.id as string]) {
        results[col.id as string] += (isNaN(Number(col.value)) ? 0 : Number(col.value))
      } else {
        results[col.id as string] = isNaN(Number(col.value)) ? 0 : Number(col.value)
      }
    })
  })

  return results;
})

const getUniqId = (): number => {
  const randNum = Math.floor(Math.random() * 1000)

  if (rowsData.value.map((row) => row.id).includes(randNum)) return getUniqId();

  return randNum;
}

const generateRow = (): Row => {
  const id = getUniqId();
  const cols: Col[] = [];
  for (let i = 1; i <= props.cols; i += 1) {
    cols.push({id: i ,value: ''})
  }

  return {id, cols}
}

const generateInitialData = (): void => {
  for (let i = 0; i < props.rows; i += 1) {
    rowsData.value.push(generateRow())
  }
}


onBeforeMount(() => {
  generateInitialData();
});
</script>

<style scoped>
.table {
  display: flex;
  flex-direction: column;
}

.table-row {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  border-bottom:1px solid green;
}

.table-row:first-of-type {
  border-top:1px solid green;
}

.table-row:last-of-type {
  padding-right: 101px;
  border-bottom: 0;
}

.table-row:last-of-type .table-col {
  flex-grow: 1;
  border-bottom: 1px solid green;
}

.table-col {
  min-width: 100px;
  height: 40px;
  border-left: 1px solid green;
}

.table-col input {
  height: 100%;
  box-sizing: border-box;
  border: none;
}

.table-col-result {
  width: 100px;
  background-color: rgba(23, 122, 23, 0.4);
  color: #fff;
  border-right: 1px solid green;
}
</style>
