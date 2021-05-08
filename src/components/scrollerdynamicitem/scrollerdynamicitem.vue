<!--结合scrollerdynamic实现不定高虚拟滚动，该部分负责当行元素高度的获取，以及实时触发外部容器更新-->
<script>
  export default {
    name: 'x-scrollerdynamicitem',
    inheritAttrs: false,
    inject: [
      'vscrollData',
      'vscrollParent',
    ],
    components: {},
    props: {
      item: {
        required: true,
      },
      index: {
        type: Number,
        default: undefined,
      },
    },
    data () {
      return {}
    },
    computed: {
      id () {
        return this.index
      },
      size () {
        return this.vscrollData.sizes[this.id] || 0
      },
    },
    watch: {},
    created () {
      this.vscrollParent.$on('scroll:change', this.onVscrollUpdate)
    },
    mounted () {
      this.updateSize()
    },
    render () {
      return (
        this.$slots.default
      )
    },
    methods: {
      updateSize () {
        this.$nextTick(() => {
          // 计算原生的上下margin
          // let marginTop = window.getComputedStyle(this.$el).marginTop
          // let marginBottom = window.getComputedStyle(this.$el).marginBottom
          //
          // marginTop = Number(marginTop.indexOf('px') > -1 ? marginTop.split('px')[0] : marginTop)
          // marginBottom = Number(marginBottom.indexOf('px') > -1 ? marginBottom.split('px')[0] : marginBottom)

          const height = this.$el.getBoundingClientRect().height//this.$el.offsetHeight
          this.applySize(height)
        })
      },
      applySize (height) {
        const size = height
        if (size && this.size !== size) {
          this.$set(this.vscrollData.sizes, this.id, size)
        }
      },
      onVscrollUpdate () {
        this.updateSize()
      },
    },
  }
</script>
