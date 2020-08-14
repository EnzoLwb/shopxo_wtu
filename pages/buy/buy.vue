<template>
<view>
<view v-if="goods_list.length > 0" class="page">
  <!-- 地址 -->
  <view class="address bg-white arrow-right">
    <navigator url="/pages/user-address/user-address?is_back=1" hover-class="none">
      <view v-if="address != null">
        <view class="address-base oh">
          <text>{{address.name}}</text>
          <text class="fr">{{address.tel}}</text>
        </view>
        <view class="address-detail oh">
          <image class="icon fl" src="/static/images/user-address.png" mode="widthFix"></image>
          <view class="text fr">{{address.province_name}}{{address.city_name}}{{address.county_name}}{{address.address}}</view>
        </view>
      </view>
      <view v-if="address == null" class="no-address cr-888">
        请选择地址
      </view>
    </navigator>
  </view>
  <view class="address-divider spacing-mb"></view>

  <!-- 商品 -->
  <view class="goods bg-white spacing-mb">
    <view v-for="(item, index) in goods_list" :key="index" class="goods-item oh">
      <image class="goods-image fl" :src="item.images" mode="aspectFill"></image>
      <view class="goods-base">
        <view class="goods-title multi-text">{{item.title}}</view>
        <block v-if="item.spec != null">
          <view v-for="(spec, index2) in item.spec" :key="index2" class="goods-spec cr-888">{{spec.type}}:{{spec.value}}
          </view>
        </block>
      </view>
      <view class="oh goods-price">
        <text class="sales-price">￥{{item.price}}
        </text>
        <text v-if="item.original_price > 0" class="original-price">￥{{item.original_price}}
        </text>
        <text class="buy-number cr-888">x{{item.stock}}
        </text>
      </view>
    </view>
  </view>

  <!-- 留言 -->
  <view class="content-textarea-view bg-white spacing-mb">
    <textarea v-if="!popup_plugins_coupon_status" @input="bind_user_note_event" :value="user_note_value" maxlength="60" placeholder="留言" class="wh-auto"></textarea>
    <view v-if="popup_plugins_coupon_status" class="cr-888">{{user_note_value || '留言'}}</view>
  </view>

  <!-- 优惠劵 -->
  <view v-if="(plugins_coupon_data || null) != null && plugins_coupon_data.coupon_list.length > 0" class="plugins-coupon bg-white spacing-mb arrow-right" @tap="plugins_coupon_open_event">
    <text class="cr-666">优惠劵</text>
    <text class="cr-ccc fr">{{plugins_choice_coupon_value}}</text>
  </view>

  <!-- 扩展数据展示 -->
  <view v-if="extension_data.length > 0" class="extension-list spacing-mb">
    <view v-for="(item, index) in extension_data" :key="index" class="item oh">
      <text class="cr-666 fl">{{item.name}}
      </text>
      <text class="text-tips fr">{{item.tips}}
      </text>
    </view>
  </view>

  <!-- 支付方式 -->
  <view v-if="payment_list.length > 0 && common_order_is_booking != 1" class="payment-list bg-white oh">
    <view v-for="(item, index) in payment_list" :key="index" class="item tc fl">
      <view :class="'item-content br ' + (item.selected || '')" :data-value="item.id" @tap="payment_event">
        <image v-if="(item.logo || null) != null" class="icon" :src="item.logo" mode="widthFix"></image>
        <text>{{item.name}}</text>
      </view>
    </view>
  </view>

  <!-- 导航 -->
  <view class="buy-nav oh wh-auto">
    <view class="nav-base bg-white fl single-text">
      <text>合计：</text>
      <text class="sales-price">￥{{total_price}}</text>
    </view>
    <view class="fr nav-submit">
      <button class="bg-main wh-auto" type="default" @tap="buy_submit_event" :disabled="buy_submit_disabled_status" hover-class="none">提交订单</button>
    </view>
  </view>
</view>

<view v-if="goods.length == 0">
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

