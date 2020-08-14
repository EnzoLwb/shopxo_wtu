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
        <text class="cr-666">{{item.add_time}}</text>
        <text class="fr cr-main">{{item.status_name}}</text>
      </view>
      <view v-for="(detail, index2) in item.items" :key="index2" class="goods-item oh">
        <navigator :url="'/pages/user-order-detail/user-order-detail?id=' + item.id" hover-class="none">
          <image class="goods-image fl" :src="detail.images" mode="aspectFill"></image>
          <view class="goods-base">
            <view class="goods-title multi-text">{{detail.title}}</view>
            <block v-if="detail.spec != null">
              <view v-for="(spec, index) in detail.spec" :key="index" class="goods-spec cr-888">
                {{spec.type}}:{{spec.value}}
              </view>
            </block>
            <view class="orderaftersale-btn-text" @tap.stop="orderaftersale_event" :data-oid="item.id" :data-did="detail.id">{{detail.orderaftersale_btn_text}}</view>
          </view>
          <view class="oh goods-price">
            <text class="sales-price">￥{{detail.price}}</text>
            <text v-if="detail.original_price > 0" class="original-price">￥{{detail.original_price}}</text>
            <text class="buy-number">x{{detail.buy_number}}</text>
          </view>
        </navigator>
      </view>
      <view class="item-describe tr cr-666">{{item.describe}}</view>
      <view v-if="item.status <= 3 || (item.status == 4 && item.user_is_comments == 0)" class="item-operation tr br-t">
        <button v-if="item.status <= 1" class="submit-cancel" type="default" size="mini" @tap="cancel_event" :data-value="item.id" :data-index="index" hover-class="none">取消</button>
        <button v-if="item.status == 1" class="submit-pay cr-666 br" type="default" size="mini" @tap="pay_event" :data-value="item.id" :data-index="index" hover-class="none">支付</button>
        <button v-if="item.status == 2" class="submit-rush cr-666 br" type="default" size="mini" @tap="rush_event" :data-value="item.id" :data-index="index" hover-class="none">催催</button>
        <button v-if="item.status == 3" class="submit-success cr-666 br" type="default" size="mini" @tap="collect_event" :data-value="item.id" :data-index="index" hover-class="none">收货</button>
        <button v-if="item.status == 4 && item.user_is_comments == 0" class="submit-success cr-666 br" type="default" size="mini" @tap="comments_event" :data-value="item.id" :data-index="index" hover-class="none">评论</button>
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
  </view>
</scroll-view>

<!-- 支付方式 popup -->
<component-popup :prop-show="is_show_payment_popup" prop-position="bottom" @onclose="payment_popup_event_close">
  <view v-if="payment_list.length > 0" class="payment-list oh bg-white">
    <view v-for="(item, index) in payment_list" :key="index" class="item tc fl">
      <view class="item-content br" :data-value="item.id" @tap="popup_payment_event">
        <image v-if="(item.logo || null) != null" class="icon" :src="item.logo" mode="widthFix"></image>
        <text>{{item.name}}
        </text>
      </view>
    </view>
  </view>
  <view v-else class="payment-list oh bg-white tc cr-888">没有支付方式</view>
</component-popup>
</view>
</template>

<script>
const app = getApp();
import componentPopup from "../../components/popup/popup";

