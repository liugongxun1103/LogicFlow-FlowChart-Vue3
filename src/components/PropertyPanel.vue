<template>
  <div class="diagram-panel">
    <div class="setting-block">
      <div>快捷样式</div>
      <div class="short-styles">
        <div
          v-for="(item, index) in shortStyles"
          :key="index"
          :style="{ 'backgroundColor': item.backgroundColor, 'borderColor': item.borderColor, 'borderWidth': item.borderWidth }"
          @click="setStyle(item)"
        >
        </div>
      </div>
    </div>
    <div class="setting-block">
      <div class="setting-item">
        <span>背景色</span>
        <input
          type="color"
          :value="rgbaToHex(style.backgroundColor)"
          @input="(e) => changeColorFromHex(e.target.value, 'backgroundColor')"
          class="color-input"
        />
        <span>背景渐变色</span>
        <input
          type="color"
          :value="rgbaToHex(style.gradientColor)"
          @input="(e) => changeColorFromHex(e.target.value, 'gradientColor')"
          class="color-input"
        />
      </div>
      <div class="setting-item">
        <span>线条样式</span>
        <el-select v-model="style.borderStyle" size="small" @change="selectBorder">
          <el-option value="hidden" label="不显示"></el-option>
          <el-option
            v-for="(border, index) in borderStyles"
            :value="border.value"
            :key="index"
          >
            <div class="border-style" :style="{ 'borderBottomStyle': border.value}"></div>
          </el-option>
        </el-select>
      </div>
      <div class="setting-item">
        <span>线条颜色</span>
        <input
          type="color"
          :value="rgbaToHex(style.borderColor)"
          @input="(e) => changeColorFromHex(e.target.value, 'borderColor')"
          class="color-input"
        />
      </div>
      <div class="setting-item">
        <span>线条宽度</span>
        <el-select v-model="style.borderWidth" @change="changeBorderWidth">
          <el-option
            v-for="item in borderWidthOptions"
            :key="item"
            :label="`${item}px`"
            :value="item"
          ></el-option>
        </el-select>
      </div>
      <div class="setting-item">
        <span>文本颜色</span>
        <input
          type="color"
          :value="rgbaToHex(style.fontColor)"
          @input="(e) => changeColorFromHex(e.target.value, 'fontColor')"
          class="color-input"
        />
      </div>
      <div class="setting-item">
        <span>文本大小</span>
        <el-input-number
          v-model="style.fontSize"
          controls-position="right"
          size="mini"
          @change="changeFontSize"
          :min="12"
          :max="30">
        </el-input-number>
        <span>px</span>
      </div>
      <div class="setting-item">
        <span>文本字体</span>
        <el-select v-model="style.fontFamily" size="small" @change="changeFontFamily">
          <el-option
            v-for="(fontFamily, index) in fontFamilies"
            :value="fontFamily.value"
            :key="index"
          ></el-option>
        </el-select>
      </div>
      <div class="setting-item">
        <span>行高</span>
        <el-select v-model="style.lineHeight" size="small" @change="changeLineHeight">
          <el-option
            v-for="(item, index) in lineHeightOptions"
            :key="index"
            :label="`${item}`"
            :value="item"
          ></el-option>
        </el-select>
      </div>
      <div class="setting-item">
        <span>对齐</span>
        <el-radio-group v-model="style.textAlign" size="small" @change="changeTextAlign">
          <el-radio-button label="left">左对齐</el-radio-button>
          <el-radio-button label="center">居中</el-radio-button>
          <el-radio-button label="right">右对齐</el-radio-button>
        </el-radio-group>
      </div>
      <div class="setting-item">
        <span>文本样式</span>
        <el-button size="small" @click="changeFontWeight">B</el-button>
        <el-button size="small" @click="changeTextDecoration">U</el-button>
        <el-button size="small" @click="changeFontStyle">I</el-button>
      </div>
      <div>
        <el-button @click="emit('setZIndex', 'top')">置为顶部</el-button>
        <el-button @click="emit('setZIndex', 'bottom')">置为底部</el-button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { shortStyles, borderStyles, fontFamilies } from '../constant'

const props = defineProps({
  elementsStyle: Object,
  onlyEdge: Boolean // 是否是只设置边的属性，当只设置边的属性时，隐藏快捷样式和背景色设置
})

const emit = defineEmits(['setStyle', 'setZIndex'])

