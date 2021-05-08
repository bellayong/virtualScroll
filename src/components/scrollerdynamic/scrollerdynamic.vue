<!--结合scrollerdynamicitem实现不定高虚拟滚动-->
<template>
  <x-scrollerrecycle
    ref="scroller"
    :items="itemsWithSize"
    :min-item-size="minItemSize"
    @scrollChange="scrollChange">
    <template slot-scope="{ item: itemWithSize, index }">
      <slot v-bind="{
                      item: itemWithSize.item,
                      index,
                      itemWithSize
                    }"
      />
    </template>
  </x-scrollerrecycle>

</template>

<script>
  import scrollerrecycle from '../scrollerrecycle/scrollerrecycle'

  export default {
    name: 'x-scrollerdynamic',
    inheritAttrs: false,
    components: {
      'x-scrollerrecycle': scrollerrecycle,
    },
    provide () {
      return {
        vscrollData: this.vscrollData,
        vscrollParent: this,
      }
    },
    props: {
      items: {
        type: Array,
        required: true,
      },
      minItemSize: {
        type: Number,
      },
      height: Number,
    },
    data () {
      return {
        vscrollData: {
          sizes: {},
        },
      }
    },
    computed: {
      itemsWithSize () {
        const result = []
        const {items} = this
        const sizes = this.vscrollData.sizes

        for (let i = 0; i < items.length; i++) {
          const item = items[i]
          const id = i
          let size = sizes[id] || 0

          result.push({
            item,
            id,
            size,
          })
        }
        return result
      },
    },
    watch: {
      items () {
        this.scrollChange()
      },
    },
    created () {
    },
    mounted () {
    },
    methods: {
      scrollChange () {
        this.$emit('scroll:change')
      }
    }
  }
</script>
