<template>
  <button @click="captureTable">导出表格为图片</button>
  <div class="quote-table" ref="tableRef">
    <GroupQuoteTable
        v-for="(g, idx) in groupedData"
        :key="g.group"
        :groupName="g.groupName"
        :items="g.items"
        :showHeader="idx === 0"
    />
    <div class="total-box">合计（含票）：￥{{ total.toLocaleString() }}</div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import html2canvas from 'html2canvas'
import GroupQuoteTable from '@/components/GroupQuoteTable.vue' // 路径根据实际调整

// 原始分组数据（你的数据原样搬过来）
const tableData = [
  // ====== A组 ======
  { group: 'A', groupName: '前期费用', num: 'A1', category: '场地勘测', describe: '现场勘测', unit: '项', count: 1, price: 0, subtotal: 0 },
  { group: 'A', groupName: '前期费用', num: 'A2', category: '设计', describe: '主KV与衍生设计', unit: '项', count: 2, price: 150, subtotal: 300 },
  // ====== B组 ======
  { group: 'B', groupName: '布展', num: 'B1', category: '主KV', describe: '主KV布展（含框架作业）', unit: '项', count: 1, price: 2000, subtotal: 2000 },
  { group: 'B', groupName: '布展', num: 'B2', category: '礼品金墙', describe: '聚苯乙烯 异形首金墙（含框架作业）', unit: '项', count: 1, price: 2800, subtotal: 2800 },
  { group: 'B', groupName: '布展', num: '', category: '铝框', describe: '亚克力铝框', unit: '项', count: 15, price: 18, subtotal: 270 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh120*90cm', unit: '项', count: 8, price: 75, subtotal: 600 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh50*60cm', unit: '项', count: 4, price: 28, subtotal: 112 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh35*40cm 8个', unit: '项', count: 3, price: 40, subtotal: 120 },
  { group: 'B', groupName: '布展', num: '', category: '指引牌', describe: '立式指引牌', unit: '项', count: 1, price: 300, subtotal: 300 },
  // ====== C组 ======
  { group: 'C', groupName: '人员支持', num: 'C1', category: '摄影', describe: '图片直播1名 7H全程内通勤', unit: '项', count: 1, price: 1700, subtotal: 1700 },
  // ====== D组 ======
  { group: 'D', groupName: '其他', num: 'D1', category: '施工人员', describe: '进场施工工人1次，撤场1次', unit: '项', count: 2, price: 425, subtotal: 850 },
  { group: 'D', groupName: '其他', num: 'D2', category: '项目管理', describe: '项目运维2次', unit: '项', count: 2, price: 95, subtotal: 190 }
]

// 按 group 分组
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
.quote-table {
  /* 推荐 A4 内容区宽度 */
  max-width: 750px;
  margin: 0 auto;
  padding: 20px;
  background: #fff;
}
.el-table__empty-block {
  display: none !important;
}
.main-header-table .el-table__empty-block {
  display: none !important;
}
</style>
