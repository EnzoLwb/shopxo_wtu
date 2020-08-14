<template>
<view>
<!-- 搜索 -->
<view v-if="common_app_is_header_nav_fixed == 1" class="search-fixed-seat"></view>
<view v-if="load_status == 1 && common_app_is_enable_search == 1" :class="'search wh-auto ' + (common_app_is_header_nav_fixed == 1 ? 'search-fixed' : '')">
  <view class="search-content bg-white oh">
    <icon type="search" size="16"></icon>
    <input type="text" confirm-type="search" placeholder="其实搜索很简单^_^！" class="wh-auto cr-888" @confirm="search_input_event"></input>
  </view>
</view>

<!-- 商城公告 -->
<view v-if="common_shop_notice != null">
  <view class="tips">{{common_shop_notice}}</view>
</view>

<!-- 轮播 -->
<component-banner :prop-data="banner_list"></component-banner>

<!-- 导航 -->
<component-icon-nav :prop-data="navigation"></component-icon-nav>

<!-- 限时秒杀 -->
<view v-if="common_app_is_limitedtimediscount == 1">
  <!--<import src="/pages/lib/limitedtimediscount/home.wxml"></import>-->
  <block data-type="template" data-is="limitedtimediscount" data-attr="plugins_limitedtimediscount_data: plugins_limitedtimediscount_data, plugins_limitedtimediscount_is_show_time: plugins_limitedtimediscount_is_show_time, plugins_limitedtimediscount_timer_title: plugins_limitedtimediscount_timer_title">
  <view v-if="plugins_limitedtimediscount_data.goods.length > 0" class="limitedtimediscount">
    <view class="nav-title">
      <image class="nav-icon" src="/static/pages/lib/limitedtimediscount/nav-icon.png" mode="aspectFit"></image>
      <text class="text-wrapper">限时秒杀</text>
      <view class="countdown">
        <block v-if="plugins_limitedtimediscount_is_show_time">
          <view class="timer-hours seconds">{{plugins_limitedtimediscount_data.time.seconds}}</view>
          <view class="ds">:</view>
          <view class="timer-hours minutes">{{plugins_limitedtimediscount_data.time.minutes}}</view>
          <view class="ds">:</view>
          <view class="timer-hours hours">{{plugins_limitedtimediscount_data.time.hours}}</view>
        </block>
        <view class="timer-title">{{plugins_limitedtimediscount_timer_title}}</view>
      </view>
    </view>
    <view class="goods-list">
      <scroll-view scroll-x>
        <view v-for="(item, index) in plugins_limitedtimediscount_data.goods" :key="index" class="item">
          <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + item.goods_id" hover-class="none">
            <image class="dis-block" :src="item.images" mode="aspectFit"></image>
            <view class="goods-base">
              <view class="goods-title single-text">{{item.title}}</view>
              <view class="goods-price single-text">￥{{item.price}}</view>
              <view v-if="(item.original_price || null) != null" class="goods-original-price single-text">￥{{item.original_price}}</view>
              <button size="mini">抢购</button>
            </view>
          </navigator>
        </view>
      </scroll-view>
    </view>
  </view>
</block>
</view>

<!-- 楼层数据 -->
<block v-if="data_list.length > 0">
  <view v-for="(floor, index) in data_list" :key="index" class="floor spacing-mb">
    <view class="spacing-nav-title">
      <text class="line"></text>
      <text class="text-wrapper">{{floor.name}}</text>
    </view>
    <view class="floor-list">
      <view class="word" :style="'background-color:' + floor.bg_color || '#eaeaea'">
        <view v-if="floor.items.length > 0">
          <block v-for="(icv, icx) in floor.items" :key="icx">
            <navigator v-if="icx < 6 && icv.is_home_recommended == 1" class="word-icon" :url="'/pages/goods-search/goods-search?category_id=' + icv.id" hover-class="none">
              {{icv.name}}
            </navigator>
          </block>
        </view>
        <view v-if="floor.describe.length > 0" class="vice-name">{{floor.describe}}</view>
        <navigator :url="'/pages/goods-search/goods-search?category_id=' + floor.id" hover-class="none">
          <image v-if="floor.big_images.length > 0" :src="floor.big_images" mode="aspectFit" class="dis-block"></image>
        </navigator>
      </view>
      <view class="goods-list" v-if="floor.goods.length > 0">
        <view v-for="(goods, index2) in floor.goods" :key="index2" class="goods bg-white">
          <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + goods.id" hover-class="none">
            <image :src="goods.home_recommended_images" mode="aspectFit"></image>
            <view class="goods-base">
              <view class="goods-title single-text">{{goods.title}}</view>
              <view class="sales-price">￥{{goods.min_price}}</view>
            </view>
          </navigator>
        </view>
      </view>
    </view>
  </view>
</block>
<view v-if="data_list.length == 0">
  <!--<import src="/pages/common/nodata.wxml"></import>-->
  <block data-type="template" data-is="nodata" data-attr="status: data_list_loding_status">
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

<!-- 留言 -->
<view v-if="load_status == 1 && common_app_is_enable_answer == 1" class="spacing-10">
  <navigator url="/pages/answer-form/answer-form" hover-class="none" class="bg-white">
    <image class="wh-auto" mode="widthFix" src="/static/images/home-consulting-image.jpg"></image>
  </navigator>
</view>

<!-- 结尾 -->
<!--<import src="/pages/common/bottom_line.wxml"></import>-->
<block data-type="template" data-is="bottom_line" data-attr="status: data_bottom_line_status">
    <view v-if="(status || false)" class="data-bottom-line">
        <view class="left fl"></view>
        <view class="msg fl">我是有底线的</view>
        <view class="right fr"></view>
    </view>
</block>

