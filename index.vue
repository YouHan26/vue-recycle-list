<template>
    <div class="vue-recycleList">
        <div class="vue-recycleList-items" :style="{height: height + 'px'}">
            <div
                    v-for="(item, index) in visibleItems"
                    :key="index"
                    class="vue-recycleList-item"
                    :style="{transform: 'translate3d(0,' + item.top + 'px,0)'}"
            >
                <div>
                    <slot name="item" :data="item.data" :index="index"></slot>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
  /**
   * example:
   * <div style="height: 500px; background-color: #00a745">
   <virtualList
   :data="list"
   :size="20"
   :itemHeight="50"
   >
   <template slot="item" slot-scope="props">
   <div
   @click="itemClick(props.data.id)"
   :id="props.data.id"
   style="height: 50px;border: solid 1px gray"
   >
   {{props.data.id}}
   </div>
   </template>
   </virtualList>
   </div>
   * @type {Array}
   */
  let cache = []

  export default {
    name: 'virtualList',
    props: ['itemHeight', 'data', 'size'],
    data() {
      return {
        start: 0,
        end: 0,
        items: [],
      }
    },
    computed: {
      len() {
        return this.items.length
      },
      height() {
        return this.len * this.itemHeight
      },
      visibleItems() {
        return this.items.slice(Math.max(0, this.start), Math.min(this.len, this.start + this.size))
      }
    },
    watch: {
      data(value) {
        this.updateStartAndEnd()
        this.loadItems(value)
      },
    },
    mounted() {
      cache = []
      this.$el.addEventListener('scroll', this.onScroll, {passive: true})
    },
    beforeDestroy() {
      this.$el.removeEventListener('scroll', this.onScroll)
    },
    methods: {
      onScroll() {
        this.refresh()
      },
      refresh() {
        this.updateStartAndEnd()
      },
      updateStartAndEnd() {
        let top = this.$el.scrollTop
        const start = ~~(top / this.itemHeight) - 1
        if (start !== this.start) {
          this.start = start
        }
      },
      loadItems(value) {
        this.items = value.map((data, index) => {
          return {
            data,
            top: this.getItem(index)
          }
        })
      },
      getItem(index) {
        cache[index] || (cache[index] = index * this.itemHeight)
        return cache[index]
        // let last = this.len
        // while (last > 0 && cache[last--]) {
        // }
        // for (let i = last; i < index; i++) {
        //   cache[i + 1] = cache[i] + this.itemHeight
        // }
        // return cache[index]
      }
    }
  }
</script>

<style lang="scss" scoped>
    .vue-recycleList {
        overflow-x: hidden;
        overflow-y: auto;
        height: 100%;
        .vue-recycleList-items {
            position: relative;
            margin: 0;
            padding: 0;
            .vue-recycleList-item {
                position: absolute;
                width: 100%;
            }
        }
    }
</style>

