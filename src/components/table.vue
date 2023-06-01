<template>
  <div class="table">
    <p class="table-desc">{{previewMessage}}</p>
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
      <div class="table-col table-col-result">{{ rowResults?.[row.id] ?? 0 }} ₽</div>
    </div>

    <div v-if="rowsData.at(0)" class="table-row">
      <div v-for="col in rowsData.at(0).cols" class="table-col table-col-result">{{ colsResult[col.id] }} ₽</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {computed, onBeforeMount, ref, watch} from "vue";

type Col = {
  id: number;
  value: number | '';
}


type Row = {
  id: number;
  cols: Col[];
}

const props = defineProps<{
  rows: number;
  cols: number;
}>();

const rowsData = ref<Row[]>([])

const rowResults = ref<Record<Row["id"], number>>({});

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

const previewMessage = ref<string>('')

const animate = ({timing, draw, duration}) => {
  let start = performance.now();

  requestAnimationFrame(function a(time) {
    let timeFr = (time - start) / duration;
    if (timeFr > 1) timeFr = 1;

    let progress = timing(timeFr);

    draw(progress)

    if (timeFr < 1) {
      requestAnimationFrame(a)
    }
  })
}

watch(
    () => rowsResult.value,
    (value, oldValue) => {
      const changedValueKeys = []
      for (const key in value) {
        if (value[key] !== oldValue[key] && oldValue[key] != undefined) changedValueKeys.push(key)
      }

      changedValueKeys.forEach((k) => {
        const old = oldValue[k];
        const newV = value[k];

        const diff = newV - old;

        animate({
          duration: 800,
          timing: (fr) => {
              return 1 - Math.pow(1 - fr, 2);
          },
          draw(progress) {
            if (progress < 0) return;
            const addedV = Math.floor(diff * progress);

            if (addedV) rowResults.value[k] = old + addedV
          }
        })
      })
    }, { deep: true, flush: "post" })

const getUniqId = (): number => {
  const randNum = Math.floor(Math.random() * 1000)

  if (rowsData.value.map((row) => row.id).includes(randNum)) return getUniqId();

  return randNum;
}

const generateRow = (): Row => {
  const id = getUniqId();
  const cols: Col[] = [];
  for (let i = 1; i <= props.cols; i += 1) {
    cols.push({id: i, value: ''})
  }

  return {id, cols}
}

const generateInitialData = (): void => {
  for (let i = 0; i < props.rows; i += 1) {
    rowsData.value.push(generateRow())
  }
}

const animateText = () => {
  let text = 'Привет, здесь я пытался сделать анимацию печати текста. По-моему, получилось неплохо! Как думаешь? Напиши коммент к репозиторию! :)';
  let to = text.length,
      from = 0;

  animate({
    duration: 7000,
    timing: (tf) => tf,
    draw: function(progress) {
      let result = (to - from) * progress + from;
      const resultText = text.slice(0, Math.ceil(result))

      previewMessage.value = progress < 1 ? resultText + '|' : resultText;
    }
  });
}


onBeforeMount(() => {
  generateInitialData();

  animateText();
});
</script>

<style scoped>
.table-desc {
  height: 150px;
  width: 800px;
  text-align: left;
  margin: 0 auto;
  font-size: 30px;
}

.table {
  display: flex;
  flex-direction: column;
}

.table-row {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  border-bottom: 1px solid green;
}

.table-row:first-of-type {
  border-top: 1px solid green;
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
