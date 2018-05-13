<template>
  <transition name="slide">
    <music-list :title="disc.dissname" :bg-image="disc.imgurl" :songs="songs"></music-list>
  </transition>
</template>
<script>
import MusicList from 'components/music-list/music-list'
import {getSongList} from 'api/recommend'
import {ERR_OK} from 'api/config'
import {createSong} from 'common/js/song'
import {mapGetters} from 'vuex'
import {getMusic} from 'api/singer'
export default {
  data () {
    return {
      songs: []
    }
  },
  created () {
    this._getDiscDetail()
    if (!this.disc.dissid) {
      this.$router.push('/recommend')
    }
  },
  components: {
    MusicList
  },
  computed: {
    ...mapGetters(['disc'])
  },
  methods: {
    _getDiscDetail () {
      getSongList(this.disc.dissid).then((res) => {
        if (res.code === ERR_OK) {
          this.songs = this._normalizeSongs(res.cdlist[0].songlist)
          // console.log(res.cdlist[0].songlist)
        }
      })
    },
    _normalizeSongs (lists) { // 格式化歌曲数据
      let ret = []
      lists.forEach((item) => {
      ///  console.log(musicData.songmid)
        if (item.songmid && item.albummid) {
          getMusic(item.songmid).then((res) => {
            if (res.code === ERR_OK) {
              const svkey = res.data.items
              const songVkey = svkey[0].vkey
              const newSong = createSong(item, songVkey)
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
.slide-enter-active, .slide-leave-active
  transition: all 0.3s

.slide-enter, .slide-leave-to
  transform: translate3d(100%, 0, 0)
</style>
