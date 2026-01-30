<template>
  <div>
    <div class="toolbar-item" :class="{'selection-active': selectionOpened}" @click="selectionSelect()">
      <area-select size="18" />
    </div>
    <!-- <div class="toolbar-item toolbar-color-picker">
      <el-popover
        placement="top-start"
        title="填充样式"
        width="220"
        trigger="click"
      >
        <sketch-picker :value="fillColor"  @input="changeFillColor"/>
        <color-fill size="24" slot="reference" />
      </el-popover>
    </div> -->
    <!-- <div class="toolbar-item">
      <color-text size="20" />
    </div>
    <div class="toolbar-item">
      <icon-font size="18" />
    </div>
    <div class="toolbar-item">
      <icon-blod size="18" />
    </div>
    <div class="toolbar-item">
      <icon-line size="18" />
    </div> -->
    <div class="toolbar-item" @click="zoomIn()">
      <zoom-in size="18" />
    </div>
    <div class="toolbar-item" @click="zoomOut()">
      <zoom-out size="18" />
    </div>
    <div
      class="toolbar-item"
      :class="{'disabled': !undoAble}"
      @click="undo()"
    >
      <step-back size="18" />
    </div>
    <div
      class="toolbar-item"
      :class="{'disabled': !redoAble}"
      @click="redo()"
    >
      <step-foward size="18" />
    </div>
    <el-dropdown @command="handleExport">
      <div class="toolbar-item">
        <span style="font-size: 12px; cursor: pointer;">导出</span>
      </div>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item command="json">导出为 JSON</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    <el-dropdown @command="handleImportCommand">
      <div class="toolbar-item">
        <span style="font-size: 12px; cursor: pointer;">导入</span>
      </div>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item command="json">导入 JSON</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    <el-dropdown @command="handleDeleteCommand">
      <div class="toolbar-item">
        <span style="font-size: 12px; cursor: pointer;">删除</span>
      </div>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item command="delete">删除选中元素</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    <input
      ref="fileInput"
      type="file"
      accept=".json"
      style="display: none"
      @change="handleImport"
    />
    <!-- <div>
      <button @click="saveGraph">保存</button>
    </div> -->
    <div>
      <el-select v-model="linetype" size="mini" @change="changeLineType">
        <el-option
          v-for="item in lineOptions"
          :key="item.value"
          :value="item.value"
          :label="item.label"
        ></el-option>
      </el-select>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import ZoomIn from './icon/ZoomIn.vue'
import ZoomOut from './icon/ZoomOut.vue'
import StepBack from './icon/StepBack.vue'
import StepFoward from './icon/StepFoward.vue'
import AreaSelect from './icon/AreaSelect.vue'

// 防抖函数
const debounce = (fn, delay) => {
  let timeoutId = null
  return function(...args) {
    if (timeoutId) clearTimeout(timeoutId)
    timeoutId = setTimeout(() => {
      fn.apply(this, args)
    }, delay)
  }
}

const props = defineProps({
  lf: Object,
  activeEdges: Array,
  fillColor: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['changeNodeFillColor', 'saveGraph', 'importGraph'])

const selectionOpened = ref(false)
const undoAble = ref(false)
const redoAble = ref(false)
const colors = ref('#345678')
const linetype = ref('pro-polyline')
const fileInput = ref(null)
const lineOptions = ref([
  {
    value: 'pro-polyline',
    label: '折线'
  },
  {
    value: 'pro-line',
    label: '直线'
  },
  {
    value: 'pro-bezier',
    label: '曲线'
  }
])

let historyChangeHandler = null

onMounted(() => {
  historyChangeHandler = ({ data: { undoAble: ua, redoAble: ra } }) => {
    redoAble.value = ra
    undoAble.value = ua
  }
  props.lf.on('history:change', historyChangeHandler)
})

onUnmounted(() => {
  if (historyChangeHandler && props.lf) {
    props.lf.off('history:change', historyChangeHandler)
  }
})

const changeFillColor = (val) => {
  emit('changeNodeFillColor', val.hex)
}

const saveGraph = () => {
  emit('saveGraph')
}

const zoomIn = debounce(() => {
  props.lf.zoom(true)
}, 100)

const zoomOut = debounce(() => {
  props.lf.zoom(false)
}, 100)

const undo = () => {
  if (undoAble.value) {
    props.lf.undo()
  }
}

const redo = () => {
  if (redoAble.value) {
    props.lf.redo()
  }
}

const selectionSelect = () => {
  selectionOpened.value = !selectionOpened.value
  if (selectionOpened.value) {
    props.lf.extension.selectionSelect.openSelectionSelect()
  } else {
    props.lf.extension.selectionSelect.closeSelectionSelect()
  }
}

const changeLineType = (value) => {
  const { graphModel } = props.lf
  props.lf.setDefaultEdgeType(value)
  if (props.activeEdges && props.activeEdges.length > 0) {
    props.activeEdges.forEach(edge => {
      graphModel.changeEdgeType(edge.id, value)
    })
  }
}

// 导出为 JSON
const exportJSON = () => {
  const data = props.lf.getGraphData()
  const jsonString = JSON.stringify(data, null, 2)
  const blob = new Blob([jsonString], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = '流程图.json'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

// 处理导出命令
const handleExport = (command) => {
  if (command === 'json') {
    exportJSON()
  }
}

// 触发文件选择
const triggerImport = () => {
  fileInput.value.click()
}

// 处理导入命令（从下拉菜单）
const handleImportCommand = (command) => {
  if (command === 'json') {
    triggerImport()
  }
}

// 处理导入
const handleImport = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      // 发送导入事件给父组件
      emit('importGraph', data)
      // 重置文件输入
      fileInput.value.value = ''
    } catch (error) {
      alert('JSON 文件格式错误，请检查文件内容')
      console.error('导入失败:', error)
    }
  }
  reader.readAsText(file)
}

// 删除选中的元素
const deleteSelected = () => {
  const { nodes, edges } = props.lf.getSelectElements()

  if (nodes.length === 0 && edges.length === 0) {
    alert('请先选择要删除的元素')
    return
  }

  // 删除节点
  nodes.forEach(node => {
    props.lf.deleteNode(node.id)
  })

  // 删除边
  edges.forEach(edge => {
    props.lf.deleteEdge(edge.id)
  })
}

// 处理删除命令（从下拉菜单）
const handleDeleteCommand = (command) => {
  if (command === 'delete') {
    deleteSelected()
  }
}
</script>

<style scoped>
.toolbar-item {
  width: 18px;
  height: 18px;
  float: left;
  margin: 12px 4px;
  cursor: pointer;
}
.toolbar-color-picker {
  width: 24px;
  height: 24px;
  margin: 8px 4px;
}
.selection-active {
  background: #33a3dc;
}
</style>
