<template>
  <div class="group-quote-table">
<!--    <div class="group-title">{{ groupName }}</div>-->
    <el-table
        :data="items"
        border
        style="width: 100%"
        v-if="showHeader"
    >
      <!-- 多级表头结构 -->
      <el-table-column
          prop="num"
          label="序号"
          width="60"
          align="center"
          :render-header="renderMultiLineHeader"
      />
      <el-table-column
          label="服务要求"
          align="center"
          :render-header="renderMultiLineHeader"
      >
        <el-table-column
            prop="category"
            label="类目"
            align="center"
            width="100"
            :render-header="renderMultiLineHeader"
        />
        <el-table-column
            prop="describe"
            label="描述"
            align="center"
            :render-header="renderMultiLineHeader"
        />
      </el-table-column>
      <el-table-column
          label="报价"
          align="center"
          :render-header="renderMultiLineHeader"
      >
        <el-table-column
            prop="unit"
            label="单位"
            align="center"
            width="80"
            :render-header="renderMultiLineHeader"
        />
        <el-table-column
            prop="count"
            label="数量"
            align="center"
            width="80"
            :render-header="renderMultiLineHeader"
        />
        <el-table-column
            prop="price"
            label="不含税单价"
            align="center"
            width="120"
            :render-header="renderMultiLineHeader"
        />
        <el-table-column
            prop="subtotal"
            label="不含税小计"
            align="center"
            width="120"
            :render-header="renderMultiLineHeader"
        >
          <template #default="{ row }">￥{{ row.subtotal?.toFixed(2) }}</template>
        </el-table-column>
      </el-table-column>
    </el-table>
    <!-- 其它分组用无表头表格 -->
    <el-table
        v-else
        :data="items"
        border
        style="width: 100%"
        :show-header="false"
    >
      <el-table-column prop="num" width="60" align="center"/>
      <el-table-column prop="category" width="100" align="center"/>
      <el-table-column prop="describe"/>
      <el-table-column prop="unit" width="80" align="center"/>
      <el-table-column prop="count" width="80" align="center"/>
      <el-table-column prop="price" width="120" align="center"/>
      <el-table-column prop="subtotal" width="120" align="center">
        <template #default="{ row }">￥{{ row.subtotal?.toFixed(2) }}</template>
      </el-table-column>
    </el-table>
    <div class="subtotal-row">{{ groupName }} |
      小计：￥{{ subtotal.toLocaleString() }}
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
const props = defineProps({
  groupName: String,
  items: Array,
  showHeader: {
    type: Boolean,
    default: false
  }
})
const subtotal = computed(() => {
  return props.items.reduce((s, r) => s + (r.subtotal || 0), 0)
})
</script>

<style scoped>
.group-title { font-weight: bold; margin-bottom: 4px; }
.subtotal-row { text-align: right; background: #fafafa; padding: 8px 0 8px 0;text-align: center}
</style>
