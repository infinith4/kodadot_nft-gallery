<template>
  <div class="setting-chooser-wrapper">
    <NeoField :label="label">
      <b-select v-model="selected" :placeholder="label" expanded>
        <option
          v-for="option in options"
          :key="option.value"
          :value="option.value">
          {{ option.text }}
        </option>
      </b-select>
    </NeoField>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'nuxt-property-decorator'

@Component({})
export default class SettingChooserDevelopment extends Vue {
  @Prop() public label!: string
  @Prop() public selector!: string
  @Prop() public setter!: string

  get options() {
    return this.$store.state.development[this.selector]
  }

  get selected() {
    return this.$store.state.development[this.selector].value
  }

  set selected(value) {
    this.$store.commit(this.setter, { status: value })
  }

  public async mounted() {
    this.$store.commit('setDevelopment', {
      status: this.$store.state.development.status || false,
      options: [
        { text: 'Hide Dev Accounts & Refresh', value: false },
        { text: 'Show Dev Accounts & Refresh', value: true },
      ],
    })
  }
}
</script>
<style scoped>
.setting-chooser-wrapper {
  margin-top: 1em;
}
</style>
