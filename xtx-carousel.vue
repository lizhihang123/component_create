<template>
  <div class="xtx-carousel"
  @mouseover="stop()"
  @mouseleave="start()"
  >
    <ul class="carousel-body">
      <li class="carousel-item"
      v-for="(item,index) in slider"
      :key="index"
      :class="{fade: index===fade}"
      >
        <!-- 如果没有item.hrefUrl就是同类推荐  -->
        <RouterLink  v-if="item.hrefUrl" :to="item.hrefUrl">
          <img :src="item.imgUrl" alt="" />
        </RouterLink>
        <div v-else class="slider">
          <!-- 注意这里的数据结构 一个item里面就有四个商品 [[4], [4], [4], [4]] -->
          <RouterLink v-for="goods in item" :key="goods.id" :to="`product/${good.id}`">
            <img :src="goods.picture" />
            <p class="name ellipsis">{{goods.name}}</p>
            <!-- &yen; -》就是￥ -->
            <p class="price">&yen;{{goods.price}}</p>
          </RouterLink>
        </div>
      </li>
    </ul>
    <a href="javascript:;" class="carousel-btn prev">
      <i class="iconfont icon-angle-left"></i>
    </a>
    <a href="javascript:;" class="carousel-btn next">
      <i class="iconfont icon-angle-right"></i>
    </a>
    <div class="carousel-indicator">
      <span></span>
    </div>
  </div>
</template>

<script>
/*
1.实现哪些功能？

- 自动播放功能
- 鼠标进入，关闭自动播放；鼠标离开，开启自动播放
- 点击左右按钮 能够进行切换
- 销毁组件时 清除定时器

2.暴露哪些数据
slider - 图片数据
autoPlay - 是否开启自动播放
duration - 每张图片的持续时间

3.结构是怎么样的
ul>li
a>i
a>i
4.
第一步，渲染出li的数据
第二步，自动播放功能
第三步，鼠标进入，鼠标离开
第四步，点击左右按钮的切换 没能够完成 下面的小圆点也没有好
  */
import { ref, watch } from 'vue'
export default {
  name: 'XtxCarousel',
  props: {
    slider: {
      type: Array,
      default: () => []
    },
    autoPlay: {
      type: Boolean,
      default: false
    },
    duration: {
      type: Number,
      default: 1000
    }
  },
  setup(props) {
    // 控制显示的轮播图
    const fade = ref(null)
    // 1. 自动播放功能
    let timer = null
    const autoPlay = () => {
      clearInterval(timer)
      timer = setInterval(() => {
        fade.value++
        if (fade.value > props.slider.length) {
          fade.value = 0
        }
      }, props.duration)
    }

    // 2. 注意监听 一上来的时候 就判断是否要自动播放
    // slider有数据 && autoPlay是true
    watch(() => props.slider.value, (newValue) => {
      if (newValue && props.autoPlay) {
        // 默认显示第一张图片
        fade.value = 0
        autoPlay()
      }
    }, {
      immediate: true
    })
    // 3. 停止自动播放 鼠标经过 调用
    const stop = () => {
      clearInterval(timer)
    }
    // 3. 开始自动播放 鼠标离开调用
    const start = () => {
      if (props.slider && props.slider.length && props.autoPlay) {
        if (fade.value > props.slider.length) {
          // 如果说滚动到最后一张了 直接让他为0
          fade.value = 0
        }
        if (fade.value < 0) {
          fade.value = props.slider.length - 1
        }
        // 开启自动播放
        autoPlay()
      }
    }
    return {
      fade,
      stop,
      start
    }
  }
}
</script>
<style scoped lang="less">
.xtx-carousel {
  width: 100%;
  height: 100%;
  min-width: 300px;
  min-height: 150px;
  position: relative;
  .carousel {
    &-body {
      width: 100%;
      height: 100%;
    }
    &-item {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      opacity: 0;
      transition: opacity 0.5s linear;
      &.fade {
        opacity: 1;
        z-index: 1;
      }
      img {
        width: 100%;
        height: 100%;
      }
    }
    &-indicator {
      position: absolute;
      left: 0;
      bottom: 20px;
      z-index: 2;
      width: 100%;
      text-align: center;
      span {
        display: inline-block;
        width: 12px;
        height: 12px;
        background: rgba(0, 0, 0, 0.2);
        border-radius: 50%;
        cursor: pointer;
        ~ span {
          margin-left: 12px;
        }
        &.active {
          background: #fff;
        }
      }
    }
    &-btn {
      width: 44px;
      height: 44px;
      background: rgba(0, 0, 0, 0.2);
      color: #fff;
      border-radius: 50%;
      position: absolute;
      top: 228px;
      z-index: 2;
      text-align: center;
      line-height: 44px;
      opacity: 0;
      transition: all 0.5s;
      &.prev {
        left: 20px;
      }
      &.next {
        right: 20px;
      }
    }
  }
  &:hover {
    .carousel-btn {
      opacity: 1;
    }
  }
}

// 轮播商品
.slider {
  display: flex;
  justify-content: space-around;
  padding: 0 40px;
  > a {
    width: 240px;
    text-align: center;
    img {
      padding: 20px;
      width: 230px !important;
      height: 230px !important;
    }
    .name {
      font-size: 16px;
      color: #666;
      padding: 0 40px;
    }
    .price {
      font-size: 16px;
      color: @priceColor;
      margin-top: 15px;
    }
  }
}
</style>
