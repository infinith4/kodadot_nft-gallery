<template>
  <NeoField>
    <b-taginput
      v-model="tags"
      :data="allTags"
      ellipsis
      icon="tag"
      placeholder="Search by type"
      aria-close-label="Delete this tag"
      autocomplete
      open-on-focus
      type="is-primary"
      @input="handleInput" />
  </NeoField>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from 'nuxt-property-decorator'

@Component({})
export default class TypeTagInput extends Vue {
  private allTags: string[] = ['audio', 'video', 'image', 'gif', 'svg']
  @Prop() public value!: string

  get tags() {
    return this.value ? this.value.split('|') : []
  }

  set tags(value: string[]) {
    this.handleInput(value)
  }

  @Emit('input')
  handleInput(value: string[]) {
    return value.join('|')
  }
}
</script>
