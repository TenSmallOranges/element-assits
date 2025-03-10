<template>
<ul ref="ul" class="ea-select-panel">
  <div
    v-if="!root"
    ref="div"
    class="ea-select-container"
    :class="{[endSize]: 1}" />
  <li
    v-for="item in cloneOptions"
    :key="item.value"
    class="ea-select-panel__item"
    :class="{ active: !notActive && item.show, [item.colorClass || '']: 1 }"
    @click="handleSelect(item)"
    @mouseenter="handleEnter(item)">
    <span>
      <i :class="item.icon" />
      {{ item.label }}
    </span>
    <template v-if="item.children">
      <i class="el-icon-arrow-right" />
      <SelectPanel
        v-if="item.show"
        :root="root || $refs.ul.parentElement"
        :options="item.children"
        @select="$emit('select', $event)"
        @open="$emit('open', $event)" />
    </template>
  </li>
</ul>
</template>

<script>
import { cloneDeep, isEmpty } from 'lodash-es'
export default {
  name: 'SelectPanel',
  props: {
    options: {
      type: Array,
      required: true
    },
    root: {
      type: HTMLElement,
      default: undefined
    },
    size: {
      type: String,
      default: undefined
    },
    notActive: {
      type: Boolean,
      default: false
    }
  },
  data () {
    const cloneOptions = cloneDeep(this.options)
    cloneOptions.forEach(m => m.show = false)
    const endSize = this.size || (this.$ELEMENT && this.$ELEMENT.size) || 'small'
    return {
      endSize,
      cloneOptions
    }
  },
  mounted () {
    if (!this.root) {
      this.$el.parentElement.appendChild(this.$refs.div)
      this.$refs.div.appendChild(this.$el)
    } else {
      this.root.appendChild(this.$el)
    }
  },
  methods: {
    async handleEnter (item) {
      const target = !item.show
      this.cloneOptions.forEach(m => m.show = false)
      item.show = target
      await this.$nextTick()
      if (target && !isEmpty(item.children)) this.$emit('open', item)
    },
    async handleSelect (item) {
      if (!isEmpty(item.children)) return void(0)
      this.$emit('select', item)
    }
  }
}
</script>

<style lang="scss">
.ea-select-container {
  display: flex;
  justify-content: flex-start;
  font-size: 12px;
  &.medium {
    font-size: 14px;
    .ea-select-panel .ea-select-panel__item {
      line-height: 28px;
    }
  }
  .ea-select-panel {
    padding: 0;
    margin: 0;
    list-style: none;
    flex-grow: 1;
    min-width: 120px;
    & + .ea-select-panel {
      border-left: 1px solid #DDD;
    }
    &__item {
      width: 100%;
      line-height: 24px;
      cursor: pointer;
      padding: 0 12px;
      box-sizing: border-box;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: inherit;
      &:hover {
        background-color: $--color-border-lighter;
      }
      &.active {
        color: $--color-primary;
        font-weight: bold;
      }
    }
  }
}
</style>
