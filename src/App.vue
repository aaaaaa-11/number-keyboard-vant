<template>
  <label class="flex">
    <span class="leading-[24px] pt-[10px]">身份证号：</span>
    <NumberKeyboard
      name="idcard"
      vKey="1"
      :currentKey="currentKey"
      @keyboardFocus="keyboardFocus"
      v-model:value="formState.idcard"
      :maxlength="18"
      placeholder="请输入身份证号"
      :rules="[
        {
          pattern: /^(\d{15}|\d18|^\d{17}(\d|X|x))$/,
          message: '请填写正确的身份证号码'
        }
      ]"
      :customState="{
        theme: 'custom',
        extraKey: 'X',
        closeButtonText: '完成'
      }"
    />
  </label>
  <label class="flex">
    <span class="leading-[24px] pt-[10px]">手机号：</span>
    <NumberKeyboard
      name="phone"
      vKey="2"
      :currentKey="currentKey"
      @keyboardFocus="keyboardFocus"
      v-model:value="formState.phone"
      :maxlength="11"
      placeholder="请输入手机号"
      :rules="[
        {
          pattern: /^1[3-9][0-9]{9}$/,
          message: '请填写正确的手机号码'
        }
      ]"
    />
  </label>
  <label class="flex">
    <span class="leading-[24px] pt-[10px]">薪资：</span>
    <NumberKeyboard
      name="salary"
      vKey="3"
      :currentKey="currentKey"
      @keyboardFocus="keyboardFocus"
      v-model:value="formState.salary"
      placeholder="请输入薪资"
      :customState="{
        theme: 'custom',
        extraKey: '.',
        closeButtonText: '完成'
      }"
    />
  </label>
  <hr />
  <p>表单内容：{{ formState }}</p>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import NumberKeyboard from './components/numberKeyboard/index.vue'

type SN = string | null

interface FormState {
  idcard: SN
  phone: SN
  salary: number | SN
}

// 表单字段
const formState = reactive<FormState>({
  idcard: null,
  phone: null,
  salary: null
})

// 用来标识当前聚焦的输入框，并隐藏其他输入框键盘和光标
const currentKey = ref<string>()

// 有一个输入框聚焦，需要关闭其他输入框键盘和光标
const keyboardFocus = (key: string) => (currentKey.value = key)
</script>

<style></style>
