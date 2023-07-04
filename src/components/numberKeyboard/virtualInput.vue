<template>
  <div class="virtual-input" @touchstart.stop="clickInput(inputValueArr.length)">
    <div class="input-wrap absolute -z-1 virtual-input-placeholder" v-if="!inputValue">
      {{ placeholder }}
    </div>
    <div class="h-full w-full top-0 left-0 z-10 input-wrap">
      <div
        :class="item.class"
        v-for="(item, index) in inputValueArr"
        :key="item.id"
        @touchstart.stop="clickInput(index)"
      >
        {{ item.value }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

interface InputItem {
  value: string
  class: string
  id: number
}

const props = defineProps<{
  showCursor?: boolean
  inputValue?: string
  placeholder?: string
}>()

const emit = defineEmits<{
  (e: 'update:inputValue', value: string | null)
  (e: 'showKeyboard', name: string | null)
  (e: 'hideKeyboard')
}>()

let id = 0 // input-item id

const cursorIndex = ref<number>(-1) // 光标位置

// 将输入框内的元素转换为字符串
const getInputStr = () => {
  return inputValueArr.value.map((i: InputItem) => i.value).join('')
}

// 将字符串转换为输入框元素
const getInputElArr = () => {
  if (!props.inputValue) return []
  return props.inputValue.split('').map((str: string) => ({
    value: str,
    class: 'input-item',
    id: id++
  }))
}

// 输入框内的元素（各种值、光标等元素）
const inputValueArr = ref<InputItem[]>(getInputElArr())

// 增加光标元素
const formatInput = () => {
  inputValueArr.value = getInputElArr()
  inputValueArr.value.splice(cursorIndex.value, 0, {
    value: '',
    class: 'cursor',
    id: id++
  })
  const afterCursorEl = inputValueArr.value[cursorIndex.value + 1]
  if (afterCursorEl) {
    afterCursorEl.class = 'input-item input-item-after-cursor'
  }
}

// 点击输入框，显示光标
const clickInput = (index: number) => {
  cursorIndex.value = index
  formatInput()
  emit('showKeyboard', name)
}

// 添加数字、点等值
const addKey = (key: string) => {
  const insertIndex = cursorIndex.value // 在光标位置插入
  inputValueArr.value.splice(insertIndex, 0, {
    value: key,
    class: 'input-item',
    id: id++
  })
  cursorIndex.value++ // 光标向后移动一位
  const inputStr = getInputStr()
  emit('update:inputValue', inputStr)
}

// 删除数字、点等值
const delKey = () => {
  if (cursorIndex.value === 0) return // 没有数了，不能删

  inputValueArr.value.splice(cursorIndex.value - 1, 1)
  cursorIndex.value--
  emit('update:inputValue', getInputStr())
}

watch(
  () => props.showCursor,
  (val: boolean) => {
    if (!val) {
      // 去掉光标（如每次关闭键盘时）
      inputValueArr.value = getInputElArr()
    }
  },
  {
    immediate: true
  }
)

defineExpose({
  addKey,
  delKey
})
</script>

<style scoped lang="scss">
$cursorWidth: 2px;
$inputItemWidth: 10px;
.virtual-input {
  width: 100%;
  overflow: auto;
  padding: var(--van-cell-vertical-padding) var(--van-cell-horizontal-padding) 0;
  line-height: var(--van-cell-line-height);
  .input-wrap {
    display: flex;
    height: 100%;
    height: 24px;
    .input-item {
      height: 100%;
      width: $inputItemWidth;
      text-align: center;
      padding-left: $cursorWidth;
      box-sizing: border-box;
      &-after-cursor {
        padding-left: 0px;
        width: $inputItemWidth - $cursorWidth;
      }
    }
  }
  .cursor {
    border-left: $cursorWidth solid red;
    height: 100%;
    width: 0;
    animation: flash 0.4s ease-out infinite;
  }

  @keyframes flash {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
}
</style>
