<template>
  <div class="player"  v-show="playlist.length>0">
  <transition name="normal" @enter="enter" @after-enter="afterEnter" @leave="leave" @after-leave="afterLeave">
      <div class="normal-player" v-show="fullScreen">
          <div class="background">
              <img :src="currentSong.image" width="100%"
        height="100%">
          </div>
          <div class="top">
              <div class="back" @click="back">
                  <i class="icon-back"></i>
              </div>
              <h1 class="title">{{currentSong.name}}</h1>
              <h2 class="subtitle">{{currentSong.singer}}</h2>
          </div>
          <div class="middle">
              <div class="middle-l">
                  <div class="cd-wrapper" ref="cdWrapper">
                      <div class="cd">
                          <img :class="cdCls" class="image" :src="currentSong.image">
                      </div>
                  </div>

              </div>
          </div>
          <div class="bottom">
              <div class="progress-wrapper">
                <span class="time time-l">{{format(currentTime)}}</span>
                <div class="progress-bar-wrapper">
                  <progress-bar :percent="percent"></progress-bar>
                </div>
                <span class="time time-r">{{format(currentSong.duration)}}</span>
              </div>
              <div class="operators">
                  <div class="icon i-left">
                      <i class="icon-sequence"></i>
                  </div>
                  <div class="icon i-left" @click.stop="prev" :class="disableCls">
                      <i class="icon-prev"></i>
                  </div>
                  <div class="icon i-center" @click.stop="togglePlaying" :class="disableCls">
                      <i :class="playIcon"></i>
                  </div>
                  <div class="icon i-right" @click.stop="next" :class="disableCls">
                      <i class="icon-next"></i>
                  </div>
                  <div class="icon i-right">
                      <i class="icon icon-not-favorite"></i>
                  </div>
              </div>
          </div>
      </div>
      </transition>
      <transition name="mini"> 
      <div class="mini-player" v-show="!fullScreen" @click="open">
          <div class="icon">
              <div class="imgWrapper">
                  <img class="img" :class="cdCls" width="40" height="40" :src="currentSong.image">
              </div>
          </div>
          <div class="text">
              <h2 class="name"></h2>
              <p class="desc"></p>
          </div>
          <div class="control" @click.stop="togglePlaying">
            <i :class="miniIcon"></i>
          </div>
          <div class="control" @click.stop="showPlaylist">
              <i class="icon-playlist"></i>
          </div>
      </div>
      </transition>
      <audio ref="audio" @playing="ready" @error="error" :src="currentSong.url"  autoplay="autoplay" @timeupdate="updateTime"></audio>
  </div>
</template>