<!-- 在线客服 -->
<view v-if="common_app_is_online_service == 1">
  <!--<import src="/pages/lib/online-service/content.wxml"></import>-->
  <template is="online_service"></template>
</view>

<!-- 版权 -->
<view v-if="load_status == 1">
  <!--<import src="/pages/common/copyright.wxml"></import>-->
  <template is="copyright"></template>
</view>
</view>
</template>

<script>
const app = getApp();
import componentIconNav from "../../components/icon-nav/icon-nav";
import componentBanner from "../../components/slider/slider";

export default {
  data() {
    return {
      load_status: 0,
      data_list_loding_status: 1,
      data_bottom_line_status: false,
      data_list: [],
      banner_list: [],
      navigation: [],
      common_shop_notice: null,
      common_app_is_enable_search: 1,
      common_app_is_enable_answer: 1,
      common_app_is_header_nav_fixed: 0,
      common_app_is_online_service: 0,
      // 限时秒杀插件
      common_app_is_limitedtimediscount: 0,
      plugins_limitedtimediscount_data: null,
      plugins_limitedtimediscount_timer_title: '距离结束',
      plugins_limitedtimediscount_is_show_time: true,
      plugins_limitedtimediscount_timer: null
    };
  },

  components: {
    componentIconNav,
    componentBanner
  },
  props: {},

  onShow() {
    this.init();
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.init();
  },

  // 页面从前台变为后台时执行
  onHide: function () {
    clearInterval(this.plugins_limitedtimediscount_timer);
  },
  // 页面销毁时执行
  onUnload: function () {
    clearInterval(this.plugins_limitedtimediscount_timer);
  },

  // 自定义分享
  onShareAppMessage() {
    return {
      title: app.globalData.data.application_title,
      desc: app.globalData.data.application_describe,
      path: '/pages/index/index?share=index'
    };
  },

  methods: {
    // 获取数据列表
    init() {
      var self = this; // 加载loding

      this.setData({
        data_list_loding_status: 1
      }); // 加载loding

      wx.request({
        url: app.globalData.get_request_url("index", "index"),
        method: "POST",
        data: {},
        dataType: "json",
        success: res => {
          wx.stopPullDownRefresh();
          self.setData({
            load_status: 1
          });

          if (res.data.code == 0) {
            var data = res.data.data;
            self.setData({
              data_bottom_line_status: true,
              banner_list: data.banner_list || [],
              navigation: data.navigation || [],
              data_list: data.data_list,
              common_shop_notice: data.common_shop_notice || null,
              common_app_is_enable_search: data.common_app_is_enable_search,
              common_app_is_enable_answer: data.common_app_is_enable_answer,
              common_app_is_header_nav_fixed: data.common_app_is_header_nav_fixed,
              data_list_loding_status: data.data_list.length == 0 ? 0 : 3,
              common_app_is_online_service: data.common_app_is_online_service || 0,
              common_app_is_limitedtimediscount: data.common_app_is_limitedtimediscount || 0,
              plugins_limitedtimediscount_data: data.plugins_limitedtimediscount_data || null
            }); // 限时秒杀倒计时

            if (this.common_app_is_limitedtimediscount == 1 && this.plugins_limitedtimediscount_data != null) {
              this.plugins_limitedtimediscount_countdown();
            }
          } else {
            self.setData({
              data_list_loding_status: 0,
              data_bottom_line_status: true
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.stopPullDownRefresh();
          self.setData({
            data_list_loding_status: 2,
            data_bottom_line_status: true,
            load_status: 1
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 搜索事件
    search_input_event(e) {
      var keywords = e.detail.value || null;

      if (keywords == null) {
        app.globalData.showToast("请输入搜索关键字");
        return false;
      } // 进入搜索页面


      wx.navigateTo({
        url: '/pages/goods-search/goods-search?keywords=' + keywords
      });
    },

    // 显示秒杀插件-倒计时
    plugins_limitedtimediscount_countdown() {
      // 销毁之前的任务
      clearInterval(this.plugins_limitedtimediscount_timer); // 定时参数

      var status = this.plugins_limitedtimediscount_data.time.status || 0;
      var msg = this.plugins_limitedtimediscount_data.time.msg || '';
      var hours = parseInt(this.plugins_limitedtimediscount_data.time.hours) || 0;
      var minutes = parseInt(this.plugins_limitedtimediscount_data.time.minutes) || 0;
      var seconds = parseInt(this.plugins_limitedtimediscount_data.time.seconds) || 0;
      var self = this;

      if (status == 1) {
        // 秒
        var timer = setInterval(function () {
          if (seconds <= 0) {
            if (minutes > 0) {
              minutes--;
              seconds = 59;
            } else if (hours > 0) {
              hours--;
              minutes = 59;
              seconds = 59;
            }
          } else {
            seconds--;
          }

          self.setData({
            'plugins_limitedtimediscount_data.time.hours': hours < 10 ? '0' + hours : hours,
            'plugins_limitedtimediscount_data.time.minutes': minutes < 10 ? '0' + minutes : minutes,
            'plugins_limitedtimediscount_data.time.seconds': seconds < 10 ? '0' + seconds : seconds,
            plugins_limitedtimediscount_timer: timer
          });

          if (hours <= 0 && minutes <= 0 && seconds <= 0) {
            // 停止时间
            clearInterval(timer); // 活动已结束

            self.setData({
              plugins_limitedtimediscount_timer_title: '活动已结束',
              plugins_limitedtimediscount_is_show_time: false
            });
          }
        }, 1000);
      } else {
        // 活动已结束
        self.setData({
          plugins_limitedtimediscount_timer_title: msg,
          plugins_limitedtimediscount_is_show_time: false
        });
      }
    }

  }
};
</script>
<style>
@import "./index.css";
</style>