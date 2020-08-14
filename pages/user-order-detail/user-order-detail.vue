<template>
<view>
<view v-if="detail != null">
  <view class="address bg-white spacing-mb">
      <view class="address-base oh">
          <text>{{detail.receive_name}}</text>
          <text class="fr">{{detail.receive_tel}}</text>
      </view>
      <view class="address-detail oh">
          <image class="icon fl" src="/static/images/user-address.png" mode="widthFix"></image>
          <view class="text fr">{{detail.receive_province_name}}{{detail.receive_city_name}}{{detail.receive_county_name}}{{detail.receive_address}}</view>
      </view>
  </view>

  <view class="goods bg-white spacing-mb">
      <view v-for="(item, index) in detail.items" :key="index" class="goods-item oh">
        <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + item.goods_id" hover-class="none">
          <image class="goods-image fl" :src="item.images" mode="aspectFill"></image>
          <view class="goods-base">
            <view class="goods-title multi-text">{{item.title}}</view>
            <block v-if="item.spec != null">
              <view v-for="(spec, index2) in item.spec" :key="index2" class="goods-attribute cr-888">
                {{spec.type}}:{{spec.value}}
              </view>
            </block>
          </view>
          <view class="oh goods-price">
            <text class="sales-price">￥{{item.price}}</text>
            <text v-if="item.original_price > 0" class="original-price">￥{{item.original_price}}</text>
            <text class="buy-number">x{{item.buy_number}}</text>
          </view>
        </navigator>
      </view>
  </view>

  <!-- 订单基础数据 -->
  <view v-if="detail_list.length > 0" class="detail-list bg-white">
    <view v-for="(item, index) in detail_list" :key="index" class="item br-b oh">
      <view class="title fl">{{item.name}}</view>
      <view class="content cr-888 fl br-l">{{item.value}}</view>
    </view>
  </view>

  <!-- 扩展数据 -->
  <view v-if="extension_data.length > 0" class="extension-list bg-white  spacing-mt">
    <view v-for="(item, index) in extension_data" :key="index" class="item br-b oh">
      <text class="title">{{item.name}}</text>
      <text class="content cr-888 br-l">{{item.tips}}</text>
    </view>
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

<view v-if="detail == null">
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

<view class="nav-back tc wh-auto">
  <navigator url="/pages/user-order/user-order" open-type="navigateBack" hover-class="none">
    <button type="default" size="mini" class="cr-888 br" hover-class="none">返回</button>
  </navigator>
</view>
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
      detail: null,
      detail_list: [],
      extension_data: []
    };
  },

  components: {},
  props: {},

  onLoad(params) {
    this.setData({
      params: params
    });
    this.init();
  },

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.data.common_pages_title.user_order_detail
    });
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.init();
  },

  methods: {
    init() {
      var self = this;
      wx.showLoading({
        title: "加载中..."
      });
      this.setData({
        data_list_loding_status: 1
      });
      wx.request({
        url: app.globalData.get_request_url("detail", "order"),
        method: "POST",
        data: {
          id: this.params.id
        },
        dataType: "json",
        success: res => {
          wx.hideLoading();
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            var data = res.data.data;
            self.setData({
              detail: data,
              detail_list: [{
                name: "订单号",
                value: data.order_no || ''
              }, {
                name: "状态",
                value: data.status_name || ''
              }, {
                name: "支付状态",
                value: data.pay_status_name || ''
              }, {
                name: "单价",
                value: data.price || ''
              }, {
                name: "总价",
                value: data.total_price || ''
              }, {
                name: "优惠金额",
                value: data.preferential_price || ''
              }, {
                name: "增加金额",
                value: data.increase_price || ''
              }, {
                name: "支付金额",
                value: data.pay_price || ''
              }, {
                name: "支付方式",
                value: data.payment_name || ''
              }, {
                name: "快递公司",
                value: data.express_name || ''
              }, {
                name: "快递单号",
                value: data.express_number || ''
              }, {
                name: "用户留言",
                value: data.user_note || ''
              }, {
                name: "创建时间",
                value: data.add_time || ''
              }, {
                name: "确认时间",
                value: data.confirm_time || ''
              }, {
                name: "支付时间",
                value: data.pay_time || ''
              }, {
                name: "发货时间",
                value: data.delivery_time || ''
              }, {
                name: "收货时间",
                value: data.collect_time || ''
              }, {
                name: "取消时间",
                value: data.close_time || ''
              }, {
                name: "关闭时间",
                value: data.close_time || ''
              }],
              extension_data: data.extension_data || [],
              data_list_loding_status: 3,
              data_bottom_line_status: true,
              data_list_loding_msg: ''
            });
          } else {
            self.setData({
              data_list_loding_status: 2,
              data_bottom_line_status: false,
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
            data_bottom_line_status: false,
            data_list_loding_msg: '服务器请求出错'
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    }

  }
};
</script>
<style>
@import "./user-order-detail.css";
</style>