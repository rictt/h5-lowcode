<template>
  <div class="form-wrapper">
    <van-form @submit="onSubmit">
      <template
        v-for="(e) in items"
      >
        <van-field
          v-if="e.ui === 'van-field'"
          v-model="model[e.key]"
          :rules="e.rules"
          :is="e.ui"
          v-bind="{ ...e }"
          :key="e.key"
        />
        <van-field
          v-else-if="clickableList.includes(e.ui)"
          v-model="model[e.key]"
          readonly
          clickable
          @click="onOpenPicker(e)"
          :rules="e.rules"
          :label="e.label"
          :key="e.key"
        />
        <CheckboxGroup
          v-model="model[e.key]"
          v-else-if="e.ui === 'van-checkbox-group'"
          v-bind="e"
          :key="e.key"
        />
      </template>
      <!-- 统一的选择弹框 -->
      <van-popup v-model:show="picker.show" position="bottom">
        <van-datetime-picker
          v-if="picker && picker.type"
          :type="picker.type"
          @confirm="onConfirmTimePicker(picker.key, $event)"
          @cancel="onCancelPicker"
        />
        <van-picker
          v-if="picker.columns && picker.columns.length"
          :columns="picker.columns"
          :columns-field-names="picker.columnsFieldNames"
          @confirm="onConfirmPicker(picker.key, $event)"
          @cancel="onCancelPicker"
        />
      </van-popup>
      <div style="margin: 16px">
        <van-button round block type="primary" native-type="submit">
          提交
        </van-button>
      </div>
    </van-form>
  </div>
</template>

<script>
import { Form, Field, Popup, Button, DatetimePicker, Switch, Checkbox, Picker } from 'vant'
import { reactive, toRefs, ref } from 'vue'
import { InjectPropsListMap, Data, Props } from './index'
import axios from 'axios'
import CheckboxGroup from './checkbox-group.vue'

export default {
  tplData: { ...Data },
  tplProps: { ...Props },
  name: 'TplForm',
  components: {
    CheckboxGroup,
    [Form.name]: Form,
    [Field.name]: Field,
    [Button.name]: Button,
    [DatetimePicker.name]: DatetimePicker,
    [Switch.name]: Switch,
    [Checkbox.name]: Checkbox,
    [Popup.name]: Popup,
    [Picker.name]: Picker
  },
  props: {
    ...InjectPropsListMap
  },
  setup(props) {
    const { env, url } = toRefs(props)
    const data = reactive({
      model: {},
      picker: {
        show: false,
        key: null,
        type: '',
        columns: [],
        columnsFieldNames: {
          text: 'label',
          values: 'values',
          children: 'children'
        }
      }
    })
    const dataAsRefs = toRefs(data)
    const clickableList = ref(['van-datetime-picker', 'van-picker'])

    const onSubmit = (e) => {
      const host = env.value + url.value
      console.log('[Request Host]:', host)
      console.log('[Request Model]:', data.model)
      axios.get(host, { params: data.model })
        .then(res => {
          console.log('[Request Result]:')
          console.log(res)
        })
    }

    const onOpenPicker = (item) => {
      const { key, type, columns } = item
      data.picker.show = true
      data.picker.key = key
      data.picker.type = type
      data.picker.columns = columns
      if (type) {
        data.picker.columns = []
      }
      if (columns) {
        data.picker.type = ''
      }
    }

    const onConfirmTimePicker = (key, val) => {
      if (typeof val === 'object') {
        val = val.toLocaleDateString()
      }
      data.model[key] = val
      data.picker.show = false
    }

    // TODO: 选择器列表数据显示，选中后的回显，model绑定名称的统一
    const onConfirmPicker = (key, val) => {
      data.model[key] = val
      data.picker.show = false
    }
    
    const onCancelPicker = () => {
      data.picker.show = false
      data.picker.key = ''
      data.picker.type = ''
      data.picker.columns = []
    }

    return {
      ...dataAsRefs,
      onSubmit,
      onConfirmTimePicker,
      onConfirmPicker,
      onCancelPicker,
      onOpenPicker,
      clickableList
    }
  }
}
</script>

<style>
</style>
