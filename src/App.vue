<template>
  <div id="app">
    <VirtualList :listData="listData" />
  </div>
</template>

<script>
import faker from "faker";
import VirtualList from "./components/VirtualList.vue";
export default {
  name: 'App',
  components:{
    VirtualList
  },
  data() {
    return {
      listData: [{id:0,value:1111}]
    }
  },
  mounted(){
    let data = []
    for (let i = 0; i < 1000; i++) {
      data.push({
        id: i,
        value: faker.lorem.sentences()
      })
    }
    this.listData = data
  }
}
</script>
<style>
*{
  margin: 0;
  padding: 0;
}
#app{
  height: 100vh;
}
</style>