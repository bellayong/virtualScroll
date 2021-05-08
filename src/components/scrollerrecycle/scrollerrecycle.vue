<template>
  <div class="xm-scrollerrecycle" :style="`height:${height}px;overflow: ${useScrollPlug?'hidden':'auto'};`"
       ref="scroll" @scroll="onScroll">
    <div class="xm-scrollerrecycle__lists" :style="'height:'+totalSize+'px'">
      <div class="xm-scrollerrecycle__item" v-for="(item,index) in pool" v-bind:key="index"
           :style="'transform: translateY('+item.position+'px);'">
        <slot :item="item.item" :index="item.nr.index"/>
      </div>
    </div>
  </div>
</template>

<script>
  // 实现虚拟滚动（定高、不定高）的核心逻辑，以及IScroll滚动逻辑
  import IScroll from 'iscroll/build/iscroll-probe'

  //iscroll插件滚动时需要阻止默认事件，这里是对对应dom容器touchmove事件阻止默认行为的管理
  const touchMoveEventMgr = {
    setShouldPeventDefault (value) {
      this.shouldPreventDefault = value
    },

    add (options) {
      Object.assign(this, options)

      this.handler = (event) => {
        if (this.shouldPreventDefault) {
          event.preventDefault()
        }
      }

      this.el.addEventListener('touchmove', this.handler, false)
    },

    remove () {
      this.el.removeEventListener('touchmove', this.handler)
    },
  }
  let uid = 0

  export default {
    name: 'x-scrollerrecycle',
    inheritAttrs: false,
    props: {
      items: {
        type: Array,
        default () {
          return []
        },
      },
      height: Number,
      itemSize: Number,// 每行的高度
      minItemSize: {type: Number, default: 50},// 每行最小高度
    },
    data () {
      let iosVersion = this.iOSversion()
      iosVersion = iosVersion ? iosVersion[0] : null

      let useScrollPlug = !!(iosVersion && iosVersion <= 7)

      return {
        totalSize: 0,// 总高度 根据总数据计算
        pool: [], // 用于真实渲染的列表
        scrollEvent: null,// 保存当前滚动信息
        buffer: 200, // 滚动偏移量 滚动开始、结束位置添加，以便页面滚动无缝衔接。

        // 是否试用三方滚动插件（低版本ios需使用）
        useScrollPlug: useScrollPlug
      }
    },
    computed: {
      // 没有固定高度时，根据最小高度 和 每行数据size 计算得到包含每条数据高度信息的数组
      sizes () {
        if (!this.itemSize) {
          const sizes = {
            '-1': {accumulator: 0},
          }
          const items = this.items
          const minItemSize = this.minItemSize
          let computedMinSize = 10000
          let accumulator = 0
          let current
          for (let i = 0, l = items.length; i < l; i++) {
            current = items[i]['size'] || minItemSize// 该项所占高度
            if (current < computedMinSize) {
              computedMinSize = current
            }
            accumulator += current// 该项及前面全部项的高度之和(根据最小高度和每行数据size计算得出)
            sizes[i] = {accumulator, size: current}
          }

          return sizes
        }
        return []
      },
    },
    watch: {
      // 影响高度的原生变化后需要更新滚动条 且在页面渲染后执行
      items: function () {
        this.$nextTick(() => {
          this.updateScroll()
        })
      },
      pool: function () {
        this.$nextTick(() => {
          this.updateScroll()
        })
      },
      totalSize: function () {
        this.$nextTick(() => {
          this.updateScroll()
        })
      },
      // 深度监听sizes变化
      sizes: {
        handler () {
          this.updateVisibleItems()
        },
        deep: true,
      },
    },
    created () {
    },
    beforeDestroy () {
      //移除全部绑定的touchmove事件
      touchMoveEventMgr.remove()
    },
    mounted () {
      this.updateVisibleItems()
      this.$nextTick(() => {
        this.initIScroll(this.$el)//初始化滚动条 且在页面渲染后执行
      })
    },
    methods: {
      // 初始化指定区域滚动条
      initIScroll (elm) {
        if (!elm || !this.useScrollPlug) {
          return
        }

        let instance = new IScroll(elm, {
          probeType: 3,
          scrollX: true,
          scrollY: true,
          bounce: false
        })
        let that = this
        touchMoveEventMgr.add({el: elm, shouldPreventDefault: false})
        instance.on('scroll', function () {
          let flag = !(Math.abs(this.y) >= Math.abs(this.maxScrollY))
          //当滚动条在可移动情况下禁用默认touchmove事件，反之取消
          touchMoveEventMgr.setShouldPeventDefault(flag)
          that.$emit('scrollChange')
          that.scrollEvent = this
          that.updateVisibleItems(this)
        })

        instance.on('scrollEnd', function () {
          that.$emit('scrollChange')
          that.scrollEvent = this
          that.updateVisibleItems(this)
        })

        this.scrollInstance = instance
      },

      onScroll (event) {
        if (this.useScrollPlug) { // 虚拟滚动，但使用三方滚动插件
          return
        }
        this.$emit('scrollChange')
        this.scrollEvent = {
          y: event.target.scrollTop
        }
        this.updateVisibleItems({y: event.target.scrollTop})
      },

      updateScroll () {
        if (!this.useScrollPlug) {
          return
        }
        this.scrollInstance ? this.scrollInstance.refresh() : this.initIScroll()
      },

      // 获取滚动开始、结束位置。
      getScroll (event) {
        const {$el: el} = this
        let scrollTop = event ? Math.abs(event.y) : 0

        return {
          start: scrollTop,
          end: scrollTop + el.clientHeight,// 开始位置加上容器高度
        }
      },

      updateVisibleItems (event) {
        const items = this.items

        if (!items || !items.length) {
          this.pool = []
          return
        }

        const count = items.length
        const itemSize = this.itemSize
        const pool = []
        const sizes = this.sizes
        const scroll = this.getScroll(event ? event : this.scrollEvent)
        const buffer = this.buffer
        scroll.start -= buffer
        scroll.end += buffer

        let startIndex = ~~(scroll.start / itemSize)// 四舍五入取最小整数
        let endIndex = Math.ceil(scroll.end / itemSize)// 四舍五入取最大整数

        startIndex < 0 && (startIndex = 0)
        endIndex > count && (endIndex = count)

        if (!itemSize) {
          // 二分法：根据对应元素的位置查找当前需要展示的开始、结束index
          let h
          let a = 0
          let b = count - 1
          let i = ~~(count / 2)
          let oldI

          // 查找startIndex
          do {
            oldI = i
            h = sizes[i].accumulator
            if (h < scroll.start) {
              a = i
            } else if (i < count - 1 && sizes[i + 1].accumulator > scroll.start) {
              b = i
            }
            i = ~~((a + b) / 2)
          } while (i !== oldI)
          i < 0 && (i = 0)
          startIndex = i

          this.totalSize = sizes[count - 1].accumulator

          // 查找endIndex
          for (endIndex = i; endIndex < count && sizes[endIndex].accumulator < scroll.end; endIndex++) {
            if (endIndex === -1) {
              endIndex = items.length - 1
            } else {
              endIndex++

              endIndex > count && (endIndex = count)
            }
          }

          if (endIndex > count) {
            endIndex = count
          }
        } else {
          this.totalSize = count * itemSize
        }

        // 获取当前展示的项 且设置其对应位置
        for (let index = startIndex; index < endIndex; index++) {
          let item = items[index]

          let view = this.addView(pool, index, item)

          // 更新position
          if (!itemSize) {
            view.position = sizes[index - 1].accumulator
          } else {
            view.position = index * itemSize
          }
        }

        this.pool = pool
      },
      // 修饰当前项
      addView (pool, index, item) {
        const view = {
          item,
          position: 0,
        }
        const nonReactive = {
          id: uid++,
          index,
          used: true,
        }
        Object.defineProperty(view, 'nr', {
          configurable: false,
          value: nonReactive,
        })
        pool.push(view)
        return view
      },

      // 判断ios版本，> 7 时原生滚动能触发onscroll
      iOSversion () {
        if (/iP(hone|od|ad)/.test(navigator.platform)) {
          var v = (navigator.appVersion).match(/OS (\d+)_(\d+)_?(\d+)?/)
          return [parseInt(v[1], 10), parseInt(v[2], 10), parseInt(v[3] || 0, 10)]
        }
      }
    }
  }
</script>

<style scoped>
  .xm-scrollerrecycle {
    height: 100%;
    overflow: auto;
  }

  .xm-scrollerrecycle__lists {
    position: relative;
  }

  .xm-scrollerrecycle__item {
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
  }
</style>
