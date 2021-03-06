<!--轮播图组件开发-->
<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span
        class="dot"
        v-for="(dot,index) in dots"
        :key="dot"
        :class="{'active':index === currentPageIndex}"
      ></span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import BSscroll from "better-scroll";
export default {
  props: {
    loop: {
      type: Boolean,
      default: true
    },
    autoPlay: {
      type: Boolean,
      default: true
    },
    interval: {
      type: Number,
      default: 4000
    }
  },
  data() {
    return {
      dots: [],
      currentPageIndex: 0
    };
  },
  mounted() {
    this.$nextTick(() => {
      this._setSliderWidth();
      this._initDots();
      this._initBScroll();
      if (this.autoPlay) {
        this._play();
      }
    });
    // 窗口改变
    window.addEventListener("resize", () => {
      this._setSliderWidth(true);
      this.slider.refresh();
    });
  },
  activated() {
    if (!this.slider) {
      return;
    }
    this.slider.enable();
    let pageIndex = this.slider.getCurrentPage().pageX;
    this.slider.goToPage(pageIndex, 0, 0);
    this.currentPageIndex = pageIndex;
    if (this.autoPlay) {
      this._play();
    }
  },
  deactivated() {
    this.slider.disable();
    clearTimeout(this.timer);
  },
  beforeDestroy() {
    this.slider.disable();
    clearTimeout(this.timer);
  },
  methods: {
    _setSliderWidth(isResize) {
      let width = 0;
      let sliderWidth = this.$refs.slider.clientWidth;
      let children = this.$refs.sliderGroup.children;
      for (let i = 0; i < children.length; i++) {
        let child = children[i];
        child.classList.add("slider-item");
        child.style.width = sliderWidth + "px";
        width += sliderWidth;
      }
      if (this.loop && !isResize) {
        width += 2 * sliderWidth;
      }
      this.$refs.sliderGroup.style.width = width + "px";
    },
    _initDots() {
      // 初始化dots
      this.dots = new Array(this.$refs.sliderGroup.children.length);
    },
    _initBScroll() {
      this.slider = new BSscroll(this.$refs.slider, {
        scrollX: true,
        scrollY: false,
        momentum: false, //滑动惯性
        snap: {
          loop: this.loop, //无缝循环轮播
          threshold: 0.3, //用手指滑动时页面可切换的阈值，大于这个阈值可以滑动到下一页
          speed: 400 //轮播图切换的动画世界
        }
      });
      this.slider.on("scrollEnd", () => {
        let pageIndex = this.slider.getCurrentPage().pageX;
        this.currentPageIndex = pageIndex;
        if (this.autoPlay) {
          this._play();
        }
      });
    },
    _play() {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.slider.next();
      }, this.interval);
    }
  },
  destroyed(){
    clearTimeout(this.timer)
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable';

.slider {
  min-height: 1px;
  position: relative;
  overflow: hidden;

  .slider-group {
    position: relative;
    overflow: hidden;
    white-space: nowrap;

    .slider-item {
      float: left;
      box-sizing: border-box;
      overflow: hidden;
      text-align: center;

      a {
        display: block;
        width: 100%;
        overflow: hidden;
        text-decoration: none;

        img {
          display: block;
          width: 100%;
        }
      }
    }
  }

  .dots {
    position: absolute;
    right: 0;
    left: 0;
    bottom: 12px;
    text-align: center;
    font-size: 0;

    .dot {
      display: inline-block;
      margin: 0 4px;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: rgba(255,255,255,0.5);

      &.active {
        width: 15px;
        border-radius: 5px;
        background: rgba(255,255,255,0.8);
      }
    }
  }
}
</style>
