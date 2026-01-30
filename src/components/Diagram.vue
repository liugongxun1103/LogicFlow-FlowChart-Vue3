<template>
  <div class="diagram">
    <diagram-toolbar
      class="diagram-toolbar"
      v-if="lf"
      :lf="lf"
      :activeEdges="activeEdges"
      @changeNodeFillColor="changeNodeFill"
      @saveGraph="saveGraph"
      @importGraph="importGraph"
    />
    <div class="diagram-main">
      <diagram-sidebar class="diagram-sidebar" @dragInNode="dragInNode" />
      <div class="diagram-container" ref="container">
        <div class="diagram-wrapper">
          <div class="lf-diagram" ref="diagram"></div>
        </div>
      </div>
    </div>
    <!-- 右侧属性面板 -->
    <PropertyPanel
      class="diagram-panel"
      v-show="activeNodes.length>0 || activeEdges.length > 0"
      :onlyEdge="activeNodes.length === 0"
      :elementsStyle="properties"
      @setStyle="setStyle"
      @setZIndex="setZIndex"
    />
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'
import LogicFlow from '@logicflow/core'
import { SelectionSelect } from '@logicflow/extension'
import '@logicflow/core/dist/style/index.css'
import '@logicflow/extension/lib/style/index.css'
import DiagramToolbar from './DiagramToolbar.vue'
import DiagramSidebar from './DiagramSidebar.vue'
import PropertyPanel from './PropertyPanel.vue'
import { registerCustomElement } from './node'

const sidebarWidth = ref(200)
const diagramWidth = ref(0)
const diagramHeight = ref(0)
const lf = ref('')
const filename = ref('')
const activeNodes = ref([])
const activeEdges = ref([])
const properties = ref({})
const diagram = ref(null)
const container = ref(null)
const background = ref(null)

// 保存 updatePatternTransform 函数的引用
let updatePatternTransformFn = null

const initLogicFlow = (data) => {
  // 引入框选插件
  LogicFlow.use(SelectionSelect)
  const lfInstance = new LogicFlow({
    container: diagram.value,
    overlapMode: 1,
    autoWrap: true,
    metaKeyMultipleSelected: true,
    keyboard: {
      enabled: true
    },
    grid: {
      visible: false,
      size: 20
    },
    background: {
      backgroundColor: 'transparent'
    }
  })
  lfInstance.setTheme(
    {
      baseEdge: { strokeWidth: 1 },
      baseNode: { strokeWidth: 1 },
      nodeText: { overflowMode: 'autoWrap', lineHeight: 1.5 },
      edgeText: { overflowMode: 'autoWrap', lineHeight: 1.5 }
    }
  )
  // 注册自定义元素
  registerCustomElement(lfInstance)
  lfInstance.setDefaultEdgeType('pro-polyline')
  lfInstance.render(data)
  lf.value = lfInstance

  // 使用防抖处理选择事件，避免频繁更新
  let selectionTimeout = null
  const handleSelection = () => {
    if (selectionTimeout) clearTimeout(selectionTimeout)
    selectionTimeout = setTimeout(() => {
      const { nodes, edges } = lf.value.getSelectElements()
      activeNodes.value = nodes
      activeEdges.value = edges
      getProperty()
    }, 50)
  }

  lf.value.on('selection:selected,node:click,blank:click,edge:click', handleSelection)

  // 在 SVG 中添加背景网格图案
  setTimeout(() => {
    const svgElement = diagram.value.querySelector('svg')
    if (svgElement) {
      // 创建 defs 元素
      let defs = svgElement.querySelector('defs')
      if (!defs) {
        defs = document.createElementNS('http://www.w3.org/2000/svg', 'defs')
        svgElement.insertBefore(defs, svgElement.firstChild)
      }

      // 创建网格图案
      const pattern = document.createElementNS('http://www.w3.org/2000/svg', 'pattern')
      pattern.setAttribute('id', 'grid-pattern')
      pattern.setAttribute('width', '20')
      pattern.setAttribute('height', '20')
      pattern.setAttribute('patternUnits', 'userSpaceOnUse')

      // 创建垂直线
      const verticalLine = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      verticalLine.setAttribute('x1', '0')
      verticalLine.setAttribute('y1', '0')
      verticalLine.setAttribute('x2', '0')
      verticalLine.setAttribute('y2', '20')
      verticalLine.setAttribute('stroke', '#d0d0d0')
      verticalLine.setAttribute('stroke-width', '1')

      // 创建水平线
      const horizontalLine = document.createElementNS('http://www.w3.org/2000/svg', 'line')
      horizontalLine.setAttribute('x1', '0')
      horizontalLine.setAttribute('y1', '0')
      horizontalLine.setAttribute('x2', '20')
      horizontalLine.setAttribute('y2', '0')
      horizontalLine.setAttribute('stroke', '#d0d0d0')
      horizontalLine.setAttribute('stroke-width', '1')

      pattern.appendChild(verticalLine)
      pattern.appendChild(horizontalLine)
      defs.appendChild(pattern)

      const lfBase = svgElement.querySelector('.lf-base')

      if (lfBase) {
        // 创建背景矩形
        const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect')
        rect.setAttribute('x', '-5000')
        rect.setAttribute('y', '-5000')
        rect.setAttribute('width', '10000')
        rect.setAttribute('height', '10000')
        rect.setAttribute('fill', 'url(#grid-pattern)')
        rect.setAttribute('style', 'pointer-events: none;')
        rect.setAttribute('class', 'grid-background')

        lfBase.insertBefore(rect, lfBase.firstChild)

        // 监听画布移动，更新 pattern 的 transform
        const updatePatternTransform = () => {
          const outerG = svgElement.querySelector('g')
          const transform = outerG?.getAttribute('transform')
          if (transform) {
            const match = transform.match(/matrix\(([^,]+),([^,]+),([^,]+),([^,]+),([^,]+),([^)]+)\)/)
            if (match) {
              const translateX = parseFloat(match[5])
              const translateY = parseFloat(match[6])
              pattern.setAttribute('patternTransform', `translate(${translateX}, ${translateY})`)
            }
          }
        }

        // 保存函数引用，供外部调用
        updatePatternTransformFn = updatePatternTransform

        // 监听事件
        lfInstance.on('graph:transform', updatePatternTransform)
        lfInstance.on('blank:mousemove', updatePatternTransform)
        lfInstance.on('blank:mouseup', updatePatternTransform)
      }
    }
  }, 200)
}