export default {
  data() {
    return {
      data_list: [],
      data_page_total: 0,
      data_page: 1,
      data_list_loding_status: 1,
      data_bottom_line_status: false,
      params: null,
      input_keyword_value: '',
      load_status: 0,
      is_show_payment_popup: false,
      payment_list: [],
      payment_id: 0,
      temp_pay_value: 0,
      temp_pay_index: 0,
      nav_status_list: [{
        name: "全部",
        value: "-1"
      }, {
        name: "待付款",
        value: "1"
      }, {
        name: "待发货",
        value: "2"
      }, {
        name: "待收货",
        value: "3"
      }, {
        name: "已完成",
        value: "4"
      }, {
        name: "已失效",
        value: "5,6"
      }],
      nav_status_index: 0,
      data_total: ""
    };
  },

  components: {
    componentPopup
  },
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
      nav_status_index: nav_status_index
    });
    this.init();
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.user_order
    });
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

    // 输入框事件
    input_event(e) {
      this.setData({
        input_keyword_value: e.detail.value
      });
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

      var order_status = (this.nav_status_list[this.nav_status_index] || null) == null ? -1 : this.nav_status_list[this.nav_status_index]['value']; // 获取数据

      wx.request({
        url: app.globalData.get_request_url("index", "order"),
        method: "POST",
        data: {
          page: this.data_page,
          keywords: this.input_keyword_value || "",
          status: order_status,
          is_more: 1
        },
        dataType: "json",
        success: res => {
          wx.hideLoading();
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            if (res.data.data.data.length > 0) {
              if (this.data_page <= 1) {
                var temp_data_list = res.data.data.data; // 下订单支付处理

                if (this.load_status == 0) {
                  if ((this.params.is_pay || 0) == 1 && (this.params.order_id || 0) != 0) {
                    for (var i in temp_data_list) {
                      if (this.params.order_id == temp_data_list[i]['id']) {
                        this.pay_handle(this.params.order_id, i);
                        break;
                      }
                    }
                  }
                }
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
                data_page: this.data_page + 1,
                load_status: 1,
                payment_list: res.data.data.payment_list || []
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
                load_status: 1,
                data_list: [],
                data_bottom_line_status: false
              });
            }
          } else {
            this.setData({
              data_list_loding_status: 0,
              load_status: 1
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.hideLoading();
          wx.stopPullDownRefresh();
          this.setData({
            data_list_loding_status: 2,
            load_status: 1
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 滚动加载
    scroll_lower(e) {
      this.get_data_list();
    },

    // 支付
    pay_event(e) {
      this.setData({
        is_show_payment_popup: true,
        temp_pay_value: e.currentTarget.dataset.value,
        temp_pay_index: e.currentTarget.dataset.index
      });
    },

    // 支付弹窗关闭
    payment_popup_event_close(e) {
      this.setData({
        is_show_payment_popup: false
      });
    },

    // 支付弹窗发起支付
    popup_payment_event(e) {
      var payment_id = e.currentTarget.dataset.value || 0;
      this.setData({
        payment_id: payment_id
      });
      this.payment_popup_event_close();
      this.pay_handle(this.temp_pay_value, this.temp_pay_index);
    },

    // 支付方法
    pay_handle(order_id, index) {
      var self = this; // 加载loding

      wx.showLoading({
        title: "请求中..."
      });
      wx.request({
        url: app.globalData.get_request_url("pay", "order"),
        method: "POST",
        data: {
          id: order_id,
          payment_id: this.payment_id
        },
        dataType: "json",
        success: res => {
          wx.hideLoading();

          if (res.data.code == 0) {
            // 是否在线支付,非在线支付则支付成功
            if (res.data.data.is_online_pay == 0) {
              var temp_data_list = this.data_list;
              temp_data_list[index]['status'] = 2;
              temp_data_list[index]['status_name'] = '待发货';
              this.setData({
                data_list: temp_data_list
              });
              app.globalData.showToast("支付成功", "success");
            } else {
              wx.requestPayment({
                timeStamp: res.data.data.data.timeStamp,
                nonceStr: res.data.data.data.nonceStr,
                package: res.data.data.data.package,
                signType: res.data.data.data.signType,
                paySign: res.data.data.data.paySign,
                success: function (res) {
                  // 数据设置
                  var temp_data_list = self.data_list;
                  temp_data_list[index]['status'] = 2;
                  temp_data_list[index]['status_name'] = '待发货';
                  self.setData({
                    data_list: temp_data_list
                  }); // 跳转支付页面

                  wx.navigateTo({
                    url: "/pages/paytips/paytips?code=9000&total_price=" + self.data_list[index]['total_price']
                  });
                },
                fail: function (res) {
                  app.globalData.showToast('支付失败');
                }
              });
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
              url: app.globalData.get_request_url("cancel", "order"),
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
                  temp_data_list[index]['status_name'] = '已取消';
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

    // 收货
    collect_event(e) {
      wx.showModal({
        title: "温馨提示",
        content: "请确认已收到货物或已完成，操作后不可恢复，确定继续吗?",
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
              url: app.globalData.get_request_url("collect", "order"),
              method: "POST",
              data: {
                id: id
              },
              dataType: "json",
              success: res => {
                wx.hideLoading();

                if (res.data.code == 0) {
                  var temp_data_list = this.data_list;
                  temp_data_list[index]['status'] = 4;
                  temp_data_list[index]['status_name'] = '已完成';
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

    // 催催
    rush_event(e) {
      app.globalData.showToast("催促成功", "success");
    },

    // 导航事件
    nav_event(e) {
      this.setData({
        nav_status_index: e.currentTarget.dataset.index || 0,
        data_page: 1
      });
      this.get_data_list(1);
    },

    // 售后订单事件
    orderaftersale_event(e) {
      var oid = e.currentTarget.dataset.oid || 0;
      var did = e.currentTarget.dataset.did || 0;

      if (oid == 0 || did == 0) {
        app.globalData.showToast("参数有误");
        return false;
      } // 进入售后页面


      wx.navigateTo({
        url: "/pages/user-orderaftersale-detail/user-orderaftersale-detail?oid=" + oid + "&did=" + did
      });
    },

    // 订单评论
    comments_event(e) {
      wx.navigateTo({
        url: "/pages/user-order-comments/user-order-comments?id=" + e.currentTarget.dataset.value
      });
    }

  }
};
</script>
<style>
@import "./user-order.css";
</style>