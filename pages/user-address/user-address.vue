<template>
<view>
<view class="page">
  <view v-if="data_list.length > 0">
    <view v-for="(item, index) in data_list" :key="index" class="item bg-white spacing-mb">
      <view @tap="address_conent_event" :data-index="index">
        <view class="base oh">
          <text>{{item.name}}</text>
          <text class="fr">{{item.tel}}</text>
        </view>
        <view class="address oh">
          <image class="item-icon fl" src="/static/images/user-address.png" mode="widthFix"></image>
          <view class="text fr">{{item.province_name}}{{item.city_name}}{{item.county_name}}{{item.address}}</view>
        </view>
      </view>
      <view class="operation br-t oh">
        <view class="default fl" @tap="address_default_event" :data-value="item.id">
          <image v-if="is_default == item.id" class="item-icon" src="/static/images/default-select-active-icon.png" mode="widthFix"></image>
          <image v-else class="item-icon" src="/static/images/default-select-icon.png" mode="widthFix"></image>
          <text>设为默认地址</text>
        </view>
        <button class="fr cr-666 delete-submit br" type="default" size="mini" @tap="address_delete_event" :data-index="index" :data-value="item.id" hover-class="none">删除</button>
        <navigator :url="'/pages/user-address-save/user-address-save?id=' + item.id" open-type="navigate" hover-class="none">
          <button class="fr cr-666 br" type="default" size="mini" @tap="address_edit_event" hover-class="none">编辑</button>
        </navigator>
      </view>
    </view>
  </view>

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
    
  <!--<import src="/pages/common/bottom_line.wxml"></import>-->
  <block data-type="template" data-is="bottom_line" data-attr="status: data_bottom_line_status">
    <view v-if="(status || false)" class="data-bottom-line">
        <view class="left fl"></view>
        <view class="msg fl">我是有底线的</view>
        <view class="right fr"></view>
    </view>
</block>
  
  <navigator url="/pages/user-address-save/user-address-save" open-type="navigate" hover-class="none">
    <button class="submit-fixed submit-bottom" type="default" hover-class="none">新增地址</button>
  </navigator>
</view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      data_list_loding_status: 1,
      data_bottom_line_status: false,
      data_list: [],
      params: null,
      is_default: 0
    };
  },

  components: {},
  props: {},

  onLoad(params) {
    this.setData({
      params: params
    });
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.data.common_pages_title.user_address
    });
    this.init();
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.get_data_list();
  },

  methods: {
    // 初始化
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

    // 获取数据列表
    get_data_list() {
      // 加载loding
      wx.showLoading({
        title: "加载中..."
      });
      this.setData({
        data_list_loding_status: 1
      }); // 获取数据

      wx.request({
        url: app.globalData.get_request_url("index", "useraddress"),
        method: "POST",
        data: {},
        dataType: "json",
        success: res => {
          wx.hideLoading();
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            if (res.data.data.length > 0) {
              // 获取当前默认地址
              var is_default = 0;

              for (var i in res.data.data) {
                if (res.data.data[i]['is_default'] == 1) {
                  is_default = res.data.data[i]['id'];
                }
              } // 设置数据


              this.setData({
                data_list: res.data.data,
                is_default: is_default,
                data_list_loding_status: 3,
                data_bottom_line_status: true
              });
            } else {
              this.setData({
                data_list_loding_status: 0
              });
            }
          } else {
            this.setData({
              data_list_loding_status: 0
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.hideLoading();
          wx.stopPullDownRefresh();
          this.setData({
            data_list_loding_status: 2
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 删除地址
    address_delete_event(e) {
      var index = e.currentTarget.dataset.index;
      var value = e.currentTarget.dataset.value || null;

      if (value == null) {
        app.globalData.showToast("地址ID有误");
        return false;
      }

      var self = this;
      wx.showModal({
        title: "温馨提示",
        content: "删除后不可恢复，确定继续吗?",
        confirmText: "确认",
        cancelText: "不了",
        success: result => {
          if (result.confirm) {
            // 加载loding
            wx.showLoading({
              title: "处理中..."
            }); // 获取数据

            wx.request({
              url: app.globalData.get_request_url("delete", "useraddress"),
              method: "POST",
              data: {
                id: value
              },
              dataType: "json",
              success: res => {
                wx.hideLoading();

                if (res.data.code == 0) {
                  var temp_data = self.data_list;
                  temp_data.splice(index, 1);
                  self.setData({
                    data_list: temp_data,
                    data_list_loding_status: temp_data.length == 0 ? 0 : 3,
                    data_bottom_line_status: temp_data.length == 0 ? false : true
                  });
                  app.globalData.showToast(res.data.msg, "success"); // 当前删除是否存在缓存中，存在则删除

                  var cache_address = wx.getStorageSync(app.globalData.data.cache_buy_user_address_select_key);

                  if ((cache_address.data || null) != null) {
                    if (cache_address.data.id == value) {
                      // 删除地址缓存
                      wx.removeStorageSync(app.globalData.data.cache_buy_user_address_select_key);
                    }
                  }
                } else {
                  app.globalData.showToast(res.data.msg);
                }
              },
              fail: () => {
                wx.hideLoading();
                app.globalData.showToast("服务器请求出错");
              }
            });
          }
        }
      });
    },

    // 默认地址设置
    address_default_event(e) {
      var value = e.currentTarget.dataset.value || null;

      if (value == null) {
        app.globalData.showToast("地址ID有误");
        return false;
      }

      var self = this;

      if (value == self.is_default) {
        app.globalData.showToast("设置成功", "success");
        return false;
      } // 加载loding


      wx.showLoading({
        title: "处理中..."
      }); // 获取数据

      wx.request({
        url: app.globalData.get_request_url("setdefault", "useraddress"),
        method: "POST",
        data: {
          id: value
        },
        dataType: "json",
        success: res => {
          wx.hideLoading();

          if (res.data.code == 0) {
            self.setData({
              is_default: value
            });
            app.globalData.showToast(res.data.msg, "success");
          } else {
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.hideLoading();
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 地址内容事件
    address_conent_event(e) {
      var index = e.currentTarget.dataset.index || 0;
      var is_back = this.params.is_back || 0;

      if (is_back == 1) {
        wx.setStorage({
          key: app.globalData.data.cache_buy_user_address_select_key,
          data: this.data_list[index]
        });
        wx.navigateBack();
      }
    }

  }
};
</script>
<style>
@import "./user-address.css";
</style>