// 获取可以进行设置的属性
const getProperty = () => {
  let props = {}
  const { nodes, edges } = lf.value.getSelectElements()
  nodes.forEach(node => {
    props = { ...props, ...node.properties }
  })
  edges.forEach(edge => {
    props = { ...props, ...edge.properties }
  })
  properties.value = props
  return props
}

const dragInNode = (type) => {
  lf.value.dnd.startDrag({
    type
  })
}

const changeNodeFill = (color) => {
  const { nodes } = lf.value.graphModel.getSelectElements()
  nodes.forEach(({ id }) => {
    lf.value.setProperties(id, {
      fill: color
    })
  })
}

const setStyle = (item) => {
  activeNodes.value.forEach(({ id }) => {
    lf.value.setProperties(id, item)
  })
  activeEdges.value.forEach(({ id }) => {
    lf.value.setProperties(id, item)
  })
  getProperty()
}

const setZIndex = (type) => {
  activeNodes.value.forEach(({ id }) => {
    lf.value.setElementZIndex(id, type)
  })
  activeEdges.value.forEach(({ id }) => {
    lf.value.setElementZIndex(id, type)
  })
}

const saveGraph = () => {
  const data = lf.value.getGraphData()
  download(filename.value, JSON.stringify(data))
}

const importGraph = (data) => {
  if (!data || !lf.value) {
    alert('导入数据无效')
    return
  }
  try {
    // 清空当前图表
    lf.value.clearData()
    // 加载新数据
    lf.value.render(data)
    // 清空选择
    activeNodes.value = []
    activeEdges.value = []
    properties.value = {}
    alert('导入成功')
  } catch (error) {
    alert('导入失败，请检查数据格式')
    console.error('导入错误:', error)
  }
}

const download = (fname, text) => {
  window.sessionStorage.setItem(fname, text)
  const element = document.createElement('a')
  element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text))
  element.setAttribute('download', fname)
  element.style.display = 'none'
  document.body.appendChild(element)
  element.click()
  document.body.removeChild(element)
}

// 初始化
let data = ''
if (window.location.search) {
  const query = window.location.search.substring(1).split('&').reduce((map, kv) => {
    const [key, value] = kv.split('=')
    map[key] = value
    return map
  }, {})
  filename.value = query.filename
  const d = window.sessionStorage.getItem(filename.value)
  if (d) {
    data = JSON.parse(d)
  }
}

onMounted(() => {
  initLogicFlow(data)

  // 初始化时进行一次微小的放大，避免缩放比例为 1.0 时网格跟随功能失效
  setTimeout(() => {
    if (lf.value) {
      lf.value.zoom(true)
    }
  }, 300)
})
</script>

<style scoped>

.diagram {
  width: 100%;
  height: 100%;
}
.diagram * {
  box-sizing: border-box;
}
.diagram-toolbar {
  position: absolute;
  top: 0;
  left: 200px;
  height: 40px;
  width: 310px;
  display: flex;
  align-items: center;
  border-bottom: 1px solid #e5e5e5;
  z-index: 10;
  background: #e5e5e5;
}
.diagram-main {
  display: flex;
  width: 100%;
  height: 100%;
}
.diagram-sidebar {
  width: 185px;
  height: calc(100% - 40px);
  border-right: 1px solid #dadce0;
  padding: 10px;
}
.diagram-panel {
  width: 300px;
  background: #fff;
  height: calc(100% - 40px);
  position: absolute;
  right: 0px;
  top: 40px;
  border-left: 1px solid #dadce0;
}
.diagram-container {
  flex: 1;
}
.diagram-wrapper {
  box-sizing: border-box;
  width: 100%;
  height: 100%;
}
.lf-diagram {
  box-shadow: 0px 0px 4px #838284;
  width: 100%;
  height: 100%;
  cursor: grab;
  user-select: none;
}
.lf-diagram:active {
  cursor: grabbing;
}
/* LogicFlow 背景层设置为透明 */
.diagram :deep(.lf-background) {
  background: transparent !important;
}
.diagram :deep(.lf-canvas-overlay) {
  background: transparent !important;
}
.diagram :deep(.lf-canvas) {
  background: transparent !important;
}
.diagram :deep(svg) {
  background: transparent !important;
}
::-webkit-scrollbar {
  width: 9px;
  height: 9px;
  background: white;
  border-left: 1px solid #e8e8e8;
}
::-webkit-scrollbar-thumb {
  border-width: 1px;
  border-style: solid;
  border-color: #fff;
  border-radius: 6px;
  background: #c9c9c9;
}
::-webkit-scrollbar-thumb:hover {
  background: #b5b5b5;
}
</style>