<script type="text/ecmascript-6">
import {mapGetters ,mapMutations} from 'vuex'
import animations from 'create-keyframe-animation'
import { prefixStyle } from 'common/js/dom'
import ProgressBar from 'base/progress-bar/progress-bar'
 const transform = prefixStyle('transform')

  export default {
    data(){
      return {
        songReady: false,
        currentTime: 0,
      }
    },
    computed: {
      cdCls(){
        return this.playing ? 'play' :'pause'
      },
      playIcon(){
        return this.playing ? 'icon-pause' :'icon-play'
      },
      miniIcon(){
        return this.playing ? 'icon-pause-mini' :'icon-play-mini'
      },
      disableCls(){
        return !this.songReady ? 'disable' : ''
      },
      percent() {
        return this.currentTime / this.currentSong.duration
      },

      ...mapGetters(['fullScreen','playlist','currentSong','playing','currentIndex'])
    },
    methods: {
      back(){
        this.setFullScreen(false)
      },
      open(){
        this.setFullScreen(true)
      },
      ready(){
        clearTimeout(this.timer)
        this.songReady = true
      },
      error(){
        clearTimeout(this.timer)
        this.timer = setTimeout(()=>{
          this.songReady = true
        },4000)
      },
      prev(){
        //不是在播放的状态不可以点上一首
         if(!this.songReady){
          return
        } 
        
        if(this.playlist.length === 1) {
          return 
        }
          let index = this.currentIndex
          index--
          if(index < 0) {
            index = this.playlist.length -1
          }
          this.setCurrentIndex(index)
          //播放
          if(!this.playing) {
            this.setPlayingState(true)
          }
          this.songReady = false
      },
      next(){
        //不是在播放的状态不可以点下一首
         if(!this.songReady){
          return
        } 
        
        if(this.playlist.length === 1) {
          return 
        }
        let index = this.currentIndex
          index++
          if(index > this.playlist.length -1) {
            index = 0
          }
          this.setCurrentIndex(index)
          //播放
          if(!this.playing) {
            this.setPlayingState(true)
          }
          this.songReady = false
      },
      togglePlaying(){
        if(!this.songReady){
          return
        } 
            this.setPlayingState(!this.playing)
      },
      updateTime(e){
        this.currentTime = e.target.currentTime
      },
      format(interval){
        let minutes = Math.floor(interval/ 60)
        let seconds = this._pad(Math.floor(interval % 60)) //或0取整 
        return `${minutes}:${seconds}`
      },
      _pad(num ,n =2){
        //加0的格式
        let len = num.toString().length

          while(n>len) {
            num = '0' + num
            n--
          }
         return num
      },
      showPlaylist(){

      },
      enter(el, done){
/*为了制作动画，需要初始位置。再向原本的位置做动画。
知道初始位置之后，就可以在enter的时候把元素移动到初始位置(从大到小)，设置动画时长
这样在动画结束的时候就会自动出现在css设置的位置上
结束的时候也是设置动画时长后，在动画的最后，到达初始位置。*/

       const{x, y, scale} = this._getPosAndScale()
       let animation = {
        0:{
          transform:  `translate3d(${x}px,${y}px,0) scale(${scale})`
        },
        60:{
          transform: `translate3d(0,0,0)  scale(1.1)`
        },
        100:{
          transform:`translate3d(0,0,0)  scale(1)`
        }
       }
       animations.registerAnimation({
        name:'move',
        animation,
        presets:{
          duration: 400,
          easing: 'linear'
        }
       })
       animations.runAnimation(this.$refs.cdWrapper,'move',done)
      },
      afterEnter(){
        //结开始动画之后
        animations.unregisterAnimation('move')
        this.$refs.cdWrapper.style.animation = ''

      },
      leave(el,done){
        //离开组件，开始动画的时候做什么
        const{x, y, scale} = this._getPosAndScale()
        this.$refs.cdWrapper.style.transition =  'all 0.4s'
        this.$refs.cdWrapper.style[transform] = `translate3d(${x}px,${y}px,0) scale(${scale})`
        const timer = setTimeout(done,400)
        this.$refs.cdWrapper.addEventListener('transitionend',()=>{
          clearTimeout(timer)
          done()
          
        })
      },
      afterLeave(){
        //结束结束动画之后
        this.$refs.cdWrapper.style['transform'] = ''
        this.$refs.cdWrapper.style.transition = ''
      },
      
      _getPosAndScale(){
        //从上向下移动到初始位置后，再向原本的位置做动画。
        const targetWidth = 40
        const paddingBottom = 30
        const paddingLeft = 40
        const cdPaddingTop = 80
        const cdWidth = window.innerWidth*0.8
        const scale = targetWidth/cdWidth
        let x = -(window.innerWidth/2 - paddingLeft)
        let y = window.innerHeight - cdPaddingTop - cdWidth/2 - paddingBottom
        return {x,y,scale}
      },
      
      ...mapMutations({setFullScreen:'SET_FULL_SCREEN',
                        setPlayingState:'SET_PLAYING_STATE',
                        setCurrentIndex:'SET_CURRENT_INDEX'
                      })
    
    },
    mounted(){
    },
    watch: {
      currentSong(newSong, oldSong){
        //选择，比如点击一个列表中的元素。 以及改变选择的歌曲后,
        if(!newSong.id || newSong.id === oldSong.id){
          return 
        }
        this.$nextTick(()=> {
        this.$refs.audio.src = newSong.url
        this.$refs.audio.play()
      })
      },
      playing(newPlaying){
        //作为改变了状态之后自动更新视图
        
        const audio = this.$refs.audio
        this.$nextTick(()=> {
          newPlaying ? audio.play() : audio.pause()
        })
      },
    },
    components: {
      ProgressBar
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .player
    .normal-player
      position: fixed
      left: 0
      right: 0
      top: 0
      bottom: 0
      z-index: 150
      background: $color-background
      .background
        position: absolute
        left: 0
        top: 0
        width: 100%
        height: 100%
        z-index: -1
        opacity: 0.6
        filter: blur(20px)
      .top
        position: relative
        margin-bottom: 25px
        .back
          position absolute
          top: 0
          left: 6px
          z-index: 50
          .icon-back
            display: block
            padding: 9px
            font-size: $font-size-large-x
            color: $color-theme
            transform: rotate(-90deg)
        .title
          width: 70%
          margin: 0 auto
          line-height: 40px
          text-align: center
          no-wrap()
          font-size: $font-size-large
          color: $color-text
        .subtitle
          line-height: 20px
          text-align: center
          font-size: $font-size-medium
          color: $color-text
      .middle
        position: fixed
        width: 100%
        top: 80px
        bottom: 170px
        white-space: nowrap
        font-size: 0
        .middle-l
          display: inline-block
          vertical-align: top
          position: relative
          width: 100%
          height: 0
          padding-top: 80%
          .cd-wrapper
            position: absolute
            left: 10%
            top: 0
            width: 80%
            box-sizing: border-box
            height: 100%
            .cd
              width: 100%
              height: 100%
              border-radius: 50%
              .image
                position: absolute
                left: 0
                top: 0
                width: 100%
                height: 100%
                box-sizing: border-box
                border-radius: 50%
                border: 10px solid rgba(255, 255, 255, 0.1)
              .play
                animation: rotate 20s linear infinite
          .playing-lyric-wrapper
            width: 80%
            margin: 30px auto 0 auto
            overflow: hidden
            text-align: center
            .playing-lyric
              height: 20px
              line-height: 20px
              font-size: $font-size-medium
              color: $color-text-l
        .middle-r
          display: inline-block
          vertical-align: top
          width: 100%
          height: 100%
          overflow: hidden
          .lyric-wrapper
            width: 80%
            margin: 0 auto
            overflow: hidden
            text-align: center
            .text
              line-height: 32px
              color: $color-text-l
              font-size: $font-size-medium
              &.current
                color: $color-text
            .pure-music
              padding-top: 50%
              line-height: 32px
              color: $color-text-l
              font-size: $font-size-medium
      .bottom
        position: absolute
        bottom: 50px
        width: 100%
        .dot-wrapper
          text-align: center
          font-size: 0
          .dot
            display: inline-block
            vertical-align: middle
            margin: 0 4px
            width: 8px
            height: 8px
            border-radius: 50%
            background: $color-text-l
            &.active
              width: 20px
              border-radius: 5px
              background: $color-text-ll
        .progress-wrapper
          display: flex
          align-items: center
          width: 80%
          margin: 0px auto
          padding: 10px 0
          .time
            color: $color-text
            font-size: $font-size-small
            flex: 0 0 30px
            line-height: 30px
            width: 30px
            &.time-l
              text-align: left
            &.time-r
              text-align: right
          .progress-bar-wrapper
            flex: 1
        .operators
          display: flex
          align-items: center
          .icon
            flex: 1
            color: $color-theme
            &.disable
              color: $color-theme-d
            i
              font-size: 30px
          .i-left
            text-align: right
          .i-center
            padding: 0 20px
            text-align: center
            i
              font-size: 40px
          .i-right
            text-align: left
          .icon-favorite
            color: $color-sub-theme
      &.normal-enter-active, &.normal-leave-active
        transition: all 0.4s
        .top, .bottom
          transition: all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
      &.normal-enter, &.normal-leave-to
        opacity: 0
        .top
          transform: translate3d(0, -100px, 0)
        .bottom
          transform: translate3d(0, 100px, 0)
    .mini-player
      display: flex
      align-items: center
      position: fixed
      left: 0
      bottom: 0
      z-index: 180
      width: 100%
      height: 60px
      background: $color-highlight-background
      &.mini-enter-active, &.mini-leave-active
        transition: all 0.4s
      &.mini-enter, &.mini-leave-to
        opacity: 0
      .icon
        flex: 0 0 40px
        width: 40px
        height: 40px
        padding: 0 10px 0 20px
        .imgWrapper
          height: 100%
          width: 100%
          img
            border-radius: 50%
            &.play
              animation: rotate 10s linear infinite
            &.pause
              animation-play-state: paused
      .text
        display: flex
        flex-direction: column
        justify-content: center
        flex: 1
        line-height: 20px
        overflow: hidden
        .name
          margin-bottom: 2px
          no-wrap()
          font-size: $font-size-medium
          color: $color-text
        .desc
          no-wrap()
          font-size: $font-size-small
          color: $color-text-d
      .control
        flex: 0 0 30px
        width: 30px
        padding: 0 10px
        .icon-play-mini, .icon-pause-mini, .icon-playlist
          font-size: 30px
          color: $color-theme-d
        .icon-mini
          font-size: 32px
          position: absolute
          left: 0
          top: 0

  @keyframes rotate
    0%
      transform: rotate(0)
    100%
      transform: rotate(360deg)
</style>
