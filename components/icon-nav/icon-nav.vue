<template>
<view v-if="propData.length > 0">
  <view class="data-list">
    <view v-for="(item, index) in propData" :key="index" class="items">
      <view class="items-content" :data-value="item.event_value" :data-type="item.event_type" @tap="navigation_event" :style="'background-color:' + item.bg_color">
        <image :src="item.images_url" mode="aspectFit"></image>
      </view>
      <view class="title">{{item.name}}</view>
    </view>
  </view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {};
  },

  components: {},
  props: {
    propData: Array
  },
  methods: {
    navigation_event(e) {
      app.globalData.operation_event(e);
    }

  }
};
</script>
<style>
@import "./icon-nav.css";
</style>