/**
  <slide
    :param="{}"
    :banners="[]"
  ></slide>
*/
<template>
  <div
    class="slide"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd">
    <ul
      class="slide__list"
      :style="{
        transform: `translateX(${translateX}px)`,
        transition: transition
      }">
      <li
        class="slide-list__item"
        v-for="(item, index) in banners"
        :key="index"
        :style="{ left: `${getLeft(index)}px` }">
        <a :href="item.link">
          <img :src="item.img">
        </a>
      </li>
      <div v-if="this.banners.length > 1 && banners.length < 3">
        <li
          class="slide-list__item"
          :style="{ left: `${getLeft(0) + boxSize}px` }">
          <a :href="banners[1].link">
            <img :src="banners[1].img">
          </a>
        </li>
        <li
          class="slide-list__item"
          :style="{ left: `${getLeft(1) - boxSize}px` }">
          <a :href="banners[0].link">
            <img :src="banners[0].img">
          </a>
        </li>
      </div>
    </ul>
    <div
      v-if="banners.length > 1"
      class="slide__dotbox">
      <span
        v-for="(item, index) in banners"
        :key="index"
        :class="index === activeIndex ? 'slide-dotbox__dot slide-dotbox__dot--active' : 'slide-dotbox__dot'">
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'slide',
  props: {
    param: {
      required: false,
      type: Object
    },
    banners: {
      required: true,
      type: Array
    }
  },
  data () {
    return {
      boxSize: document.documentElement.clientWidth,
      startX: 0, // touch X 起点
      startTime: 0, // touch start time
      translateX: 0, // 偏移量
      translateStartX: 0, // 偏移量后的 touch X 的起点
      direct: 1, // 方向，从左往右为 -1，从右往左为 1
      transition: '', // 在 touchend 时候添加过渡， touchmove 去除过渡
      activeIndex: 0, // 当前项
      round: 0, // 全部图片都播放过的轮数
      speed: 3000, // 自动播放间隔
      timer: '' // 计时器
    }
  },
  mounted () {
    if (this.banners.length < 2) return
    this.timer = setInterval(this.handleAutoPlay, this.speed)
  },
  methods: {
    // 获取 touch X 起点
    handleTouchStart (e) {
      if (this.banners.length < 2) return
      clearInterval(this.timer)
      this.startX = e.touches[0].pageX
      this.startTime = new Date().getTime()
      this.translateStartX = e.touches[0].pageX - this.translateX
      this.transition = ''
    },
    // 拖 move
    handleTouchMove (e) {
      if (this.banners.length < 2) return
      this.translateX = e.changedTouches[0].pageX - this.translateStartX
      this.direct = (e.changedTouches[0].pageX - this.startX) > 0 ? 1 : -1 // 从左往右为 -1，从右往左为 1
    },
    // 获取 touch X 终点
    handleTouchEnd (e) {
      if (this.banners.length < 2) return
      const speed = new Date().getTime() - this.startTime
      let translate
      if (speed < 200 && speed > 0) {
        translate = -this.direct * Math.floor(-this.direct * this.translateX / this.boxSize) * this.boxSize
      } else {
        translate = -this.direct * Math.round(-this.direct * this.translateX / this.boxSize) * this.boxSize
      }
      this.handlePlay(translate)
      this.timer = setInterval(this.handleAutoPlay, 3000)
    },
    // 轮播
    handlePlay (translate) {
      this.round = Math.ceil(translate / (this.boxSize * this.banners.length))
      this.translateX = translate
      this.transition = 'transform ease-out .3s'
      this.activeIndex = -(translate / this.boxSize) % this.banners.length >= 0 ? -((translate / this.boxSize) % this.banners.length) : (this.banners.length - (translate / this.boxSize) % this.banners.length)
    },
    // 自动播放，从右往左
    handleAutoPlay () {
      const translate = this.translateX - this.boxSize
      this.handlePlay(translate)
    },
    getLeft (index) {
      if (this.banners.length < 2) return
      const length = this.banners.length
      const active = this.activeIndex
      const round = this.round
      let left = 0

      if (active === 0 && index === (length - 1)) {
        left = index * this.boxSize - (1 + round) * length * this.boxSize
      } else if (active === (length - 1) && index === 0) {
        left = index * this.boxSize + (1 - round) * length * this.boxSize
      } else {
        left = index * this.boxSize - round * length * this.boxSize
      }
      return left
    }
  }
}
</script>

<style lang="less">
.slide {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;

  .slide__list {
    height: 100%;
    margin: 0;
    padding: 0;
    list-style: none;

    .slide-list__item {
      position: absolute;
      width: 100%;
      height: 100%;

      a {
        display: block;
        width: 100%;
        height: 100%;

        img {
          display: block;
          width: 100%;
          height: 100%;
        }
      }
    }
  }

  .slide__dotbox {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 20px;

    .slide-dotbox__dot {
      display: block;
      width: 20px;
      height: 4px;
      overflow: hidden;
      margin: 0 2px;
      background-color: rgba(0, 0, 0, 0.2);
    }

    .slide-dotbox__dot--active {
      background-color: rgba(0, 0, 0, 0.5);
    }
  }
}
</style>
