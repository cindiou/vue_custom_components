<template>
  <div
    class="swiper"
    @mouseenter="mouseEnterHandler"
    @mouseleave="mouseLeaveHandler"
    :style="{
      width: width + 'px',
      height: height + 'px',
    }"
  >
    <div
      class="imgs"
      ref="imgsContainer"
      :style="{
        transform: 'translateX(' + dist + 'px)',
        transition: switchTransition
          ? 'transform ' + duration + 'ms ease-out'
          : 'none',
      }"
    >
      <img
        :style="{ width: width, height: height }"
        class="single-img"
        v-for="(src, i) in iterImages"
        :src="src"
        :key="i"
      />
    </div>
    <div class="controller">
      <div class="arrows">
        <span class="arrow arrow-left" @click="prev">
          <slot name="arrow-left">《</slot>
        </span>
        <span class="arrow arrow-right" @click="next">
          <slot name="arrow-right">》</slot>
        </span>
      </div>
      <div class="dots" @click="dotsDelegateHandelr">
        <span
          :class="i === activeIndex ? 'active' : ''"
          v-for="(_, i) in imgs.length"
          :key="i"
          :data-index="i"
        ></span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "swiper",
  props: {
    imgs: {
      type: Array,
      required: true,
    },
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    duration: {
      type: Number,
      default: 1500,
    },
    shouldTransition: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      index: 0,
      timer: null,
      switchTransition: null,
    };
  },
  computed: {
    iterImages() {
      // imgs.length >= 2;
      const first = this.imgs[0];
      const last = this.imgs[this.imgs.length - 1];
      return [last, ...this.imgs, first];
    },
    dist() {
      return -(this.index + 1) * this.width;
    },
    activeIndex() {
      if (this.index === -1) {
        return this.imgs.length - 1;
      } else if (this.index === this.imgs.length) {
        return 0;
      } else {
        return this.index;
      }
    },
  },
  methods: {
    mouseEnterHandler() {
      clearInterval(this.timer);
    },
    mouseLeaveHandler() {
      this.timer = setInterval(() => {
        this.next();
        // 为什么要+100？
        // 因为在设置transiton-duration=interval-duration后;
        // 有可能存在:本次过渡还没完成（transitionend事件还没触发）
        // 但计时器又触发了下一次过渡，导致transitionend事件被累积；
        // 而我们要确保在第一或最后一张图片过渡完后，立即切换this.index、并确保在切换时过渡效果不生效;
        // 这个操作一定不能被累积、延迟，否则轮播图会出现一页或多页空白
      }, this.duration + 100);
    },
    dotsDelegateHandelr(e) {
      const val = e.target.dataset.index;
      if (!Object.is(val, NaN)) {
        this.index = Number(val);
      }
    },
    next() {
      this.index++;
      this.switchTransition = this.shouldTransition;
      if (this.index >= this.imgs.length) {
        if (this.shouldTransition) {
          // setTimeout(() => {
          //   // 复位；禁止过渡
          //   this.switchTransition = false;
          //   this.index = 0;
          // });
          this.$refs.imgsContainer.addEventListener(
            "transitionend",
            () => {
              // 这个过渡完成之后，立即复位；禁止过渡
              this.switchTransition = false;
              this.index = 0;
            },
            {
              once: true,
            }
          );
        } else {
          this.index = 0;
        }
      }
    },
    prev() {
      this.index--;
      this.switchTransition = this.shouldTransition;
      if (this.index <= -1) {
        if (this.shouldTransition) {
          this.$refs.imgsContainer.addEventListener(
            "transitionend",
            () => {
              this.index = this.imgs.length - 1;
              // 复位；禁止过渡
              this.switchTransition = false;
            },
            {
              once: true,
            }
          );
        } else {
          this.index = this.imgs.length - 1;
        }
      }
    },
  },
  mounted() {
    this.timer = setInterval(() => {
      this.next();
    }, this.duration + 100);
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
  activated(){
    this.timer = setInterval(() => {
      this.next();
    }, this.duration + 100);
  },
  deactivated(){
    clearInterval(this.timer);
  },
  watch: {},
};
</script>

<style scoped lang="scss">
.swiper {
  position: relative;
  overflow: hidden;
  .imgs {
    display: flex;
    height: inherit;
    .single-img {
      float: left;
    }
  }
  &:hover {
    .controller .arrows {
      display: flex;
    }
  }
  .controller {
    position: absolute;
    left: 0;
    top: 0;
    width: inherit;
    height: inherit;
    .arrows {
      position: absolute;
      top: 50%;
      left: 0;
      transform: translateY(-50%);
      width: inherit;
      display: none; /*只有在图片悬浮才显示*/
      /* display: flex; */
      justify-content: space-between;
      align-items: center;
      /* border: 1px solid red; */

      .arrow {
        cursor: pointer;
        &:hover {
          background-color: rgba(0, 0, 0, 0.8);
        }
        color: white;
        font-size: 2rem;
      }
      .arrow-left {
        margin-left: 10px;
      }
      .arrow-right {
        margin-right: 10px;
      }
    }
    .dots {
      z-index: 1;
      position: absolute;
      left: 50%;
      bottom: 10px;
      transform: translateX(-50%);
      display: inline;
      span {
        display: inline-block;
        margin-right: 2px;
        height: 8px;
        width: 12px;
        border-radius: 4px;
        background-color: white;

        transition: width 0.5s ease-out;
        &:hover {
          background-color: rgba(0, 0, 0, 0.2);
        }
        &.active {
          width: 40px;
        }
        &:not(:last-of-type) {
          margin-right: 8px;
        }
      }
    }
  }
}
</style>