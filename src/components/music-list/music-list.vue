<template>
  <div class="music-list">
    <div class="back" @click="back">
      <i class="icon-back"></i>
    </div>
    <h1 class="title" v-html="title"></h1>
    <div class="bg-image" :style="bgStyle" ref="bgImage">
      <div class="play-wrapper" ref="playBtn">
        <div class="play-btn" @click="random" v-show="songs.length > 0">
          <i class="icon-play"></i>
          <span class="text">随机播放全部</span>
        </div>
      </div>
      <div class="filter"></div>
    </div>
    <div class="layer" ref="layer"></div>
    <scroll :probe-type="probeType" @scroll="scroll"
            :listen-scroll="listenScroll" :data="songs"
            class="list-wrapper" ref="list">
      <div class="song-list-wrapper" :class="{'isRank': rank === true}">
        <song-list @select="selectSong" :rank="rank" :songs="songs"></song-list>
      </div>
      <div class="loading-container" v-show="!songs.length">
        <loading title="加载中"></loading>
      </div>
    </scroll>
  </div>
</template>

<script type="text/ecmascript-6">
  import SongList from 'base/song-list/song-list'
  import Scroll from 'base/scroll/scroll'
  import Loading from 'base/loading/loading'
  import {prefixStyle} from 'common/js/dom'
  import {mapActions} from 'vuex'
  import {playlistMixin} from 'common/js/mixin'

  const RESERVED_HEIGHT = 100
  const transform = prefixStyle('transform')

  export default {
    mixins: [playlistMixin],
    props: {
      songs: {
        type: Array,
        default: []
      },
      bgImage: {
        type: String,
        default: ''
      },
      title: {
        type: String,
        default: ''
      },
      rank: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        scrollY: -1
      }
    },
    created() {
      this.probeType = 3
      this.listenScroll = true
    },
    mounted() {
      this.imageHeight = this.$refs.bgImage.clientHeight
      this.$refs.list.$el.style.top = `${this.imageHeight}px`
      this.minTransalteY = -this.imageHeight + RESERVED_HEIGHT
    },
    methods: {
      back() {
        this.$router.back()
      },
      scroll(pos) {
        this.scrollY = pos.y
      },
      selectSong(item, index) {
        this.selectPlay({
          list: this.songs,
          index: index
        })
      },
      random() {
        this.randomPlay({
          list: this.songs
        })
      },
      handlePlaylist(playlist) {
        const bottom = playlist.length > 0 ? '60px' : 0
        this.$refs.list.$el.style.bottom = bottom
        this.$refs.list.refresh()
      },
      ...mapActions([
        'selectPlay',
        'randomPlay'
      ])
    },
    watch: {
      scrollY(newY) {
        let translateY = Math.max(this.minTransalteY, newY)
        let zIndex = 0
        let scale = 1
        const percent = Math.abs(newY / this.imageHeight)
        this.$refs.layer.style[transform] = `translate3d(0,${translateY}px,0)`
        this.$refs.playBtn.style[transform] = `translate3d(0,${translateY}px,0)`
        if (newY > 0) {
          scale = 1 + percent
          zIndex = 10
          this.$refs.playBtn.style[transform] = 'translate3d(0,0,0)'
        }
        if (newY < this.minTransalteY) {
          zIndex = 10
          this.$refs.bgImage.style.paddingTop = 0
          this.$refs.bgImage.style.height = `${RESERVED_HEIGHT}px`
          this.$refs.playBtn.style[transform] = 'translate3d(0,0,0)'
        } else {
          this.$refs.bgImage.style.paddingTop = '70%'
          this.$refs.bgImage.style.height = 0
        }
        this.$refs.bgImage.style[transform] = `scale(${scale})`
        this.$refs.bgImage.style.zIndex = zIndex
      }
    },
    computed: {
      bgStyle() {
        return `background-image:url(${this.bgImage})`
      }
    },
    components: {
      SongList,
      Scroll,
      Loading
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "music-list.styl"
</style>
