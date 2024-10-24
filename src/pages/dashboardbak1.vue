<script setup>
import { reactive, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'
import CardNode from "@core/components/cards/CardNode.vue"


// 定义响应式对象
const master = reactive({
  docsTotal: '',
  host: '',
  port: '',
  balance: false,
})

const channels = reactive([])

const masterBalance = () => {
  axios.get('http://localhost:8080/api/distribute/test')
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
      console.log(response.data)
      let total = 0
      let masterHost = ''
      let masterPort = ''
      let disableBalance = false // 初始化为 false
      channels.length = 0 // 清空之前的数据
      for (let i = 0; i < response.data.length; i++) {
        let channel = response.data[i]
        let docsTotal = channel.docsTotal
        let host = channel.targetHost
        let port = channel.targetPort
        let state = channel.state.split("_")[1]
        total += docsTotal

        if (state === 'BALANCING' || state === 'REMOVING') {
          disableBalance = true
        }

        channels.push({ "host": host, "port": port, "docsTotal": docsTotal, "state": state })
        masterHost = channel.sourceHost
        masterPort = channel.sourcePort
      }
      master.host = masterHost
      master.port = masterPort
      master.docsTotal = total
      master.balance = disableBalance
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}

// 定时调用的设置
let intervalId

// onMounted 组件挂载后开始定时调用
onMounted(() => {
  fetchNodeData() // 立即调用一次
  intervalId = setInterval(fetchNodeData, 3000) // 每3秒调用一次
})

// onBeforeUnmount 组件卸载前清除定时器
onBeforeUnmount(() => {
  if (intervalId) {
    clearInterval(intervalId)
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
          host: item.host,
          port: item.port,
          docsTotal:item.docsTotal,
          state: item.state,
          balance: master.balance,
        }"
      />
    </VCol>

  </VRow>
</template>
