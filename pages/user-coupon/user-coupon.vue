<template>
<view>
<!-- 导航 -->
<view class="nav-tabs">
  <block v-for="(item, index) in nav_tabs_list" :key="index">
    <view :class="'nav-item cr-666 fl ' + (item.value == nav_tabs_value ? 'nav-item-active' : '')" :data-index="index" :data-value="item.value" @tap="nav_tabs_event">{{item.name}}</view>
  </block>
</view>

<scroll-view scroll-y="true" class="scroll-box" lower-threshold="30">
  <!-- 优惠劵列表 -->
  <view v-if="data_list != null" class="coupon-container">
    <!-- 未使用 -->
    <view v-if="(data_list.not_use || null) != null && data_list.not_use.length > 0 && nav_tabs_value == 'not_use'">
      <block v-for="(item, index) in data_list.not_use" :key="index">
        <view class="item spacing-mt bg-white" :style="'border:1px solid ' + item.coupon.bg_color_value + ';'">
          <view class="v-left fl">
            <view class="base single-text" :style="'color:' + item.coupon.bg_color_value + ';'">
              <text class="symbol">￥</text>
              <text class="price">{{item.coupon.discount_value}}</text>
              <text class="unit">{{item.coupon.type_unit}}</text>
              <text v-if="(item.coupon.desc || null) != null" class="desc cr-888">{{item.coupon.desc}}</text>
            </view>
            <view v-if="(item.coupon.use_limit_type_name || null) != null" class="base-tips cr-666 single-text">{{item.coupon.use_limit_type_name}}</view>
            <view class="base-time cr-888 single-text">{{item.time_start_text}} 至 {{item.time_end_text}}</view>
          </view>
          <navigator url="/pages/index/index" open-type="switchTab" hover-class="none">
            <view class="v-right fr" :style="'background:' + item.coupon.bg_color_value + ';'">
              <text class="circle"></text>
              <text>去使用</text>
            </view>
          </navigator>
        </view>
      </block>
    </view>

    <!-- 已使用 -->
    <view v-if="(data_list.already_use || null) != null && data_list.already_use.length > 0 && nav_tabs_value == 'already_use'">
      <block v-for="(item, index) in data_list.already_use" :key="index">
        <view class="item spacing-mt bg-white item-disabled">
          <view class="v-left fl">
            <view class="base single-text">
              <text class="symbol">￥</text>
              <text class="price">{{item.coupon.discount_value}}</text>
              <text class="unit">{{item.coupon.type_unit}}</text>
              <text v-if="(item.coupon.desc || null) != null" class="desc cr-888">{{item.coupon.desc}}</text>
            </view>
            <view v-if="(item.coupon.use_limit_type_name || null) != null" class="base-tips cr-666 single-text">{{item.coupon.use_limit_type_name}}</view>
            <view class="base-time cr-888 single-text">使用 {{item.use_time_text}}</view>
          </view>
          <view class="v-right fr">
            <text class="circle"></text>
            <text>已使用</text>
          </view>
        </view>
      </block>
    </view>

    <!-- 已过期 -->
    <view v-if="(data_list.already_expire || null) != null && data_list.already_expire.length > 0 && nav_tabs_value == 'already_expire'">
      <block v-for="(item, index) in data_list.already_expire" :key="index">
        <view class="item spacing-mt bg-white item-disabled">
          <view class="v-left fl">
            <view class="base single-text">
              <text class="symbol">￥</text>
              <text class="price">{{item.coupon.discount_value}}</text>
              <text class="unit">{{item.coupon.type_unit}}</text>
              <text v-if="(item.coupon.desc || null) != null" class="desc cr-888">{{item.coupon.desc}}</text>
            </view>
            <view v-if="(item.coupon.use_limit_type_name || null) != null" class="base-tips cr-666 single-text">{{item.coupon.use_limit_type_name}}</view>
            <view class="base-time cr-888 single-text">过期 {{item.time_end_text}}</view>
          </view>
          <view class="v-right fr">
            <text class="circle"></text>
            <text>已过期</text>
          </view>
        </view>
      </block>
    </view>
  </view>

  <view v-if="data_list_loding_status != 3">
  <!--<import src="/pages/common/nodata.wxml"></import>-->
  <block data-type="template" data-is="nodata" data-attr="status: data_list_loding_status, msg: data_list_loding_msg">
  <!-- 1 加载中 -->
  <view v-if="data_list_loding_status == 1" class="no-data-loding tc">
    <text>加载中...</text>
  </view>

  <!-- 2 处理错误 -->
  <view v-else-if="data_list_loding_status == 2" class="no-data-box tc">
    <image src="/static/images/error.png" mode="widthFix"></image>
    <view class="no-data-tips">{{msg || '处理错误'}}</view>
  </view>

  <!-- 0 默认没有数据 -->
  <view v-else-if="data_list_loding_status == 0" class="no-data-box tc">
    <image src="/static/images/empty.png" mode="widthFix"></image>
    <view class="no-data-tips">{{msg || '没有相关数据'}}</view>
  </view>
