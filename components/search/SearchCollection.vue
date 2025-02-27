<template>
  <div class="content field-group-container">
    <NeoField grouped group-multiline>
      <Sort
        class="control"
        :value="sortBy"
        :is-collection="true"
        :sort-option="sortOption"
        @input="updateSortBy" />
      <div
        class="is-flex layout-search"
        :class="{
          'is-flex-grow-1 ': !hideSearch,
        }">
        <NeoField v-if="!hideSearch" expanded class="control">
          <b-input
            v-model="searchQuery"
            placeholder="Search..."
            type="search"
            icon="search"
            expanded
            class="input-search">
          </b-input>
        </NeoField>
        <BasicSwitch
          v-if="!isMoonRiver"
          v-model="vListed"
          class="is-flex control mb-5"
          :label="!replaceBuyNowWithYolo ? 'sort.listed' : 'YOLO'"
          size="is-medium"
          label-color="has-text-success"
          :disabled="disableToggle"
          :message="$t('tooltip.buy')" />
        <BasicSwitch
          v-if="showOwnerSwitch"
          v-model="vOwned"
          class="is-flex control mb-5"
          :label="'sort.own'"
          size="is-medium"
          label-color="has-text-success"
          :message="$t('tooltip.own')" />
        <slot />
      </div>
    </NeoField>
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, mixins } from 'nuxt-property-decorator'
import { Debounce } from 'vue-debounce-decorator'
import { exist } from './exist'
import KeyboardEventsMixin from '~/utils/mixins/keyboardEventsMixin'
import { usePreferencesStore } from '@/stores/preferences'

@Component({
  components: {
    Sort: () => import('./SearchSortDropdown.vue'),
    TypeTagInput: () => import('./TypeTagInput.vue'),
    Pagination: () => import('@/components/rmrk/Gallery/Pagination.vue'),
    BasicSwitch: () => import('@/components/shared/form/BasicSwitch.vue'),
  },
})
export default class SearchCollection extends mixins(KeyboardEventsMixin) {
  @Prop(String) public search!: string
  @Prop(String) public type!: string
  @Prop({ type: String, default: 'blockNumber_DESC' }) public sortBy!: string
  @Prop({ type: Boolean, default: false }) public listed!: boolean
  @Prop({ type: Boolean, default: false }) public owned!: boolean
  @Prop(Boolean) public disableToggle!: boolean
  @Prop(Boolean) public hideSearch!: boolean
  @Prop(Boolean) public isMoonRiver!: boolean
  @Prop(Boolean) public showOwnerSwitch!: boolean
  @Prop(Array) public sortOption?: string[]
  protected isVisible = false
  private preferencesStore = usePreferencesStore()

  public mounted(): void {
    exist(this.$route.query.search, this.updateSearch)
    exist(this.$route.query.type, this.updateType)
    exist(this.$route.query.sort, this.updateSortBy)
    exist(this.$route.query.listed, this.updateListed)
    exist(this.$route.query.owned, this.updateOwned)
  }

  public created() {
    this.initKeyboardEventHandler({
      f: this.bindFilterEvents,
    })
  }

  private bindFilterEvents(event) {
    switch (event.key) {
      case 'b':
        this.updateListed(!this.vListed)
        break
      case 'n':
        this.updateSortBy(this.sortOption?.[0] || '')
        break
      case 'o':
        this.updateSortBy(this.sortOption?.[1] || '')
        break
      case 'a':
        this.updateSortBy(this.sortOption?.[2] || '')
        break
      case 'e':
        this.updateSortBy(this.sortOption?.[3] || '')
        break
    }
  }

  get vListed(): boolean {
    return this.listed
  }

  set vListed(listed: boolean) {
    this.updateListed(listed)
  }

  get vOwned(): boolean {
    return this.owned
  }

  set vOwned(owned: boolean) {
    this.updateOwned(owned)
  }

  get searchQuery(): string {
    return this.search
  }

  set searchQuery(value: string) {
    this.updateSearch(value)
  }

  get typeQuery(): string {
    return this.type
  }

  set typeQuery(value: string) {
    this.updateType(value)
  }

  get replaceBuyNowWithYolo(): boolean {
    return this.preferencesStore.getReplaceBuyNowWithYolo
  }

  @Emit('update:listed')
  @Debounce(50)
  updateListed(value: string | boolean): boolean {
    const queryValue = String(value) === 'true'
    this.replaceUrl(queryValue, 'listed')
    return queryValue
  }

  @Emit('update:owned')
  @Debounce(50)
  updateOwned(value: string | boolean): boolean {
    const queryValue = value ? String(value) : ''
    this.replaceUrl(queryValue, 'owned')
    return Boolean(queryValue)
  }

  @Emit('update:type')
  @Debounce(50)
  updateType(value: string): string {
    this.replaceUrl(value, 'type')
    return value
  }

  @Emit('update:sortBy')
  @Debounce(400)
  updateSortBy(value: string): string {
    const listed = Boolean(value?.toLowerCase().indexOf('price') > -1)
    if (listed && !this.vListed) {
      this.vListed = true
    }

    this.replaceUrl(value, 'sort')
    return value
  }

  @Emit('update:search')
  @Debounce(400)
  updateSearch(value: string): string {
    value !== this.searchQuery && this.replaceUrl(value)
    return value
  }

  @Debounce(100)
  replaceUrl(value: boolean | string, key = 'search'): void {
    this.$router
      .replace({
        path: String(this.$route.path),
        query: {
          ...this.$route.query,
          search: this.searchQuery || undefined,
          [key]: value ? String(value) : undefined,
        },
      })
      .catch(this.$consola.warn /*Navigation Duplicate err fix later */)
  }
}
</script>

<style scoped lang="scss">
@import '@/styles/abstracts/variables';

.card {
  box-shadow: 0px 0px 5px 0.5px $primary;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

<style lang="scss">
/* cry in scss (global) */
.field-group-container {
  .is-grouped-multiline {
    flex-wrap: initial !important;
    justify-content: space-between;
    .layout-search {
      @media screen and (max-width: 768px) {
        flex-wrap: wrap !important;
        gap: 10px;
      }
    }
    @media screen and (max-width: 768px) {
      flex-wrap: wrap !important;
    }
  }
  .input-search {
    input {
      border: 1px solid #7d7d7d !important;
    }
  }
}
</style>
