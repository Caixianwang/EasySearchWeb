<script setup>
import { reactive, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'
import CardNode from "@core/components/cards/CardNode.vue"


// 定义响应式对象
const master = reactive({
  docsTotal: '',
  host: '',
  indexPath: '',
  port: '',
  webPort: '',
  balance: false,
})

const channels = reactive([])

const masterBalance = () => {
  axios.get('http://localhost:8080/api/distribute/balance')
    .then(response => {
      console.log(response.data)
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}

const fetchNodeData = () => {
  axios.get('http://localhost:8080/api/node/queryNodes')
    .then(response => {
      // console.log(response.data)
      handleSlaves(response.data)
      websocket()
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}

const handleSlaves = slaves => {
  channels.length = 0
  let total = 0
  master.balance = false
  for (let i = 0; i < slaves.length; i++) {
    let channel = slaves[i]
    let docsTotal = channel.docsTotal
    let host = channel.targetHost
    let port = channel.targetPort
    let webPort = channel.slaveServerPort
    let indexPath = channel.indexPath

    let state = channel.state.split("_")[1]
    total += docsTotal
    if (state === 'BALANCING' || state === 'REMOVING') {
      master.balance = true
    }
    channels.push({
      "host": host,
      "port": port,
      "docsTotal": docsTotal,
      "state": state,
      "webPort": webPort,
      "indexPath": indexPath,
    })
    master.host = channel.sourceHost
    master.port = channel.sourcePort
  }
  master.docsTotal = total
}

const websocket = () => {
  let ws = new WebSocket("ws://localhost:8080/websocket/slaveState")
  ws.onopen = () => {
    console.log('onopen...')
  }
  ws.onmessage = evt => {
    console.log('onmessage...')
    let resJson = JSON.parse(evt.data)
    handleSlaves(resJson)

  }
  ws.onclose = function () {
    // 关闭 websocket
    console.log('onclose...')
  }
}

// 定时调用的设置
let intervalId

// onMounted 组件挂载后开始定时调用
onMounted(() => {
  fetchNodeData() // 立即调用一次
  // intervalId = setInterval(fetchNodeData, 500) // 每3秒调用一次
})

// onBeforeUnmount 组件卸载前清除定时器
onBeforeUnmount(() => {
  if (intervalId) {
    // clearInterval(intervalId)
  }
})
</script>

<template>
  <VRow>
    <!-- 👉 Profit -->
    <VCol
      cols="12"
      md="12"
    >
      <VCard>
        <VCardItem class="pb-3">
          <VCardTitle class="text-primary">
            Master Information
          </VCardTitle>
        </VCardItem>
        <VCardText>
          <p class="mb-1">
            IP: {{ master.host }}
          </p>
          <p class="mb-1">
            Port: {{ master.port }}
          </p>
          <h5 class="text-h5 text-no-wrap mb-3">
            docsTotal: {{ master.docsTotal }}
          </h5>
          <VBtn
            variant="tonal"
            class="mt-6"
            size="small"
            :disabled="master.balance"
            @click="masterBalance"
          >
            Balance
          </VBtn>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
  <VRow>
    <!-- 👉 Profit -->
    <VCol
      cols="12"
      md="4"
      v-for="(item, index) in channels" :key="index"
    >
      <CardNode
        v-bind="{
          indexPath: item.indexPath,
          host: item.host,
          port: item.port,
          webPort: item.webPort,
          docsTotal:item.docsTotal,
          state: item.state,
          balance: master.balance,
        }"
      />
    </VCol>

  </VRow>
</template>
