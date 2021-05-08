<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <h2>定高</h2>
    <div style="height: 300px;border: 3px solid #6495ED;">
      <x-scrollerrecycle :items="items" :item-size="41">
        <template v-slot="{ item, index }">
          <div style="height: 40px;line-height: 40px;overflow:hidden;border-bottom: 1px solid #aaa;"> {{ item }}</div>
        </template>
      </x-scrollerrecycle>
    </div>

    <h2>不定高</h2>
    <div style="height: 300px;border: 3px solid #6495ED;">


      <x-scrollerdynamic :items="items" :min-item-size="54">
        <template v-slot="{ item, index }">
          <x-scrollerdynamicitem :item="item" :index="index">
            <div style="border-bottom: 1px solid #aaa;padding: 6px 0;"> {{ item }}</div>
          </x-scrollerdynamicitem>
        </template>
      </x-scrollerdynamic>
    </div>

  </div>
</template>

<script>
  import scrollerdynamicitem from './scrollerdynamicitem/scrollerdynamicitem'
  import scrollerdynamic from './scrollerdynamic/scrollerdynamic'
  import scrollerrecycle from './scrollerrecycle/scrollerrecycle'

  export default {
    name: 'HelloWorld',
    components: {
      'x-scrollerdynamicitem': scrollerdynamicitem,
      'x-scrollerdynamic': scrollerdynamic,
      'x-scrollerrecycle': scrollerrecycle,
    },
    data () {
      let items = []
      for (let i = 0; i < 10000; i++) {
        let item = {
          key: i,
          text: '王大虎' + i,
        }

        if (i % 2 > 0) {
          item.text = '王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎王大虎' + i
        }

        items.push(item)
      }

      return {
        msg: 'Welcome to Your Vue.js App',
        items: items,
      }
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
