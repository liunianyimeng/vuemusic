<template>
  <transition name="slide">
    <music-list :title="singer.name" :bg-image="singer.avatar" :songs="songs"></music-list>
  </transition>
</template>
<script>
import {mapGetters} from 'vuex'
import {getSingerDetail, getMusic} from 'api/singer'
import {createSong} from 'common/js/song'
import {ERR_OK} from 'api/config'
import MusicList from 'components/music-list/music-list'
export default {
  data () {
    return {
      songs: []
    }
  },
  components: {
    MusicList
  },
  created () {
    this._getDetail()
  },
  computed: {
    ...mapGetters(['singer'])
  },
  methods: {
    _getDetail () { // 获取歌手详情
      if (!this.singer.id) { // 在当前页面刷新
        this.$router.push('/singer')
      };
      getSingerDetail(this.singer.id).then((res) => {
        if (res.code === ERR_OK) {
          this.songs = this._normalizeSongs(res.data.list)
        }
      })
    },
    _normalizeSongs (lists) { // 格式化歌曲数据
      let ret = []
      lists.forEach((item) => {
        let {musicData} = item
        if (musicData.songid && musicData.albummid) {
          getMusic(musicData.songmid).then((res) => {
            if (res.code === ERR_OK) {
              const svkey = res.data.items
              const songVkey = svkey[0].vkey
              const newSong = createSong(musicData, songVkey)
              ret.push(newSong)
            }
          })
        }
      })
      return ret
    }
  }
}
</script>
<style lang='stylus' scoped>
@import '../../common/stylus/variable'
.slide-enter-active, .slide-leave-active
  transition: 0.3s all

.slide-enter, .slide-leave-to
  transform: translate3d(100%, 0, 0)
</style>
