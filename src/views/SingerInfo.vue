<template>
  <div class="rank-info-content plist-info">
    <div class="rank-banner-wrap" :style="{backgroundImage: `url(${imgurl})`}">
    </div>

    <div class="plist-desp container">
      <p class="plist-desp-p" :class="{'plist-desp-hide': hideDesc }">{{desc}}</p>
      <img src="../assets/images/close_icon.png" @click="toggleDesc" class="plist-desp-icon" v-if="hideDesc">
      <img src="../assets/images/open_icon.png" @click="toggleDesc" class="plist-desp-icon" v-else>
    </div>
    <div class="plist-desp-bottom" style="width: 100%;height: 5px;background-color: #f1f1f1"></div>

    <div class="rank-info-list">
      <mt-cell v-for="(item,index) in songList" :title="item.filename" @click.native="playAudio(index)" :key="index">
      
      </mt-cell>
    </div>
  </div>
</template>
<script type="es6">
  import { Indicator } from 'mint-ui'
  import { PLAY_AUDIO } from '../mixins'
  export default {
    mixins: [PLAY_AUDIO],
    data(){
      return {
        imgurl: '',
        songList: [],
        updateTime: '',
        desc: '',
        hideDesc: true,
        opacity: 0
      }
    },
    //通过路由的before钩子解除router-view缓存限制
    beforeRouteEnter (to, from, next) {
      next(vm => {
        vm.$store.commit('showHead', true)
        vm.get()
        window.onscroll = ()=> {
          vm.opacity = window.pageYOffset / 250
          vm.$store.commit('setHeadStyle', {background: `rgba(166,28,0,${vm.opacity})`})
        }
      })
    },
    beforeRouteLeave(to, from, next){
      this.$store.commit('showHead', false)
      window.onscroll = null
      next()
    },
    methods: {
      get(){
        Indicator.open({
          spinnerType: 'fading-circle'
        });
        var infoID = this.$route.params.id;
        this.$http.get(`/proxy/singer/info/${infoID}&json=true`).then(({data})=> {
          Indicator.close()
	        const info = data.info
	        const songList = data.songs.list
	        this.imgurl = info.imgurl.replace('{size}', '400')
	        this.desc = info.intro
	        this.songList = songList
	        this.$store.commit('setHeadTitle', info.singername)
				});
      },
      toggleDesc(){
        this.hideDesc = !this.hideDesc
      }
    }
  }
</script>
<style scoped>
  .rank-banner-wrap {
    height: 250px;
  }
</style>
