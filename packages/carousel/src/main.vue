<template>
  <div
    :class="carouselClasses"
    @mouseenter.stop="handleMouseEnter"
    @mouseleave.stop="handleMouseLeave">
    <div
      class="el-carousel__container"
      :style="{ height: height }">
      <transition
        v-if="arrowDisplay"
        name="carousel-arrow-left">
        <button
          type="button"
          v-show="(arrow === 'always' || hover) && (loop || activeIndex > 0)"
          @mouseenter="handleButtonEnter('left')"
          @mouseleave="handleButtonLeave"
          @click.stop="throttledArrowClick(activeIndex - 1)"
          class="el-carousel__arrow el-carousel__arrow--left">
          <i class="el-icon-arrow-left"></i>
        </button>
      </transition>
      <transition
        v-if="arrowDisplay"
        name="carousel-arrow-right">
        <button
          type="button"
          v-show="(arrow === 'always' || hover) && (loop || activeIndex < items.length - 1)"
          @mouseenter="handleButtonEnter('right')"
          @mouseleave="handleButtonLeave"
          @click.stop="throttledArrowClick(activeIndex + 1)"
          class="el-carousel__arrow el-carousel__arrow--right">
          <i class="el-icon-arrow-right"></i>
        </button>
      </transition>
      <slot></slot>
    </div>
    <ul
      v-if="indicatorPosition !== 'none'"
      :class="indicatorsClasses">
      <li
        v-for="(item, index) in items"
        :key="index"
        :class="[
          'el-carousel__indicator',
          'el-carousel__indicator--' + direction,
          { 'is-active': index === activeIndex }]"
        @mouseenter="throttledIndicatorHover(index)"
        @click.stop="handleIndicatorClick(index)">
        <button class="el-carousel__button">
          <span v-if="hasLabel">{{ item.label }}</span>
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
import throttle from 'throttle-debounce/throttle';
import { addResizeListener, removeResizeListener } from 'element-ui/src/utils/resize-event';