</block>
</view>

<!--<import src="/pages/common/bottom_line.wxml"></import>-->
<block data-type="template" data-is="bottom_line" data-attr="status: data_bottom_line_status">
    <view v-if="(status || false)" class="data-bottom-line">
        <view class="left fl"></view>
        <view class="msg fl">我是有底线的</view>
        <view class="right fr"></view>
    </view>
</block>
</scroll-view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      data_bottom_line_status: false,
      data_list_loding_status: 1,
      data_list_loding_msg: '',
      data_list: null,
      nav_tabs_list: [{
        name: "未使用",
        value: "not_use"
      }, {
        name: "已使用",
        value: "already_use"
      }, {
        name: "已过期",
        value: "already_expire"
      }],
      nav_tabs_value: 'not_use'
    };
  },

  components: {},
  props: {},

  onLoad(params) {
    this.init();
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.user_coupon
    });
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.get_data_list();
  },

  methods: {
    init() {
      var user = app.globalData.get_user_cache_info(this, "init"); // 用户未绑定用户则转到登录页面

      if (app.globalData.user_is_need_login(user)) {
        wx.redirectTo({
          url: "/pages/login/login?event_callback=init"
        });
        return false;
      } else {
        // 获取数据
        this.get_data_list();
      }
    },

    // 获取数据
    get_data_list() {
      var self = this;
      wx.showLoading({
        title: "加载中..."
      });

      if (this.data_list == null || (this.data_list[this.nav_tabs_value] || null) == null || this.data_list[this.nav_tabs_value].length <= 0) {
        this.setData({
          data_list_loding_status: 1
        });
      }

      wx.request({
        url: app.globalData.get_request_url("user", "coupon"),
        method: "POST",
        data: {},
        dataType: "json",
        success: res => {
          wx.hideLoading();
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            self.setData({
              data_list: res.data.data || null,
              data_list_loding_msg: ''
            });
            self.data_view_handle();
          } else {
            self.setData({
              data_bottom_line_status: false,
              data_list_loding_status: 2,
              data_list_loding_msg: res.data.msg
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.hideLoading();
          wx.stopPullDownRefresh();
          self.setData({
            data_bottom_line_status: false,
            data_list_loding_status: 2,
            data_list_loding_msg: '服务器请求出错'
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 数据处理
    data_view_handle() {
      var status = 0;

      if (this.data_list != null && (this.data_list[this.nav_tabs_value] || null) != null && this.data_list[this.nav_tabs_value].length > 0) {
        status = 3;
      }

      this.setData({
        data_list_loding_status: status,
        data_bottom_line_status: status == 3
      });
    },

    // 导航事件
    nav_tabs_event(e) {
      this.setData({
        nav_tabs_value: e.currentTarget.dataset.value
      });
      this.data_view_handle();
    }

  }
};
</script>
<style>
@import "./user-coupon.css";
</style>