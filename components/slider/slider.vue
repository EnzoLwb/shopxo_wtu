<template>
<view>
<swiper :indicator-dots="propData.length > 0" :indicator-color="indicator_color" :indicator-active-color="indicator_active_color" :autoplay="propData.length > 0" :circular="circular" class="banner" v-if="propData.length > 0">
  <block v-for="(item, index) in propData" :key="index">
    <swiper-item>
      <image :src="item.images_url" mode="widthFix" :data-value="item.event_value" :data-type="item.event_type" @tap="banner_event"></image>
    </swiper-item>
  </block>
</swiper>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      indicator_dots: false,
      indicator_color: 'rgba(0, 0, 0, .3)',
      indicator_active_color: '#e31c55',
      autoplay: true,
      circular: true
    };
  },

  components: {},
  props: {
    propData: Array
  },
  methods: {
    banner_event(e) {
      app.globalData.operation_event(e);
    }

  }
};
</script>
<style>
@import "./slider.css";
</style>