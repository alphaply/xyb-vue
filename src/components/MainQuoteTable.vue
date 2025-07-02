<template>
  <button @click="captureTable">导出表格为图片</button>
  <div class="quote-table" ref="tableRef">
    <GroupQuoteTable
        v-for="g in groupedData"
        :key="g.group"
        :groupName="g.groupName"
        :items="g.items"
    />
    <div class="total-box">合计（含票）：￥{{ total.toLocaleString() }}</div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import html2canvas from 'html2canvas'
import GroupQuoteTable from './GroupQuoteTable.vue'

// 你的原始数据
const tableData = [
  { group: 'A', groupName: '前期费用', num: 'A1', category: '场地勘测', describe: '现场勘测', unit: '项', count: 1, price: 0, subtotal: 0 },
  { group: 'A', groupName: '前期费用', num: 'A2', category: '设计', describe: '主KV与衍生设计', unit: '项', count: 2, price: 150, subtotal: 300 },
  // ...其他分组数据...
]

// 按 group 拍平
const groupedData = computed(() => {
  const map = {}
  for (const row of tableData) {
    if (!map[row.group]) map[row.group] = { group: row.group, groupName: row.groupName, items: [] }
    map[row.group].items.push(row)
  }
  return Object.values(map)
})

const total = computed(() => {
  return tableData.reduce((s, r) => s + (r.subtotal || 0), 0)
})

const tableRef = ref(null)
function captureTable() {
  html2canvas(tableRef.value, {
    useCORS: true,
    backgroundColor: "#fff"
  }).then(canvas => {
    const link = document.createElement('a')
    link.download = '报价单.png'
    link.href = canvas.toDataURL('image/png')
    link.click()
  })
}
</script>

<style scoped>
.quote-table { padding: 20px; }
.total-box { text-align: right; margin-top: 16px; font-size: 16px; font-weight: bold; }
</style>
