<template>
  <div class="player">
    <transition name="move">
      <div v-show="fullScreen" class="normal-player">
        <div class="background">
          <img width="100%" height="100%" :src="songDetail.albumBlurPicUrl" alt="">
        </div>
        <div class="header">
          <div @click="back" class="back">
            <i class="icon-arrow-left"></i>
          </div>
          <div class="description">
            <h2 class="title">{{songDetail.name}}</h2>
            <span class="text">{{songDetail.artists}}</span>
          </div>
        </div>
        <div class="content">
          <div @click="shiftLcr" class="content-wrapper">
            <transition name="fade">
              <div v-show="!showLcr" class="cd-wrapper ">
                <div class="cd rotate" :class="{'paused':!playing}">
                  <img :src="songDetail.albumBlurPicUrl" alt="" class="image">
                </div>
              </div>
            </transition>
            <transition name="shift">
              <div v-show="showLcr" class="lyric-wrapper">
                <div class="background"></div>
                <div ref="line" v-show="scrolling" class="line-wrapper">
                  <i class="fa fa-play"></i>
                  <span class="line"></span>
                  <span class="time">01:22</span>
                </div>
                <scroll ref="lyricList" v-show="songLyric" @scroll="scroll" @scrollStart="scrollStart " @scrollEnd="scrollEnd" :data="songLyric" :scrollEnd="true" :scrollStart="true" :listenScroll="true" :probeType="probeType">
                  <div class="lyric">
                    <p ref="lyricLine"  :class="{current:currentLine===index}" v-for="(line,index) in songLyric.lines" :key="index">
                      {{line.txt}}</p>
                  </div>
                </scroll>
              </div>
            </transition>
          </div>
        </div>
        <div class="footer">
          <div class="progess-bar-wrapper">
            <progress-bar @barTouch="barTouch" :currentTime="currentTime" :totalTime="totalTime"></progress-bar>
          </div>
          <div class="operators">
            <div @click="toggleMode" class="mode">
              <i :class="modeIcon"></i>
            </div>
            <div class="play-control">
              <div @click="backward" class="backward">
                <i class="icon-backward"></i>
              </div>
              <div @click="togglePlaying" class="playing">
                <i :class="playState"></i>
              </div>
              <div @click="forward" class="forward">
                <i class="icon-backward"></i>
              </div>
            </div>
            <div @click="showList" class="list">
              <i class="icon-playlist"></i>
            </div>
          </div>
        </div>
      </div>
    </transition>
    <transition name="mini">
      <div @click="toFullScreen" v-show="!fullScreen&&songDetail.albumBlurPicUrl" class="mini-player">
        <div class="img-wrapper">
          <img :src="songDetail.albumBlurPicUrl" alt="" width="100%" height="100%">
        </div>
        <div class="text">
          <div class="name">{{song.name}}</div>
          <div class="tip">横滑可以切换上下首奥</div>
        </div>
        <div class="control">
          <i @click.stop="togglePlaying" :class="playState"></i>
          <i @click.stop="showList" class="icon-playlist"></i>
        </div>
      </div>
    </transition>
    <transition name="move">
      <div @click="hideList" v-show="footListShow" class="foot-list-wrapper">
        <foot-list></foot-list>
      </div>
    </transition>
    <transition name="fade">
      <div v-show="footListShow" class="foot-list-background"></div>
    </transition>
    <audio ref="audio" :src="songDetail.url" @play="ready" @error="error" @timeupdate="updateTime" @ended="end"></audio>
  </div>
</template>