export default {
  name: 'ElCarousel',

  props: {
    initialIndex: {
      type: Number,
      default: 0
    },
    height: String,
    trigger: {
      type: String,
      default: 'hover'
    },
    autoplay: {
      type: Boolean,
      default: true
    },
    interval: {
      type: Number,
      default: 3000
    },
    indicatorPosition: String,
    indicator: {
      type: Boolean,
      default: true
    },
    arrow: {
      type: String,
      default: 'hover'
    },
    type: String,
    loop: {
      type: Boolean,
      default: true
    },
    direction: {
      type: String,
      default: 'horizontal',
      validator(val) {
        return ['horizontal', 'vertical'].indexOf(val) !== -1;
      }
    }
  },

  data() {
    return {
      items: [],
      activeIndex: -1,
      containerWidth: 0,
      timer: null,
      hover: false
    };
  },

  computed: {
    // 指示器的显示
    arrowDisplay() {
      return this.arrow !== 'never' && this.direction !== 'vertical';
    },

    // 控制指示器的文本显示
    hasLabel() {
      return this.items.some(item => item.label.toString().length > 0);
    },

    // 设置幻灯片的样式
    carouselClasses() {
      const classes = ['el-carousel', 'el-carousel--' + this.direction];
      if (this.type === 'card') {
        classes.push('el-carousel--card');
      }
      return classes;
    },

    // 设置指示器的样式
    indicatorsClasses() {
      const classes = ['el-carousel__indicators', 'el-carousel__indicators--' + this.direction];
      if (this.hasLabel) {
        classes.push('el-carousel__indicators--labels');
      }
      if (this.indicatorPosition === 'outside' || this.type === 'card') {
        classes.push('el-carousel__indicators--outside');
      }
      return classes;
    }
  },

  watch: {
    // 监听幻灯片数量，设置选中的幻灯片
    items(val) {
      if (val.length > 0) this.setActiveItem(this.initialIndex);
    },

    // 监听选中的幻灯片索引，重置幻灯片位置，触发change事件
    activeIndex(val, oldVal) {
      this.resetItemPosition(oldVal);
      if (oldVal > -1) {
        this.$emit('change', val, oldVal);
      }
    },

    // 监听自动播放状态，暂停或者启动幻灯片
    autoplay(val) {
      val ? this.startTimer() : this.pauseTimer();
    },

    // 监听是否重复播放，设置选中的幻灯片
    loop() {
      this.setActiveItem(this.activeIndex);
    },

    // 监听自动切换间隔，切换幻灯片
    interval() {
      this.pauseTimer();
      this.startTimer();
    }
  },

  methods: {
    // 鼠标移入时暂停幻灯片播放
    handleMouseEnter() {
      this.hover = true;
      this.pauseTimer();
    },

    // 鼠标移出时启动幻灯片播放
    handleMouseLeave() {
      this.hover = false;
      this.startTimer();
    },

    // 判断当前幻灯片时左移还是右移
    itemInStage(item, index) {
      const length = this.items.length;
      if (index === length - 1 && item.inStage && this.items[0].active ||
        (item.inStage && this.items[index + 1] && this.items[index + 1].active)) {
        return 'left';
      } else if (index === 0 && item.inStage && this.items[length - 1].active ||
        (item.inStage && this.items[index - 1] && this.items[index - 1].active)) {
        return 'right';
      }
      return false;
    },

    // 当鼠标移入时切换幻灯片按钮的hover状态
    handleButtonEnter(arrow) {
      if (this.direction === 'vertical') return;
      this.items.forEach((item, index) => {
        if (arrow === this.itemInStage(item, index)) {
          item.hover = true;
        }
      });
      console.log(11);
    },

    // 当鼠标移出时切换幻灯片按钮的hover状态
    handleButtonLeave() {
      if (this.direction === 'vertical') return;
      this.items.forEach(item => {
        item.hover = false;
      });
    },

    // 更新幻灯片数量
    updateItems() {
      this.items = this.$children.filter(child => child.$options.name === 'ElCarouselItem');
    },

    // 重置幻灯片位置
    resetItemPosition(oldIndex) {
      this.items.forEach((item, index) => {
        // 计算幻灯片的样式
        item.translateItem(index, this.activeIndex, oldIndex);
      });
    },

    // 设置当前选中幻灯片的索引
    playSlides() {
      if (this.activeIndex < this.items.length - 1) {
        this.activeIndex++;
      } else if (this.loop) {
        this.activeIndex = 0;
      }
    },

    // 暂停幻灯片播放
    pauseTimer() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },

    // 启动幻灯片播放
    startTimer() {
      if (this.interval <= 0 || !this.autoplay || this.timer) return;
      this.timer = setInterval(this.playSlides, this.interval);
    },

    // 重置幻灯片
    resetTimer() {
      this.pauseTimer();
      this.startTimer();
    },

    // 设置选中的幻灯片
    setActiveItem(index) {
      // 如果index是字符串，则是用户设置了幻灯片的name
      if (typeof index === 'string') {
        // 找到对应name的幻灯片
        const filteredItems = this.items.filter(item => item.name === index);
        // 如果找到的items长度大于0，取第一个的索引作为我们要使用的索引
        if (filteredItems.length > 0) {
          index = this.items.indexOf(filteredItems[0]);
        }
      }
      // 索引转成数字
      index = Number(index);
      if (isNaN(index) || index !== Math.floor(index)) {
        console.warn('[Element Warn][Carousel]index must be an integer.');
        return;
      }
      let length = this.items.length;
      // 保存当前的幻灯片索引
      const oldIndex = this.activeIndex;
      // 判断index的大小
      if (index < 0) {
        this.activeIndex = this.loop ? length - 1 : 0;
      } else if (index >= length) {
        this.activeIndex = this.loop ? 0 : length - 1;
      } else {
        this.activeIndex = index;
      }
      if (oldIndex === this.activeIndex) {
        // 重置幻灯片位置
        this.resetItemPosition(oldIndex);
      }
      this.resetTimer();
    },

    // 上一个幻灯片
    prev() {
      this.setActiveItem(this.activeIndex - 1);
    },

    // 下一个幻灯片
    next() {
      this.setActiveItem(this.activeIndex + 1);
    },

    // 指示器点击
    handleIndicatorClick(index) {
      this.activeIndex = index;
    },

    // 指示器hover，切换幻灯片
    handleIndicatorHover(index) {
      if (this.trigger === 'hover' && index !== this.activeIndex) {
        this.activeIndex = index;
      }
    }
  },

  created() {
    // 节流，点击
    this.throttledArrowClick = throttle(300, true, index => {
      this.setActiveItem(index);
    });
    // 节流，hover
    this.throttledIndicatorHover = throttle(300, index => {
      this.handleIndicatorHover(index);
    });
  },

  mounted() {
    this.updateItems();
    this.$nextTick(() => {
      // 监听页面的缩放，重置幻灯片位置
      addResizeListener(this.$el, this.resetItemPosition);
      // 设置当前的幻灯片索引
      if (this.initialIndex < this.items.length && this.initialIndex >= 0) {
        this.activeIndex = this.initialIndex;
      }
      //  启动幻灯片播放
      this.startTimer();
    });
  },

  beforeDestroy() {
    // 移除页面的缩放监听器
    if (this.$el) removeResizeListener(this.$el, this.resetItemPosition);
    // 暂停幻灯片播放
    this.pauseTimer();
  }
};
</script>
