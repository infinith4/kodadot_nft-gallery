<template>
  <NeoField grouped>
    <p v-if="!disabled" class="control">
      <b-button size="is-medium" icon-left="times" @click="remove" />
    </p>
    <NeoField label="Trait" expanded>
      <b-input
        v-model="vKey"
        placeholder="Background"
        expanded
        :disabled="disabled"
        spellcheck="true" />
    </NeoField>
    <NeoField label="Value" expanded>
      <b-input
        v-model="vValue"
        placeholder="Dark"
        expanded
        spellcheck="true"
        :disabled="disabled" />
    </NeoField>
  </NeoField>
</template>

<script lang="ts">
import { Component, Emit, Prop, PropSync, Vue } from 'nuxt-property-decorator'

@Component
export default class AttributeInput extends Vue {
  @Prop(Number) index!: number
  @Prop(Boolean) disabled!: boolean
  @PropSync('trait_type', { type: String }) vKey!: string
  @PropSync('value', { type: [String, Number] }) vValue!: string | number

  @Emit('remove')
  protected remove() {
    return this.index
  }
}
</script>
