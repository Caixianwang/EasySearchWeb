<script setup>
import axios from 'axios'

const props = defineProps({
  indexPath: {
    type: String,
    required: true,
  },
  host: {
    type: String,
    required: true,
  },
  port: {
    type: Number,
    required: true,
  },
  webPort: {
    type: Number,
    required: true,
  },
  state: {
    type: String,
    required: true,
  },
  docsTotal: {
    type: Number,
    required: true,
  },
  balance: {
    type: Boolean,
    required: true,
  },
})

// 定义响应式对象
const slave = reactive({
  remove: false,
})

const slaveRemove = () => {
  let url = "http://localhost:8080/api/distribute/remove?host=" + props.host + "&port=" + props.port
  axios.get(url)
    .then(response => {
      console.log(response.data)
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}

const startSlave = () => {
  let url = "http://localhost:8080/api/distribute/startSlave?serverPort=" + props.webPort + "&port=" + props.port+ "&indexPath=" + props.indexPath
  axios.get(url)
    .then(response => {
      console.log(response.data)
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}
</script>

<template>
  <VCard>
    <VCardItem class="pb-3">
      <VCardTitle class="text-primary">
        Slave Information
      </VCardTitle>
    </VCardItem>
    <VCardText>
      <p class="mb-1">
        IP: {{ props.host }}
      </p>
      <p class="mb-1">
        WebPort: {{ props.webPort }}
      </p>
      <p class="mb-1">
        RPCPort: {{ props.port }}
      </p>
      <h5 class="text-h5 text-no-wrap mb-3">
        docsTotal: {{ props.docsTotal }}
      </h5>
      <h6 class="text-h5 text-no-wrap mb-3">
        state: <span style="color: #71d9c1">{{ props.state }}</span>
      </h6>
      <VBtn
        variant="tonal"
        class="mt-6"
        size="small"
        :disabled="props.state=='CLOSED'||props.balance"
        @click="slaveRemove"
      >
        Remove Slave
      </VBtn>
      <VBtn
        variant="tonal"
        class="mt-6"
        size="small"
        :disabled="(props.state!='CLOSED'&&props.state!='REMOVING')||props.balance"
        @click="startSlave"
      >
        Start Slave
      </VBtn>
    </VCardText>
  </VCard>
</template>
