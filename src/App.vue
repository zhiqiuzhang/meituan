<template>
  <div id="app">
    <app-header :poiInfo='poiInfo' />
    <app-nav />
    <router-view></router-view>
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
      poiInfo: {}
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
  }
}
</script>

<style>

</style>
