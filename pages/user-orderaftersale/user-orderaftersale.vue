<template>
<view>
<!-- 导航 -->
<view class="nav">
  <block v-for="(item, index) in nav_status_list" :key="index">
    <view v-if="nav_status_index == index" class="item fl tc cr-main" :data-index="index" @tap="nav_event">{{item.name}}</view>
    <view v-else class="item fl tc" :data-index="index" @tap="nav_event">{{item.name}}</view>
  </block>
</view>

<!-- 订单列表 -->
<scroll-view scroll-y="true" class="scroll-box" @scrolltolower="scroll_lower" lower-threshold="30">
  <view class="list-content">
    <view v-for="(item, index) in data_list" :key="index" class="list-item bg-white spacing-mb" v-if="data_list.length > 0">
      <view class="item-base oh br-b">
        <text class="cr-666">{{item.add_time_time}}</text>
        <text class="fr cr-main">{{item.status_text}}</text>
      </view>
      <view class="goods-item oh">
        <navigator :url="'/pages/user-orderaftersale-detail/user-orderaftersale-detail?oid=' + item.order_id + '&did=' + item.order_detail_id" hover-class="none">
          <image class="goods-image fl" :src="item.order_data.items.images" mode="aspectFill"></image>
          <view class="goods-base">
            <view class="goods-title multi-text">{{item.order_data.items.title}}</view>
            <block v-if="item.order_data.items.spec != null">
              <view v-for="(spec, index2) in item.order_data.items.spec" :key="index2" class="goods-spec cr-888">
                {{spec.type}}:{{spec.value}}
              </view>
            </block>
            <view class="orderaftersale-btn-text" @tap.stop="orderaftersale_event" :data-oid="item.id" :data-did="item.order_data.items.id">{{item.order_data.items.orderaftersale_btn_text}}</view>
          </view>
          <view class="oh goods-price">
            <text class="sales-price">￥{{item.order_data.items.price}}</text>
            <text v-if="item.order_data.items.original_price > 0" class="original-price">￥{{item.order_data.items.original_price}}</text>
            <text class="buy-number">x{{item.order_data.items.buy_number}}</text>
          </view>
        </navigator>
      </view>
      <view class="item-describe">
        <text class="cr-666">{{item.type_text}}</text>
        <text class="cr-ccc ds">/</text>
        <text class="cr-666">{{item.reason}}</text>
        <text v-if="item.price > 0" class="cr-ccc ds">/</text>
        <text v-if="item.price > 0" class="sales-price">￥{{item.price}}</text>
        <text v-if="item.number > 0" class="cr-main"> x{{item.number}}</text>
      </view>
      <view v-if="item.status <= 2 || item.status == 4" class="item-operation tr br-t">
        <button v-if="item.status != 3 && item.status != 5" class="submit-cancel" type="default" size="mini" @tap="cancel_event" :data-value="item.id" :data-index="index" hover-class="none">取消
        </button>
        <button v-if="item.status == 1 && item.type == 1" class="submit-pay cr-666 br" type="default" size="mini" @tap="delivery_event" :data-oid="item.order_id" :data-did="item.order_detail_id" :data-index="index" hover-class="none">退货</button>
      </view>
    </view>

    <view v-if="data_list.length == 0">
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
  </view>
</scroll-view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      params: null,
      data_list_loding_status: 1,
      data_list_loding_msg: '',
      data_bottom_line_status: false,
      // 接口数据
      data_list: [],
      data_page_total: 0,
      data_page: 1,
      form_keyword_value: '',
      // 导航
      // 状态（0待确认, 1待退货, 2待审核, 3已完成, 4已拒绝, 5已取消）
      nav_status_list: [{
        name: "全部",
        value: "-1"
      }, {
        name: "待确认",
        value: "0"
      }, {
        name: "待退货",
        value: "1"
      }, {
        name: "待审核",
        value: "2"
      }, {
        name: "已完成",
        value: "3"
      }, {
        name: "已拒绝",
        value: "4"
      }, {
        name: "已取消",
        value: "5"
      }],
      nav_status_index: 0,
      data_total: ""
    };
  },

  components: {},
  props: {},

  onLoad(params) {
    // 是否指定状态
    var nav_status_index = 0;

    if ((params.status || null) != null) {
      for (var i in this.nav_status_list) {
        if (this.nav_status_list[i]['value'] == params.status) {
          nav_status_index = i;
          break;
        }
      }
    }

    this.setData({
      params: params,
      form_keyword_value: params.keywords || '',
      nav_status_index: nav_status_index
    });
    this.init();
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.user_orderaftersale
    });
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.setData({
      data_page: 1
    });
    this.get_data_list(1);
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
      }); // 参数

      var status = (this.nav_status_list[this.nav_status_index] || null) == null ? -1 : this.nav_status_list[this.nav_status_index]['value'];
      wx.request({
        url: app.globalData.get_request_url("index", "orderaftersale"),
        method: "POST",
        data: {
          page: this.data_page,
          keywords: this.form_keyword_value || "",
          status: status,
          is_more: 1
        },
        dataType: "json",
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
                data_list_loding_status: 0,
                data_list_loding_msg: '没有相关数据',
                data_list: [],
                data_bottom_line_status: false
              });
            }
          } else {
            this.setData({
              data_list_loding_status: 0,
              data_list_loding_msg: res.data.msg
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.hideLoading();
          wx.stopPullDownRefresh();
          self.setData({
            data_list_loding_status: 2,
            data_list_loding_msg: '服务器请求出错'
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 滚动加载
    scroll_lower(e) {
      this.get_data_list();
    },

    // 导航事件
    nav_event(e) {
      this.setData({
        nav_status_index: e.currentTarget.dataset.index || 0,
        data_page: 1
      });
      this.get_data_list(1);
    },

    // 输入框事件
    input_event(e) {
      this.setData({
        form_keyword_value: e.detail.value
      });
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
              url: app.globalData.get_request_url("cancel", "orderaftersale"),
              method: "POST",
              data: {
                id: id
              },
              dataType: "json",
              success: res => {
                wx.hideLoading();

                if (res.data.code == 0) {
                  var temp_data_list = this.data_list;
                  temp_data_list[index]['status'] = 5;
                  temp_data_list[index]['status_text'] = '已取消';
                  this.setData({
                    data_list: temp_data_list
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
          }
        }
      });
    },

    // 退货
    delivery_event(e) {
      var oid = e.currentTarget.dataset.oid || 0;
      var did = e.currentTarget.dataset.did || 0;

      if (oid == 0 || did == 0) {
        app.globalData.showToast("参数有误");
        return false;
      } // 进入售后页面


      wx.navigateTo({
        url: "/pages/user-orderaftersale-detail/user-orderaftersale-detail?oid=" + oid + "&did=" + did + "&is_delivery_popup=1"
      });
    }

  }
};
</script>
<style>
@import "./user-orderaftersale.css";
</style>