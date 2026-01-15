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

const emit = defineEmits(['changeNodeFillColor', 'saveGraph'])

const selectionOpened = ref(false)
const undoAble = ref(false)
const redoAble = ref(false)
const colors = ref('#345678')
const linetype = ref('pro-polyline')
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