<script type="text/ecmascript-6">
import Scroll from 'base/scroll/scroll';
import ProgressBar from 'base/progress-bar/progress-bar';
import FootList from 'pages/foot-list/foot-list';
import {ERR_OK} from 'api/config';
import {playMode} from 'common/js/config';
import {mapGetters, mapMutations} from 'vuex';
import {createSong} from 'common/js/song';
import Lyric from 'lyric-parser';
export default {
  props: {},
  created() {
    this.probeType = 3;
  },
  mounted() {
    this.line = this.$refs.line;
    console.log(this.line);
  },
  components: {
    Scroll,
    FootList,
    ProgressBar
  },
  data() {
    return {
      isNull: false,
      scrolling: false,
      scrolTimer: false,
      songDetail: {},
      songLyric: {},
      showLcr: false,
      currentLine: 0,
      currentTime: 0,
      totalTime: 0,
      footListShow: false,
      playingLyric: ''
    };
  },
  methods: {
    back() {
      this.setFullScreen(false);
    },
    shiftLcr() {
      this.showLcr = !this.showLcr;
    },
    togglePlaying() {
      this.setPlaying(!this.playing);
      if (this.songLyric) {
        this.songLyric.togglePlay();
      }
    },
    toFullScreen() {
      this.setFullScreen(true);
    },
    toggleMode() {
      if (this.mode === 2) {
        this.setMode(0);
      } else {
        this.setMode(this.mode + 1);
      }
    },
    showList() {
      this.footListShow = true;
    },
    hideList() {
      this.footListShow = false;
    },
    backward() {
      if (this.currentIndex > 0) {
        this.setCurrentIndex(this.currentIndex - 1);
      } else {
        this.setCurrentIndex(this.currentPlaylist.length);
      }
    },
    forward() {
      if (this.currentIndex < this.currentPlaylist.length) {
        this.setCurrentIndex(this.currentIndex + 1);
      } else {
        this.setCurrentIndex(0);
      }
    },
    ready() {},
    scroll(pos) {
      console.log(pos);
    },
    scrollStart() {
      if (this.scrolTimer) clearTimeout(this.scrolTimer);
      this.scrolling = true;
    },
    scrollEnd() {
      this.scrolTimer = setTimeout(() => {
        this.scrolling = false;
      }, 2000);
    },
    error(e) {
      console.log(e);
    },
    updateTime(e) {
      this.currentTime = e.target.currentTime;
      this.totalTime = isNaN(e.target.duration)
        ? this.totalTime
        : e.target.duration;
    },
    loop() {
      this.$refs.audio.currentTime = 0;
      this.$refs.audio.play();
      this.setPlaying(true);
      if (this.songLyric && this.songLyric.seek) {
        this.songLyric.seek(0);
        this.songLyric.play();
      }
    },
    end() {
      if (this.mode === playMode.loop) {
        this.loop();
      } else {
        this.forward();
      }
    },
    barTouch(time) {
      this.$refs.audio.currentTime = time;
      console.log(time);
      this.songLyric.seek(time * 1000);
    },
    _getSongDetail() {
      this.$http
        .all([
          this.$http.get(
            `/api/song/detail/?id=${this.song.id}&ids=[${this.song.id}]`
          ),
          this.$http.get(
            `/api/song/lyric?os=pc&id=${this.song.id}&lv=-1&kv=-1&tv=-1`
          ),
          this.$http.get(`/newapi/music/url?id=${this.song.id}`)
        ])
        .then(
          this.$http.spread((detailResp, lyricResp, urlResp) => {
            var detail = detailResp.data;
            var lyric = lyricResp.data;
            var url = urlResp.data;
            if (detail.code === ERR_OK && detail.songs.length > 0) {
              this.songDetail = createSong(detail.songs[0], url.data[0].url);
              if (!lyric.lrc) {
                lyric.lrc = {
                  lyric: '[00:00.00] [10:00.00]暂无歌词:('
                };
              }
            }
            this.$nextTick(() => {
              this._initLyric(lyric);
            });
          })
        )
        .catch(error => {
          if (error.response) {
            this.songLyric = null;
            // 请求已发出，但服务器响应的状态码不在 2xx 范围内
            console.log('error', error.response.data);
            console.log('error', error.response.status);
            console.log('error', error.response.headers);
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log('Error', error.message);
          }
          console.log(error.config);
        });
    },
    _initLyric(lyric) {
      this._destroyLyric();
      this.songLyric = new Lyric(lyric.lrc.lyric, ({lineNum, txt}) => {
        this.currentLine = lineNum;
        if (!this.scrolling) {
          if (lineNum > 4) {
            let lineEl = this.$refs.lyricLine[lineNum - 4];
            this.$refs.lyricList.scrollToElement(lineEl, 1000);
          } else {
            this.$refs.lyricList.scrollTo(0, 0, 1000);
          }
          this.playingLyric = txt;
        }
      });
      if (this.playing) {
        this.songLyric.play();
      }
    },
    _destroyLyric() {
      if (this.songLyric && this.songLyric.stop) {
        this.songLyric.stop();
        this.songLyric = null;
      }
    },
    _updateSong() {
      this.$nextTick(() => {
        this.setPlaying(true);
        setTimeout(() => {
          this.$refs.audio.play();
        }, 500);
      });
    },
    ...mapMutations({
      setFullScreen: 'SET_FULL_SCREEN',
      setPlaying: 'SET_PLAYING',
      setCurrentIndex: 'SET_CURRENT_INDEX',
      setMode: 'SET_MODE'
    })
  },
  computed: {
    playState() {
      return this.playing ? 'icon-pause' : 'icon-play';
    },
    modeIcon() {
      let modeArr = ['icon-list-cycle', 'icon-single-cycle', 'icon-random'];
      return modeArr[this.mode];
    },
    ...mapGetters([
      'playing',
      'fullScreen',
      'currentPlaylist',
      'randomPlaylist',
      'currentIndex',
      'mode',
      'song'
    ])
  },
  watch: {
    currentIndex() {
      this._getSongDetail();
      this._updateSong();
    },
    playing(newPlaying) {
      const audio = this.$refs.audio;
      console.log(newPlaying);
      this.$nextTick(() => {
        newPlaying ? audio.play() : audio.pause();
      });
    },
    currentPlaylist() {
      this._getSongDetail();
      this.isNull = this.currentPlaylist.length === 0;
    }
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable';
@import '~common/stylus/mixin';

@keyframes lazy-animation {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

@keyframes rotating-animation {
  from {
    transform: rotate(0);
  }

  to {
    transform: rotate(360deg);
  }
}

img[lazy=loading] {
  animation: lazy-animation 2s;
}

img[lazy=loaded] {
  animation: lazy-animation 2s;
}

.player {
  color: $color-text-l;
  background: $color-background-l;

  .normal-player {
    position: fixed;
    display: flex;
    flex-direction: column;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 1;

    .background {
      position: fixed;
      z-index: -5;
      top: 0;
      left: 0;
      height: 100%;
      width: 100%;
      overflow: hidden;

      &:before {
        position: absolute;
        content: '';
        display: block;
        top: 0;
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 1);
        z-index: -10;
      }

      img {
        filter: blur(100px);
      }
    }

    .header {
      display: flex;
      flex: 50px 0 0;
      width: 100%;
      color: $color-text-lll;
      border-width: 0 0 1px 0;
      border-style: solid;
      border-color: rgba(200, 200, 200, 0.3) $color-text rgba(200, 200, 200, 0.3);

      .back {
        font-size: $font-size-large-x;
        flex: 40px 0 0;
        text-align: center;
        line-height: 50px;
      }

      .description {
        flex: 1 0 0;
        overflow: hidden;
        padding: 10px 0;

        .title {
          font-size: $font-size-medium-x;
          line-height: 20px;
          text-ellipsis();
        }

        .text {
          font-size: $font-size-small-s;
          text-ellipsis();
        }
      }
    }

    .content {
      flex: 1;
      overflow: hidden;

      .content-wrapper {
        position: relative;
        height: 100%;

        .cd-wrapper {
          position: relative;
          margin: 0 auto;
          width: 80%;
          height: 0;
          padding-top: 80%;

          .cd {
            position: absolute;
            top: 30%;
            left: 0;
            height: 100%;
            width: 100%;
            background-size: 100% 100%;
            background-image: url('cd.png');

            &.rotate {
              animation: rotating-animation 15s linear infinite;
            }

            &.paused {
              animation-play-state: paused;
            }

            img {
              padding: 15%;
              width: 70%;
              height: 70%;
              border-radius: 50%;
            }
          }
        }

        .lyric-wrapper {
          display: flex;
          position: absolute;
          top: 40px;
          bottom: 40px;
          width: 100%;
          align-items: center;

          .line-wrapper {
            display: flex;
            align-items: center;
            text-align: center;
            box-sizing: border-box;
            font-size: 0;
            position: absolute;
            width: 100%;
            padding: 0 5px;

            .fa-play {
              flex: 15px 0 0;
              font-size: $font-icon-size-small;
              height: 20px;
              line-height: 20px;
            }

            .line {
              flex: 1;
              margin: 0 1px;
              width: 90%;
              height: 0;
              opacity: 0.1;
              border-bottom: 1px solid #FFF;
            }

            .time {
              flex: 40px 0 0;
              width: 5%;
              font-size: $font-size-small;
            }
          }

          .background {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            z-index: -2;
            background: rgb(7, 17, 27);
            opacity: 0.2;
          }

          .lyric {
            width: 80%;
            margin: 0 auto;
            text-align: center;
            line-height: 35px;
            font-size: $font-size-medium;
            color: $color-text-ll;

            .current {
              color: #FFF;
            }
          }
        }
      }
    }

    .footer {
      flex: 120px 0 0;

      .progess-bar-wrapper {
        width: 100%;
        height: 20px;
        padding: 10px 0;
      }

      .operators {
        text-align: center;
        height: 60px;
        padding-bottom: 20px;
        line-height: 60px;
        display: flex;

        .mode {
          font-size: $font-icon-size-medium;
          flex: 50px 0 0;
        }

        .play-control {
          display: flex;
          flex: 1;
          color: $color-text-ll;
          font-size: $font-icon-size-large-x;
          font-weight: 300;

          .backward {
            flex: 1;
          }

          .playing {
            flex: 1;
          }

          .forward {
            flex: 1;
            transform: scaleX(-1);
          }
        }

        .list {
          font-size: $font-icon-size-medium;
          flex: 50px 0 0;
        }
      }
    }
  }

  .mini-player {
    position: fixed;
    display: flex;
    left: 0;
    right: 0;
    bottom: -1px;
    height: 50px;
    width: 100%;
    background: $color-background;
    z-index: 100;

    .img-wrapper {
      flex: 50px 0 0;
      box-sizing: border-box;
      height: 100%;
      padding: 5px;
    }

    .text {
      flex: 1;
      padding-left: 10px;

      .name {
        color: $color-text-lll;
        font-size: $font-size-medium;
        height: 30px;
        line-height: 30px;
        overflow: hidden;
      }

      .tip {
        font-size: $font-size-small-s;
      }
    }

    .control {
      display: flex;
      flex: 100px 0 0;
      font-size: $font-size-large-x;
      text-align: center;

      i {
        font-size: $font-icon-size-medium;
        line-height: 50px;
        flex: 1;
      }
    }
  }

  .foot-list-wrapper {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 100;
  }

  .foot-list-background {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 10;
    background: rgba(3, 4, 5, 0.6);
  }
}

.move-enter-active, .move-leave-active {
  transition: all 0.2s;
  opacity: 1;
}

.move-enter, .move-leave-to {
  transform: translate3d(0, 100%, 0);
  opacity: 0;
}

.fade-enter-active, .fade-leave-active {
  transition: all 0.5s;
  opacity: 1;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
