> uniapp中的swiper组件可以用来做滑动切屏的，但是有个不好的地方，就是必须设置一个固定的高度，对于在每一个swiper-item里的内容可能不一定的情况，就会造成内部的内容不能自动撑开，就被截取了，这个就很头疼，网上找了很多资料，终于解决了这个问题。

### 一、解决思路

> 1.  在每次滑动切换的时候，动态地获取swiper-item内部的DOM的元素的高度。
> 2.  将获取的高度动态设置给swiper元素。

### 二、代码解析

```vue
<template>
  <view>
    <swiper
      :autoplay="false"
      @change="changeSwiper"
      :current="currentIndex"
      :style="{ height: swiperHeight + 'px' }"
    >
      <swiper-item v-for="(item, index) in dataList" :key="item.id">
        <view :id="'content-wrap' + index">
            每一个swiper-item的内容区域
            ....
        </view>
      </swiper-item>
    </swiper>   
  </view>
</template>

<script>
export default {
  data() {
    return {
      //滑块的高度(单位px)
      swiperHeight: 0,
      //当前索引
      currentIndex: 0,
      //列表数据
      dataList: [],
    };
  },
  onLoad(args) {
    //动态设置swiper的高度
    this.$nextTick(() => {
      this.setSwiperHeight();
    });
  },
  methods: {
    //手动切换题目
    changeSwiper(e) {
      this.currentIndex = e.detail.current;
      //动态设置swiper的高度，使用nextTick延时设置
      this.$nextTick(() => {
        this.setSwiperHeight();
      });
    },
    //动态设置swiper的高度
    setSwiperHeight() {
      let element = "#content-wrap" + this.currentIndex;
      let query = uni.createSelectorQuery().in(this);
      query.select(element).boundingClientRect();
      query.exec((res) => {
        if (res && res[0]) {
          this.swiperHeight = res[0].height;
        }
      });
    },
  },
};
</script>

<style lang="scss">

</style>
```
