
<template>
  <div class="content">
    <div class="left">
      <div class="map-size">
        <p>地图宽度</p>
        <n-input-number :step="10" v-model:value="width"  />
      </div>
      <div class="block-ratio">
        <p>障碍物概率</p>
        <n-space vertical>
          <n-slider v-model:value="blockRatio" :step="10" />
          <n-input-number v-model:value="blockRatio" size="small" />
        </n-space>
        <n-button type="primary" @click="refreshGraph">刷新</n-button>
      </div>

      <div class="start-end">
        <p>起点坐标</p>
        <div class="up">
          <n-input-number v-model:value="startX" clearable />
          <n-input-number v-model:value="startY" clearable />
        </div>
        <p>终点坐标</p>
        <div class="bottom">
          <n-input-number v-model:value="endX" clearable />
          <n-input-number v-model:value="endY" clearable />
        </div>
        <n-button type="primary" @click="planPath">规划</n-button>
      </div>
    </div>
    <div class="right" :style="{ width: graphEleWidth + 'px', height: graphEleHeight + 'px' }">
      <template v-for="(item, index1) in graph">
        <div v-for="(node, index2) in item" :key="index1 * width + index2" :class="[code2className(node.value), 'node']"
          @click="changeBlock(index1, index2)" :style="{
            width: itemEleWidth + 'px', height: itemEleHeight + 'px',

            left: index1 * itemEleWidth + 'px', top: index2 * itemEleHeight + 'px'
          }"></div>
      </template>
    </div>
  </div>
</template>
<script setup>
import { ref, watch, computed } from 'vue'
const BLOCK = 1
const PATH = 2
const TERMINAL = 3
const NULL = 0

const code2className = (code) => {
  switch (code) {
    case BLOCK:
      return 'block'
    case PATH:
      return 'path'
    case TERMINAL:
      return 'terminal'
    case NULL:
      return ''
  }
}
const graphEleWidth = 800
const graphEleHeight = 800

const blockRatio = ref(20)

let width = ref(30)
let height = computed(() => {
  return width.value
})

let graph = ref(new Array(width))
let sizeChanged = false

let itemEleWidth = graphEleWidth / width.value
let itemEleHeight = graphEleHeight / height.value
watch(width, () => {
  itemEleWidth = graphEleWidth / width.value
  itemEleHeight = graphEleHeight / height.value
  sizeChanged = true
  refreshGraph()
})
const changeBlock = (x, y) => {
  graph.value[x][y].value = graph.value[x][y].value == 1 ? 0 : 1
  planPath()
}

const startX = ref(0)
const startY = ref(0)
const endX = ref(width.value-1)
const endY = ref(height.value-1)



const graphValue2ddimArr = () => {
  const arr = []
  for (let i = 0; i < width.value; i++) {
    arr[i] = []
    for (let j = 0; j < height.value; j++) {
      switch (graph.value[i][j].value) {
        case PATH:
          arr[i][j] = 0
          break
        case TERMINAL:
          arr[i][j] = 0
          break
        default:
          arr[i][j] = graph.value[i][j].value
          break
      }

    }
  }
  return arr
}


const planPath = async () => {
  const res = await fetch('/route', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      map: graphValue2ddimArr(),
      start: [startX.value, startY.value],
      end: [endX.value, endY.value]
    })
  })
  
  const pathArr = await res.json()
  console.log(pathArr)
  for (let i = 0; i < width.value; i++) {
    for (let j = 0; j < height.value; j++) {
      if (graph.value[i][j].value == PATH) {
        graph.value[i][j].value = NULL
      }
    }
  }
  pathArr.route.forEach(item => {
    if (item.toString() === [startX.value, startY.value].toString() || item.toString() == [endX.value, endY.value].toString()) {
      return
    }
    graph.value[item[0]][item[1]].value = PATH
  })
}
const refreshGraph = () => {
  graph.value = new Array(width.value)
  for (let i = 0; i < width.value; i++) {
    graph.value[i] = new Array(height)
    for (let j = 0; j < height.value; j++) {
      graph.value[i][j] = {
        x: i,
        y: j,
        value: Math.random() < (blockRatio.value / 200 + 0.1) ? 1 : 0
      }
    }
  }
  endX.value = width.value-1
  endY.value = height.value-1
  graph.value[startX.value][startY.value].value = 3
  graph.value[endX.value][endY.value].value = 3
  planPath()
}
refreshGraph()

// 更改起点终点，将上一次起点终点置为0
watch([startX, startY, endX, endY], (_, oldValue) => {
  if (oldValue.length>1 ) {
    if (sizeChanged) {
      sizeChanged = false
      return
    }
    graph.value[oldValue[0]][oldValue[1]].value = 0
    graph.value[oldValue[2]][oldValue[3]].value = 0
  }
  graph.value[startX.value][startY.value].value = 3
  graph.value[endX.value][endY.value].value = 3
  planPath()
}, { immediate: true })





</script>

<style scoped lang="scss">
.block {
  background-color: #ddd;
}

.node {
  position: absolute;
}

.path {
  background-color: aquamarine;
}

.terminal {
  background-color: red;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;

  .content {
    display: flex;
    align-items: center;
    border-radius: 20px;
    padding: 20px;
    border: 1px solid #000;
    width: 1200px;

    .left {
      flex: 1;
      margin-right: 20px;

      .start-end {
        margin-top: 20px;

        .up {
          display: flex;
        }

        .bottom {
          display: flex;
        }
      }
    }

    .right {
      box-sizing: border-box;
      position: relative;
      border: 1px solid #000;
    }
  }
}</style>
