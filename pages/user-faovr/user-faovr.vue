<template>
<view>
<scroll-view scroll-y="true" class="scroll-box" @scrolltolower="scroll_lower" lower-threshold="30">
    <view class="list-item" v-if="data_list.length > 0">
      <view v-for="(item, index) in data_list" :key="index" class="goods-item oh bg-white spacing-mb">
        <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + item.goods_id" hover-class="none">
          <image class="goods-image fl" :src="item.images" mode="aspectFill"></image>
          <view class="goods-base">
            <view class="goods-title multi-text">{{item.title}}</view>
            <view class="oh goods-price">
              <text class="sales-price">￥{{item.price}}</text>
              <text v-if="item.original_price > 0" class="original-price">￥{{item.original_price}}</text>
            </view>
          </view>
        </navigator>
        <button class="submit-cancel" type="default" size="mini" @tap="cancel_event" :data-value="item.goods_id" :data-index="index" hover-class="none">取消</button>
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
</scroll-view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      data_list: [],
      data_page_total: 0,
      data_page: 1,
      data_list_loding_status: 1,
      data_bottom_line_status: false,
      data_total: ""
    };
  },

  components: {},
  props: {},

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.data.common_pages_title.user_favor
    });
    this.init();
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.setData({
      data_page: 1
    });
    this.get_data_list(1);
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
    get_data_list(is_mandatory) {
      // 分页是否还有数据
      if ((is_mandatory || 0) == 0) {
        if (this.data_bottom_line_status == true) {
          return false;
        }
      } // 加载loding


      wx.showLoading({
        title: "加载中..."
      });
      this.setData({
        data_list_loding_status: 1
      }); // 获取数据

      wx.request({
        url: app.globalData.get_request_url("index", "usergoodsfavor"),
        method: "POST",
        data: {
          page: this.data_page
        },
        dataType: "json",
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        success: res => {
          wx.hideLoading();
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            if (res.data.data.data.length > 0) {
              if (this.data_page <= 1) {
                var temp_data_list = res.data.data.data;
              } else {
                var temp_data_list = this.data_list;
                var temp_data = res.data.data.data;

                for (var i in temp_data) {
                  temp_data_list.push(temp_data[i]);
                }
              }

              this.setData({
                data_list: temp_data_list,
                data_total: res.data.data.total,
                data_page_total: res.data.data.page_total,
                data_list_loding_status: 3,
                data_page: this.data_page + 1
              }); // 是否还有数据

              if (this.data_page > 1 && this.data_page > this.data_page_total) {
                this.setData({
                  data_bottom_line_status: true
                });
              } else {
                this.setData({
                  data_bottom_line_status: false
                });
              }
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

    // 滚动加载
    scroll_lower(e) {
      this.get_data_list();
    },

    // 取消
    cancel_event(e) {
      wx.showModal({
        title: "温馨提示",
        content: "取消后不可恢复，确定继续吗?",
        confirmText: "确认",
        cancelText: "不了",
        success: result => {
          if (result.confirm) {
            // 参数
            var id = e.currentTarget.dataset.value;
            var index = e.currentTarget.dataset.index; // 加载loding

            wx.showLoading({
              title: "处理中..."
            });
            wx.request({
              url: app.globalData.get_request_url("cancel", "usergoodsfavor"),
              method: "POST",
              data: {
                id: id
              },
              dataType: "json",
              header: {
                'content-type': 'application/x-www-form-urlencoded'
              },
              success: res => {
                wx.hideLoading();

                if (res.data.code == 0) {
                  var temp_data_list = this.data_list;
                  temp_data_list.splice(index, 1);
                  this.setData({
                    data_list: temp_data_list
                  });

                  if (temp_data_list.length == 0) {
                    this.setData({
                      data_list_loding_status: 0,
                      data_bottom_line_status: false
                    });
                  }

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
          }
        }
      });
    }

  }
};
</script>
<style>
@import "./user-faovr.css";
</style>