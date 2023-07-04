<template>
  <div class="relative flex-1">
    <van-field ref="field" :rules="rules" readonly :name="name" v-model="inputValue" />
    <virtual-input
      class="absolute left-0 top-0 z-10 bg-white"
      :placeholder="placeholder"
      ref="vInput"
      v-model:inputValue="inputValue"
      :showCursor="cursorVisible"
      @showKeyboard="showKeyboard"
    />
    <van-number-keyboard
      :show="keyboardVisible"
      :theme="customState.theme"
      :extra-key="customState.extraKey"
      :close-button-text="customState.closeButtonText"
      @input="addKey"
      @delete="delKey"
      @blur="hideKeyboard"
    />
  </div>
</template>

<script setup lang="ts">
// import VirtualInput from "./virtualInput.vue"
import type { FieldRule } from 'vant'
import { ref, computed, toRaw, nextTick, watch } from 'vue'

interface CustomState {
  theme?: string
  extraKey?: string[]
  closeButtonText?: string
}

const props = withDefaults(
  defineProps<{
    value?: string | number
    rules?: FieldRule[]
    required?: boolean
    placeholder?: string
    maxlength?: number
    name?: string
    vKey: string // 当前输入框的唯一值
    currentKey?: string // 标识当前聚焦的输入框，用来关闭其他输入框键盘和光标
    customState?: CustomState
  }>(),
  {
    customState: {
      theme: 'default',
      extraKey: '',
      closeButtonText: '-'
    }
  }
)

const emit = defineEmits<{
  (e: 'update:value', value: string | number): void
}>()

// 光标是否显示
const cursorVisible = ref<boolean>(false)
// 虚拟键盘是否显示
const keyboardVisible = ref<boolean>(false)
// 真正的输入框元素，用来调用validate等方法
const field = ref<HTMLElement>()
// 虚拟输入框元素
const vInput = ref<HTMLElement>()

// 与父组件绑定的输入值
const inputValue = computed({
  get() {
    return props.value
  },
  set(newValue: string | number) {
    emit('update:value', newValue)
  }
})

// 显示虚拟键盘
const showKeyboard = () => {
  cursorVisible.value = true
  keyboardVisible.value = true
  emit('keyboardFocus', props.vKey)
}

// 隐藏虚拟键盘
const hideKeyboard = () => {
  keyboardVisible.value = false
  cursorVisible.value = false
  const { name, rules } = props
  nextTick(() => {
    name && rules && field.value.validate()
  })
}

// 键盘input事件
const addKey = (key: string) => {
  if (inputValue.value?.length >= props.maxlength) return
  vInput.value.addKey(key)
}

// 键盘delete事件
const delKey = () => {
  vInput.value.delKey()
}

// 切换输入框时，关闭其他输入框键盘和光标
watch(
  () => props.currentKey,
  (newKey?: string, oldKey?: string) => {
    if (oldKey && newKey !== props.vKey) {
      hideKeyboard()
    }
  }
)
</script>
