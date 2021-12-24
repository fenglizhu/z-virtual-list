<template>
  <div ref="list" class="virtual-list-container" @scroll="scrollEvent($event)">
    <div class="virtual-list-phantom" :style="{height: listHeight +'px'}"></div>
    <div class="virtual-list" :style="{ transform: getTransform }">
      <ul>
        <li ref="item" v-for="item in visibleData" :key="item.id" class="list-item" :data-id="item.id">
          <span style="color:red">{{item.id}}</span>
          {{item.value}}
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
export default {
  props:{
    listData:{
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      // 每项的高度
      itemSize: 100,
      // 可视区域的高度
      screenHeight: 0,
      // 开始索引
      start: 0,
      // 结束索引
      end: null,
      //偏移量
      startOffset: 0,
      // 所有滚动过数据的高度集合
      hasViewHeightMap: {}
    }
  },
  computed: {

    // 列表总高度
    listHeight() {
      return this.itemSize * this.listData.length;
    },

    // 可显示的列表项数
    visibleCount() {
      return Math.ceil(this.screenHeight / this.itemSize);
    },
    // 可视数据，前后需要缓冲一屏
    visibleData() {
      let start = this.start > this.visibleCount ? this.start - this.visibleCount : this.start
      return this.listData.slice(start, this.end + this.visibleCount)
    },

    // 偏移量对应的style
    getTransform() {
      return `translateY(${this.startOffset}px)`;
    }
  },
  mounted() {
    this.screenHeight = this.$el.clientHeight;
    this.end = this.start + this.visibleCount;
    /**
     * virtual-list-container 可视区域的容器
     * virtual-list-phantom 为容器内的占位，高度为总列表高度，用于形成滚动条
     * virtual-list 为列表项的渲染区域
     */
  },
  updated() {
    this.$nextTick(()=>{
      if(this.$refs.item && this.$refs.item.length) {
        this.setItemSize()
      }
    })
  },
  methods:{
    scrollEvent(){
      // 当前滚动的位置
      let scrollTop = this.$refs.list.scrollTop;

      // 此时开始的索引
      this.start = Math.floor(scrollTop / this.itemSize);

      // 此时结束的索引
      this.end = this.start + this.visibleCount;

      //此时的偏移量
      let offset = scrollTop - (scrollTop % this.itemSize) - this.itemSize * this.visibleCount
      this.startOffset = offset > 0 ? offset : scrollTop - (scrollTop % this.itemSize);
    },
    /**
     * 计算平均高度
     */
    setItemSize() {
      for (let i = 0; i < this.$refs.item.length; i++) {
        let id = this.$refs.item[i].getAttribute('data-id');
        this.hasViewHeightMap[id] = this.$refs.item[i].offsetHeight
      }
      // 所有滚动过数据的高度集合
      let vTotal = 0;
      let length = Object.keys(this.hasViewHeightMap).length;
      for (const key in this.hasViewHeightMap) {
        vTotal += this.hasViewHeightMap[key];
      }
      // 每一项的高度可以取已经滑动过的所有高度的平均值，跟之前一样就不用对比了
      let size = Math.floor(vTotal / length)
      if(this.itemSize == size) return
      this.itemSize = size;
    },
  }
}
</script>
<style>
  .virtual-list-container {
    height: 100%;
    overflow: auto;
    position: relative;
  }

  .virtual-list-phantom {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    z-index: -1;
  }

  .virtual-list {
    left: 0;
    right: 0;
    top: 0;
    position: absolute;
  }

  .list-item {
    padding: 30px 0;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
  }
</style>