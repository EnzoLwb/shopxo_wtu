<template>
<view>
<view v-if="goods_attribute.length > 0" class="goods-attribute bg-white">
  <view v-for="(item, index) in goods_attribute" :key="index" class="item br-b oh">
    <view class="title fl br-r">{{item.name}}</view>
    <view class="content cr-888 fl">
      <text v-for="(items, keys) in item.find" :key="keys">
        <text>{{items.name}}</text>
        <text v-if="keys < item.find.length-1"> , </text>
      </text>
    </view>
  </view>
</view>

<view class="nav-back tc wh-auto">
  <navigator url="/pages/goods-detail/goods-detail" open-type="navigateBack" hover-class="none">
    <button type="default" size="mini" class="cr-888" hover-class="none">返回</button>
  </navigator>
</view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      goods_attribute: []
    };
  },

  components: {},
  props: {},

  onLoad(params) {
    if ((params.data || null) == null) {
      wx.alert({
        title: '温馨提示',
        content: '属性数据有误',
        buttonText: '返回',
        success: () => {
          wx.navigateBack();
        }
      });
    } else {
      this.setData({
        goods_attribute: JSON.parse(params.data)
      });
    }
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.goods_attribute
    });
  },

  methods: {}
};
</script>
<style>
@import "./goods-attribute.css";
</style>