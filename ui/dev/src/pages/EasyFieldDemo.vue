<template>
  <q-page padding>
    <div class="row no-wrap items-start q-gutter-md q-mb-lg">
      <EasyField v-model="selectedField" v-bind="fieldPicker" />
      <InfoBoxWrapper color="primary" label="v-model" style="flex: 2;">
        <q-markdown v-if="selectedField !== 'markdown'" class="q-py-md q-px-sm" :src="modelShownAsBadge" />
        <div v-else class="q-ma-sm q-py-sm q-px-xs text-wrap-all bg-grey-2" style="width: 97%; border-radius: 0.3em">{{ modelShownAsBadge }}</div>
      </InfoBoxWrapper>
    </div>
    <q-markdown
      v-if="rawComponent.desc"
      :src="rawComponent.desc"
    />
    <InfoCard
      tag="EasyField"
      :key="selectedField"
      v-model="settings"
      :settingsSchema="settingsSchema"
    >
      <InfoBoxWrapper
        color="accent"
        label="interactive preview"
        class="q-mb-md js-interactive-preview"
      >
        <EasyField
          v-model="model"
          v-bind="field"
          :key="selectedField"
          style="width: 95%"
        />
      </InfoBoxWrapper>
    </InfoCard>
  </q-page>
</template>

<style lang="sass">
</style>

<script>
import { isString, isUndefined, isArray, isPlainObject } from 'is-what'
import copy from 'copy-anything'
import merge from 'merge-anything'
import EasyForms from 'ui'
import demoOptions from '../schemas/easyFields'
import { getInfoCardSchema, getRawComponent } from '../helpers/schemaBuilders'

const selectableFields = require
  .context('../../../src/components/fields', true, /^\.\/.*\.vue$/)
  .keys()
  .map(k => {
    const name = k.replace(/\.\/Ef/, '').replace(/\.vue/, '')
    return name[0].toLowerCase() + name.slice(1)
  })
  .sort()

export default {
  name: 'EasyFieldDemo',
  data () {
    const selectedField = 'input'
    const fieldPicker = {
      label: 'Pick a field',
      fieldType: 'select',
      emitValue: true,
      options: selectableFields,
    }
    return {
      selectedField,
      selectableFields,
      fieldPicker,
      model: '',
      settings: {},
    }
  },
  watch: {
    model (newValue, oldValue) {
      if (isUndefined(this.settings.value)) return
      this.settings.value = newValue
    },
    settings (newSettings, oldSettings) {
      if (!newSettings.value || newSettings.value === oldSettings.value) return
      this.model = newSettings.value
    },
    selectedField: {
      handler (newValue, oldValue) {
        const { addTestOptions } = this
        this.model = undefined
        this.settings.valueType = undefined
        if (newValue === oldValue) return
        const ops = copy(demoOptions[newValue])
        addTestOptions(ops)
        this.settings.label = `My awesome "${newValue}" field`
      },
      immediate: true,
    },
  },
  computed: {
    field () {
      return this.settings
    },
    rawComponent () { return getRawComponent(this.selectedField) },
    settingsSchema () { return getInfoCardSchema(this.selectedField) },
    modelShownAsBadge () {
      const { model } = this
      const parsedModel = isString(model)
        ? `"${model}"`
        : (isArray(model) || isPlainObject(model))
          ? JSON.stringify(model)
          : model
      return `\`${parsedModel}\``
    },
  },
  methods: {
    addTestOptions (ops = {}) {
      if (!ops) return
      const { value } = ops
      if (value !== undefined) {
        this.model = value
      }
      this.settings = { value }
      Object.entries(ops).forEach(([key, value]) => {
        this.$set(this.settings, key, value)
      })
    },
    log(...args) {
      console.log(...args)
    },
  },
}
</script>