<!-- 优惠劵选择 -->
<component-popup :prop-show="popup_plugins_coupon_status" prop-position="bottom" @onclose="plugins_coupon_close_event">
  <view class="plugins-coupon-popup bg-white">
    <view class="close oh">
      <view class="fr" @tap.stop="plugins_coupon_close_event">
        <icon type="clear" size="20"></icon>
      </view>
    </view>
    <view v-if="(plugins_coupon_data || null) != null && plugins_coupon_data.coupon_list.length > 0" class="coupon-container oh br-b">
      <view class="not-use-tips tc">
        <text @tap="plugins_coupon_not_use_event">不使用优惠劵</text>
      </view>
      <block v-for="(item, index) in plugins_coupon_data.coupon_list" :key="index">
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
          <view class="v-right fr" :style="'background:' + item.coupon.bg_color_value + ';'" :data-index="index" :data-value="item.id" @tap="plugins_coupon_use_event">
            <text class="circle"></text>
            <text>{{plugins_use_coupon_id == item.id ? '已选' : '选择'}}</text>
          </view>
        </view>
      </block>
    </view>
  </view>
</component-popup>
</view>
</template>

<script>
const app = getApp();
import componentPopup from "../../components/popup/popup";

export default {
  data() {
    return {
      data_list_loding_status: 1,
      buy_submit_disabled_status: false,
      data_list_loding_msg: '',
      params: null,
      payment_list: [],
      goods_list: [],
      address: null,
      address_id: 0,
      total_price: 0,
      user_note_value: '',
      is_first: 1,
      extension_data: [],
      payment_id: 0,
      common_order_is_booking: 0,
      // 优惠劵
      plugins_coupon_data: null,
      plugins_use_coupon_id: 0,
      plugins_choice_coupon_value: '选择优惠劵',
      popup_plugins_coupon_status: false
    };
  },

  components: {
    componentPopup
  },
  props: {},

  onLoad(params) {
    if ((params.data || null) == null || app.globalData.get_length(JSON.parse(params.data)) == 0) {
      wx.alert({
        title: '温馨提示',
        content: '订单信息有误',
        buttonText: '确认',
        success: () => {
          wx.navigateBack();
        }
      });
    } else {
      this.setData({
        params: JSON.parse(params.data)
      }); // 删除地址缓存

      wx.removeStorageSync(app.globalData.data.cache_buy_user_address_select_key);
    }
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.data.common_pages_title.buy
    });
    this.init();
    this.setData({
      is_first: 0
    });
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.init();
  },

  methods: {
    // 获取数据列表
    init() {
      // 本地缓存地址
      if (this.is_first == 0) {
        var cache_address = wx.getStorageSync(app.globalData.data.cache_buy_user_address_select_key);

        if ((cache_address || null) != null) {
          this.setData({
            address: cache_address,
            address_id: cache_address.id
          });
        } else {
          this.setData({
            address: null,
            address_id: 0
          });
        }
      } // 加载loding


      wx.showLoading({
        title: '加载中...'
      });
      this.setData({
        data_list_loding_status: 1
      });
      var data = this.params;
      data['address_id'] = this.address_id;
      data['payment_id'] = this.payment_id;
      data['coupon_id'] = this.plugins_use_coupon_id;
      wx.request({
        url: app.globalData.get_request_url("index", "buy"),
        method: "POST",
        data: data,
        dataType: "json",
        success: res => {
          wx.stopPullDownRefresh();
          wx.hideLoading();

          if (res.data.code == 0) {
            var data = res.data.data;

            if (data.goods_list.length == 0) {
              this.setData({
                data_list_loding_status: 0
              });
            } else {
              this.setData({
                goods_list: data.goods_list,
                total_price: data.base.actual_price,
                extension_data: data.extension_data || [],
                data_list_loding_status: 3,
                common_order_is_booking: data.common_order_is_booking || 0,
                plugins_coupon_data: data.plugins_coupon_data || null
              }); // 优惠劵选择处理

              if ((data.plugins_coupon_data || null) != null) {
                if ((data.plugins_coupon_data.coupon_choice || null) != null) {
                  this.setData({
                    plugins_choice_coupon_value: data.plugins_coupon_data.coupon_choice.coupon.desc
                  });
                } else {
                  var coupon_count = (data.plugins_coupon_data.coupon_list || null) != null ? data.plugins_coupon_data.coupon_list.length : 0;
                  this.setData({
                    plugins_choice_coupon_value: coupon_count > 0 ? '可选优惠劵' + coupon_count + '张' : '暂无可用优惠劵'
                  });
                }
              } // 地址


              if (this.address == null || this.address_id == 0) {
                if ((data.base.address || null) != null) {
                  this.setData({
                    address: data.base.address,
                    address_id: data.base.address.id
                  });
                  wx.setStorage({
                    key: app.globalData.data.cache_buy_user_address_select_key,
                    data: data.base.address
                  });
                }
              } // 支付方式


              this.payment_list_data(data.payment_list);
            }
          } else {
            this.setData({
              data_list_loding_status: 2,
              data_list_loding_msg: res.data.msg
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.stopPullDownRefresh();
          wx.hideLoading();
          this.setData({
            data_list_loding_status: 2,
            data_list_loding_msg: '服务器请求出错'
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 用户留言事件
    bind_user_note_event(e) {
      this.setData({
        user_note_value: e.detail.value
      });
    },

    // 提交订单
    buy_submit_event(e) {
      // 表单数据
      var data = this.params;
      data['address_id'] = this.address_id;
      data['payment_id'] = this.payment_id;
      data['user_note'] = this.user_note_value;
      data['coupon_id'] = this.plugins_use_coupon_id; // 数据验证

      var validation = [{
        fields: 'address_id',
        msg: '请选择地址'
      }];

      if (this.common_order_is_booking != 1) {
        validation.push({
          fields: 'payment_id',
          msg: '请选择支付方式'
        });
      }

      if (app.globalData.fields_check(data, validation)) {
        // 加载loding
        wx.showLoading({
          title: '提交中...'
        });
        this.setData({
          buy_submit_disabled_status: true
        });
        wx.request({
          url: app.globalData.get_request_url("add", "buy"),
          method: "POST",
          data: data,
          dataType: "json",
          success: res => {
            wx.hideLoading();

            if (res.data.code == 0) {
              if (res.data.data.order.status == 1) {
                wx.redirectTo({
                  url: '/pages/user-order/user-order?is_pay=1&order_id=' + res.data.data.order.id
                });
              } else {
                wx.redirectTo({
                  url: '/pages/user-order/user-order'
                });
              }
            } else {
              app.globalData.showToast(res.data.msg);
              this.setData({
                buy_submit_disabled_status: false
              });
            }
          },
          fail: () => {
            wx.hideLoading();
            this.setData({
              buy_submit_disabled_status: false
            });
            app.globalData.showToast("服务器请求出错");
          }
        });
      }
    },

    // 支付方式选择
    payment_event(e) {
      this.setData({
        payment_id: e.currentTarget.dataset.value
      });
      this.payment_list_data(this.payment_list);
      this.init();
    },

    // 支付方式数据处理
    payment_list_data(data) {
      if (this.payment_id != 0) {
        for (var i in data) {
          if (data[i]['id'] == this.payment_id) {
            data[i]['selected'] = 'selected';
          } else {
            data[i]['selected'] = '';
          }
        }
      }

      this.setData({
        payment_list: data || []
      });
    },

    // 优惠劵弹层开启
    plugins_coupon_open_event(e) {
      this.setData({
        popup_plugins_coupon_status: true
      });
    },

    // 优惠劵弹层关闭
    plugins_coupon_close_event(e) {
      this.setData({
        popup_plugins_coupon_status: false
      });
    },

    // 优惠劵选择
    plugins_coupon_use_event(e) {
      var index = e.currentTarget.dataset.index;
      var value = e.currentTarget.dataset.value;
      this.setData({
        plugins_use_coupon_id: value,
        popup_plugins_coupon_status: false
      });
      this.init();
    },

    // 不使用优惠劵
    plugins_coupon_not_use_event(e) {
      this.setData({
        plugins_use_coupon_id: 0,
        popup_plugins_coupon_status: false
      });
      this.init();
    }

  }
};
</script>
<style>
@import "./buy.css";
</style>