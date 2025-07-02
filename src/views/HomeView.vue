<template>
  <div style="padding: 24px">
    <el-button type="primary" @click="exportPDF">导出合同为PDF</el-button>
    <div ref="pdfDom" class="contract-page" style="width: 794px; margin: 20px auto; background: #fff; padding: 32px; box-sizing: border-box;">
      <h1 style="text-align:center; font-size: 28px; margin-bottom: 24px;">技术服务合同</h1>
      <div style="margin-bottom: 16px;">
        <p>甲方（客户）：<span style="border-bottom: 1px solid #aaa; min-width:100px;display:inline-block"></span></p>
        <p>乙方（服务商）：<span style="border-bottom: 1px solid #aaa; min-width:100px;display:inline-block"></span></p>
      </div>
      <h3>一、服务内容</h3>
      <ol>
        <li>乙方为甲方提供如下技术服务，并根据下方报价单执行。</li>
        <li>服务时间、方式和其他要求按双方补充约定执行。</li>
      </ol>

      <h3 style="margin:24px 0 12px 0;">二、报价单</h3>
      <el-table
          :data="flattenedData"
          border
          style="width: 100%;"
          :row-class-name="getRowClass"
          :span-method="mergeCells"
          size="small"
      >
        <el-table-column label="服务要求" align="center">
          <el-table-column prop="num" label="序号" width="60" />
          <el-table-column prop="category" label="类别" width="100">
            <template #default="{ row }">
              <span v-if="row.isGroup"><strong>【{{ row.groupName }}】</strong></span>
              <span v-else-if="row.isSubtotal" style="font-weight: bold;">小计</span>
              <span v-else>{{ row.category }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="describe" label="描述" />
        </el-table-column>
        <el-table-column prop="unit" label="单位" width="60" />
        <el-table-column prop="count" label="数量" width="60" />
        <el-table-column label="报价" align="center">
          <el-table-column prop="price" label="不含税单价" width="100" />
          <el-table-column prop="subtotal" label="不含税小计" width="110">
            <template #default="{ row }">
              <span v-if="row.isSubtotal" style="font-weight: bold;">￥{{ row.subtotal.toFixed(2) }}</span>
              <span v-else-if="row.isGroup"></span>
              <span v-else>￥{{ row.subtotal?.toFixed(2) }}</span>
            </template>
          </el-table-column>
        </el-table-column>
      </el-table>
      <div class="total-box" style="margin-bottom:16px;">合计（含票）：￥{{ total.toLocaleString() }}</div>

      <h3>三、备注</h3>
      <p style="margin-bottom: 40px;">1. 上述报价均为人民币，不含税。如需开票请提前说明。<br>2. 本合同一式两份，甲乙双方各执一份，双方签字盖章后生效。</p>
      <div style="display:flex;justify-content:space-between;align-items:flex-end; margin-top:36px;">
        <div>
          <div>甲方（签章）：</div>
          <img src="/gongzhang.png" alt="甲方公章" style="width: 120px; margin-top:16px;">
        </div>
        <div>
          <div>乙方（签章）：</div>
          <img src="/gongzhang.png" alt="乙方公章" style="width: 120px; margin-top:16px;">
        </div>
      </div>
      <div style="margin-top: 32px; text-align:right;">签订日期：<span style="border-bottom:1px solid #aaa;min-width:120px;display:inline-block"></span></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'
import autoTable from "jspdf-autotable"
// 合同内容ref
const pdfDom = ref(null)

// 表格数据
const tableData = reactive([
  { group: 'A', groupName: '前期费用', num: 'A1', category: '场地勘测', describe: '现场勘测', unit: '项', count: 1, price: 0, subtotal: 0 },
  { group: 'A', groupName: '前期费用', num: 'A2', category: '设计', describe: '主KV与衍生设计', unit: '项', count: 2, price: 150, subtotal: 300 },
  { group: 'B', groupName: '布展', num: 'B1', category: '主KV', describe: '主KV布展（含框架作业）', unit: '项', count: 1, price: 2000, subtotal: 2000 },
  { group: 'B', groupName: '布展', num: 'B2', category: '礼品金墙', describe: '异形首金墙（含框架作业）', unit: '项', count: 1, price: 2800, subtotal: 2800 },
  { group: 'B', groupName: '布展', num: '', category: '铝框', describe: '亚克力铝框', unit: '项', count: 15, price: 18, subtotal: 270 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh120*90cm', unit: '项', count: 8, price: 75, subtotal: 600 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh50*60cm', unit: '项', count: 4, price: 28, subtotal: 112 },
  { group: 'B', groupName: '布展', num: '', category: '军令状', describe: 'KT军令状 wh35*40cm', unit: '项', count: 3, price: 40, subtotal: 120 },
  { group: 'B', groupName: '布展', num: '', category: '指引牌', describe: '立式指引牌', unit: '项', count: 1, price: 300, subtotal: 300 },
  { group: 'C', groupName: '人员支持', num: 'C1', category: '摄影', describe: '图片直播1名 7H全程内通勤', unit: '项', count: 1, price: 1700, subtotal: 1700 },
  { group: 'D', groupName: '其他', num: 'D1', category: '施工人员', describe: '进场施工工人1次，撤场1次', unit: '项', count: 2, price: 425, subtotal: 850 },
  { group: 'D', groupName: '其他', num: 'D2', category: '项目管理', describe: '项目运维2次', unit: '项', count: 2, price: 95, subtotal: 190 }
])

const flattenedData = computed(() => {
  const result = []
  const grouped = {}
  for (const row of tableData) {
    if (!grouped[row.group]) grouped[row.group] = []
    grouped[row.group].push(row)
  }
  for (const groupKey of Object.keys(grouped)) {
    const rows = grouped[groupKey]
    const groupName = rows[0].groupName
    result.push({ isGroup: true, group: groupKey, groupName })
    result.push(...rows)
    const subtotal = rows.reduce((s, r) => s + (r.subtotal || 0), 0)
    result.push({ isSubtotal: true, group: groupKey, subtotal })
  }
  return result
})

const total = computed(() => {
  return tableData.reduce((s, r) => s + (r.subtotal || 0), 0)
})

// 合并特殊行
function mergeCells({ row, columnIndex }) {
  if (row.isGroup || row.isSubtotal) {
    return columnIndex === 0 ? { rowspan: 1, colspan: 7 } : { rowspan: 0, colspan: 0 }
  }
}

// 设置行样式
function getRowClass({ row }) {
  if (row.isGroup) return 'group-row'
  if (row.isSubtotal) return 'subtotal-row'
  return ''
}

// 导出 PDF
function exportPDF() {
  const doc = new jsPDF('p', 'mm', 'a4')

  // 1. 合同头
  doc.setFontSize(18)
  doc.text('技术服务合同', 105, 20, { align: 'center' })

  doc.setFontSize(12)
  doc.text('甲方（客户）：_________', 20, 35)
  doc.text('乙方（服务商）：_________', 120, 35)

  // 2. 正文内容
  doc.setFontSize(13)
  doc.text('一、服务内容', 20, 50)
  doc.setFontSize(11)
  doc.text([
    '1. 乙方为甲方提供如下技术服务，并根据下方报价单执行。',
    '2. 服务时间、方式和其他要求按双方补充约定执行。'
  ], 25, 58)

  // 3. 报价单表格（数据结构化填入）
  autoTable(doc, {
    startY: 75,
    head: [['序号','类别','描述','单位','数量','单价','小计']],
    body: tableData.map(row => [
      row.num, row.category, row.describe, row.unit, row.count, row.price, row.subtotal
    ]),
    theme: 'grid'
  })

  let finalY = doc.lastAutoTable.finalY + 10

  // 4. 合计
  doc.setFontSize(12)
  doc.text(`合计（含票）：￥${total.toLocaleString()}`, 150, finalY)

  // 5. 备注
  finalY += 15
  doc.text('三、备注', 20, finalY)
  finalY += 7
  doc.setFontSize(10)
  doc.text([
    '1. 上述报价均为人民币，不含税。如需开票请提前说明。',
    '2. 本合同一式两份，甲乙双方各执一份，双方签字盖章后生效。'
  ], 25, finalY)

  // 6. 签章位置
  finalY += 20
  doc.setFontSize(12)
  doc.text('甲方（签章）：', 20, finalY)
  doc.text('乙方（签章）：', 120, finalY)

  // 7. 插入公章图片（例如加在乙方签章右侧）
  // doc.addImage(章图片base64, 'PNG', 150, finalY-10, 25, 25)

  doc.save('合同.pdf')
}



// function exportPDF() {
//   const dom = pdfDom.value
//   html2canvas(dom, { useCORS: true, scale: 2, backgroundColor: "#fff" }).then(canvas => {
//     const imgData = canvas.toDataURL('image/png')
//     const pdf = new jsPDF('p', 'pt', 'a4')
//     const pageWidth = pdf.internal.pageSize.getWidth()
//     const pageHeight = pdf.internal.pageSize.getHeight()
//     const imgWidth = pageWidth
//     const imgHeight = canvas.height * (pageWidth / canvas.width)
//
//     let heightLeft = imgHeight
//     let position = 0
//
//     while (heightLeft > 0) {
//       pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight)
//       heightLeft -= pageHeight
//       if (heightLeft > 0) {
//         pdf.addPage()
//         position = -heightLeft
//       }
//     }
//     pdf.save('合同.pdf')
//   })
// }
</script>

<style scoped>
.contract-page {
  font-size: 15px;
  color: #222;
  background: #fff;
}
.group-row {
  background-color: #f5f7fa !important;
  font-weight: bold;
  text-align: left;
}
.subtotal-row {
  background-color: #fafafa !important;
  font-weight: bold;
}
.total-box {
  text-align: right;
  margin-top: 12px;
  font-size: 16px;
  font-weight: bold;
}
</style>
