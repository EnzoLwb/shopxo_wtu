<template>
<view>
<view class="left-nav">
  <block v-for="(item, index) in data_list" :key="index">
    <view :class="'items ' + item.active || ''" :data-index="index" @tap="nav_event">
      <text>{{item.name}}</text>
    </view>
  </block>
</view>
<view class="right-content bg-white">
  <block v-if="data_content.length > 0">
    <block v-for="(v, index) in data_content" :key="index">
      <view class="content-items" :data-value="v.id" @tap="category_event">
        <image v-if="(v.icon || null) != null" :src="v.icon" mode="aspectFit" class="icon"></image>
        <view class="text single-text">{{v.name}}</view>
      </view>
    </block>
  </block>
</view>

<view v-if="data_list.length == 0 && data_list_loding_status != 0">
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
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      tab_active: 0,
      tab_active_text_color: '#d2364c',
      tab_active_line_color: '#d2364c',
      data_list_loding_status: 1,
      data_bottom_line_status: false,
      data_list: [],
      data_content: []
    };
  },

  components: {},
  props: {},

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.goods_category
    });
    this.init();
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.init();
  },

  methods: {
    // 获取数据
    init() {
      // 加载loding
      this.setData({
        data_list_loding_status: 1
      }); // 加载loding

      wx.request({
        url: app.globalData.get_request_url("category", "goods"),
        method: "POST",
        data: {},
        dataType: "json",
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        success: res => {
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            var data = res.data.data;
            var data_content = [];

            if (data.length > 0) {
              data[0]['active'] = 'nav-active';
              data_content = data[0]['items'];
            }

            this.setData({
              data_list: data,
              data_content: data_content,
              data_list_loding_status: data.length == 0 ? 0 : 3,
              data_bottom_line_status: true
            });
          } else {
            this.setData({
              data_list_loding_status: 0,
              data_bottom_line_status: true
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.stopPullDownRefresh();
          this.setData({
            data_list_loding_status: 2,
            data_bottom_line_status: true
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 导航事件
    nav_event(e) {
      var index = e.currentTarget.dataset.index;
      var temp_data = this.data_list;

      for (var i in temp_data) {
        temp_data[i]['active'] = index == i ? 'nav-active' : '';
      }

      this.setData({
        data_list: temp_data,
        data_content: temp_data[index]['items']
      });
    },

    // 事件
    category_event(e) {
      wx.navigateTo({
        url: '/pages/goods-search/goods-search?category_id=' + e.currentTarget.dataset.value
      });
    }

  }
};
</script>
<style>
@import "./goods-category.css";
</style>