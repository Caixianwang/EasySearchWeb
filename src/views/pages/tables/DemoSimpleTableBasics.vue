<script setup>
import { reactive, onMounted, watchEffect } from 'vue'
import axios from 'axios'

const form = reactive({
  content: '健康',
})

const tables = reactive([])


const queryList = () => {
  axios.get(`http://localhost:8080/api/query/queryList?content=${form.content}`)
    .then(response => {
      tables.splice(0, tables.length) // 清空之前的内容
      for (let i = 0; i < response.data.length; i++) {
        tables.push(response.data[i])
      }
    })
    .catch(error => {
      console.error('请求失败', error)
    })
}

watchEffect(() => {
  if (form.content.trim()) {
    queryList()
  }
})

onMounted(() => {
  queryList()
})
</script>

<template>
  <VRow>
    <VCol
      cols="6"
      md="6"
    />
    <VCol
      cols="6"
      md="6"
    >
      <VTextField
        v-model="form.content"
        autofocus
        label="Content"
        style="margin-right: 10px"
        placeholder="Please Input Content"
        @keyup.enter="queryList"
      />
    </VCol>
  </VRow>
  <VTable>
    <thead>
    <tr>
      <th>
        author
      </th>
      <th>
        content
      </th>
      <th>
        replies
      </th>
    </tr>
    </thead>

    <tbody>
    <tr
      v-for="item in tables"
      :key="item._ID_"
    >
      <td>
        {{ item.author }}
      </td>
      <td v-html="item.content"></td>
      <td>
        {{ item.replies }}
      </td>
    </tr>
    </tbody>
  </VTable>
</template>