const style = ref({
  backgroundColor: 'rgba(255, 255, 255, 1)', // 填充色
  gradientColor: 'rgba(255, 255, 255, 1)', // 渐变色
  borderType: 0, // 线条类型
  borderColor: 'rgba(0, 0, 0, 1)', // 填充颜色
  borderWidth: 1, // 线条宽度
  borderStyle: '', // 线条类型
  fontSize: 12, // 文本大小
  fontColor: 'rgba(0, 0, 0, 1)', // 文本颜色
  fontWeight: '', // 文本加粗
  fontFamily: '', // 文本样式
  lineHeight: '', // 行高
  textAlign: '' // 对齐
})

const borderWidthOptions = ref(Array(11).fill().map((_, i) => i))
const fontWeight = ref('')
const lineHeightOptions = ref(Array(5).fill(1).map((_, i) => _ + i * 0.5))

watch(
  () => props.elementsStyle,
  (val) => {
    if (!val || Object.keys(val).length === 0) return
    // 只合并有效的值，避免 undefined 覆盖默认值
    const validProps = {}
    Object.keys(val).forEach(key => {
      if (val[key] !== undefined && val[key] !== null && val[key] !== '') {
        validProps[key] = val[key]
      }
    })
    if (Object.keys(validProps).length > 0) {
      style.value = { ...style.value, ...validProps }
    }
  },
  { immediate: false }
)

// 将 RGBA 颜色转换为 HEX 格式（用于 HTML5 color input）
const rgbaToHex = (rgba) => {
  if (!rgba) return '#ffffff'

  // 解析 rgba 字符串
  const match = rgba.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)/)
  if (!match) return '#ffffff'

  const r = parseInt(match[1]).toString(16).padStart(2, '0')
  const g = parseInt(match[2]).toString(16).padStart(2, '0')
  const b = parseInt(match[3]).toString(16).padStart(2, '0')

  return `#${r}${g}${b}`
}

// 将 HEX 颜色转换为 RGBA 格式
const hexToRgba = (hex) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return `rgba(${r},${g},${b},1)`
}

const changeColorFromHex = (hex, type) => {
  const color = hexToRgba(hex)
  style.value[type] = color
  emit('setStyle', {
    [type]: color
  })
}

const setStyle = (item) => {
  emit('setStyle', item)
}

const selectBorder = (val) => {
  emit('setStyle', {
    borderStyle: val
  })
}

const changeFontSize = (val) => {
  emit('setStyle', {
    fontSize: val
  })
}

const changeBorderWidth = (val) => {
  emit('setStyle', {
    borderWidth: val
  })
}

const changeFontFamily = (val) => {
  emit('setStyle', {
    fontFamily: val
  })
}

const changeLineHeight = (val) => {
  emit('setStyle', {
    lineHeight: val
  })
}

const changeFontWeight = () => {
  if (style.value.fontWeight === 'bold') {
    emit('setStyle', {
      fontWeight: 'normal'
    })
  } else {
    emit('setStyle', {
      fontWeight: 'bold'
    })
  }
}

const changeTextDecoration = () => {
  if (style.value.textDecoration === 'underline') {
    emit('setStyle', {
      textDecoration: 'none'
    })
  } else {
    emit('setStyle', {
      textDecoration: 'underline'
    })
  }
}

const changeFontStyle = () => {
  if (style.value.fontStyle === 'italic') {
    emit('setStyle', {
      fontStyle: 'normal'
    })
  } else {
    emit('setStyle', {
      fontStyle: 'italic'
    })
  }
}

const changeTextAlign = (val) => {
  emit('setStyle', {
    textAlign: val
  })
}
</script>

<style scoped>
.diagram-panel {
  padding: 20px;
  overflow-y: auto;
  max-height: 100%;
}
.short-styles {
  width: 240px;
}
.short-styles > div {
  width: 20px;
  height: 20px;
  margin: 0 10px 5px 0;
  box-sizing: border-box;
  float: left;
  border: 1px solid #fff;
  cursor: pointer;
}
.border-style {
  width: 150px;
  height: 0px;
  margin-top: 18px;
  border-bottom-width: 1px;
  border-bottom-color: black;
}
.setting-block {
  overflow: hidden;
}
.setting-item {
  line-height: 12px;
  font-size: 12px;
  display: flex;
  vertical-align: middle;
  align-items: center;
  margin-top: 10px;
  flex-wrap: wrap;
}
.setting-item > span {
  width: 50px;
  margin-right: 10px;
  text-align: right;
  flex-shrink: 0;
  flex-grow: 0;
}
.color-input {
  width: 40px;
  height: 30px;
  border: 1px solid #eaeaeb;
  cursor: pointer;
  margin-right: 10px;
}
</style>
