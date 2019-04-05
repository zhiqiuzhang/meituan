<template>
  <div id="app">
    <app-header :poiInfo='poiInfo' />
    <app-nav :commentNum='commentNum'/>
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script>
import Header from './components/header/Header'
import Nav from './components/nav/Nav'

export default {
  name: 'App',
  components: {
    'app-header': Header,
    'app-nav': Nav
  },
  data() {
    return {
      poiInfo: {},
      commentNum: 0
    }
  },
  created(){
    fetch('/api/goods').then(res => {
       return res.json()
    }).then(res => {
      if(res.code === 0) {
        this.poiInfo = res.data.poi_info;
      }
    })

    fetch('/api/ratings').then(res => {
       return res.json()
    }).then(res => {
      if(res.code === 0) {
        this.commentNum = res.data.comment_num;
      }
    })
  }
}
</script>

<style>

</style>
