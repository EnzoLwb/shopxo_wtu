<template>
<view>
<view v-if="data_list.length > 0" class="page">
  <view v-for="(item, index) in data_list" :key="index" :class="'goods-item oh bg-white ' + (common_is_exhibition_mode == 1 ? 'exhibition-mode-data' : '')">
    <!-- 选择 -->
    <view v-if="common_is_exhibition_mode != 1" @tap="selectedt_event" data-type="node" :data-index="index" class="fl selected">
      <image class="icon" :src="'/static/images/default-select' + ((item.is_error || 0) == 1 ? '-disabled' : ((item.selected || false) ? '-active' : '')) + '-icon.png'" mode="widthFix"></image>
    </view>

    <view class="bg-white items">
      <!-- 图片/链接 -->
      <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + item.goods_id">
        <image class="goods-image fl" :src="item.images" mode="aspectFill"></image>
      </navigator>

      <!-- 基础 -->
      <view class="goods-base">
        <view class="goods-title multi-text">{{item.title}}
        </view>
        <block v-if="item.spec != null">
          <view v-for="(spec, index2) in item.spec" :key="index2" class="goods-attribute cr-888">{{spec.type}}:{{spec.value}}</view>
        </block>
      </view>
      <!-- 数量 -->
      <view class="number-content tc oh">
        <view @tap="goods_buy_number_event" class="number-submit tc cr-888 fl" :data-index="index" data-type="0">-</view>
        <input @blur="goods_buy_number_blur" class="tc cr-888 fl" type="number" :value="item.stock" :data-index="index"></input>
        <view @tap="goods_buy_number_event" class="number-submit tc cr-888 fl" :data-index="index" data-type="1">+</view>
      </view>

      <!-- 价格 -->
      <view class="oh goods-price">
        <text class="sales-price">￥{{item.price}}</text>
        <text v-if="item.original_price > 0" class="original-price">￥{{item.original_price}}</text>
        <text class="buy-number cr-888">x{{item.stock}}</text>

        <!-- 错误 -->
        <text v-if="(item.is_error || 0) == 1" class="error-msg">{{item.error_msg}}</text>

        <!-- 移除 -->
        <view class="fr remove" :data-id="item.id" :data-goodsid="item.goods_id" :data-index="index" @tap="cart_remove_event">移除</view>
      </view>
    </view>
  </view>

  <!-- 操作导航 -->
  <view v-if="data_list.length > 0" class="buy-nav oh wh-auto br-t">
    <!-- 展示型 -->
    <block v-if="common_is_exhibition_mode == 1">
      <view class="exhibition-mode">
        <button class="bg-main wh-auto" type="default" @tap="exhibition_submit_event" hover-class="none">{{common_is_exhibition_mode_btn_text}}</button>
      </view>
    </block>

    <!-- 销售型 -->
    <block v-else>
      <view class="nav-base bg-white fl single-text">
        <view @tap="selectedt_event" data-type="all" class="fl selected">
          <image class="icon" :src="'/static/images/default-select' + (is_selected_all ? '-active' : '') + '-icon.png'" mode="widthFix"></image>
          <text>全选</text>
        </view>
        <view class="fr price">
          <view class="sales-price single-text fr">￥{{total_price}}</view>
          <view class="fr">合计：</view>
        </view>
      </view>
      <view class="fr nav-submit">
        <button class="bg-main wh-auto" type="default" @tap="buy_submit_event" :disabled="buy_submit_disabled_status" hover-class="none">结算</button>
      </view>
    </block>
  </view>
</view>

<!-- 空购物车 -->
<view v-if="data_list.length == 0 && data_list_loding_status == 0" class="no-data-box tc">
  <image src="/static/images/default-cart-empty.png" mode="widthFix"></image>
  <view class="no-data-tips">{{data_list_loding_msg || '购物车空空如也'}}</view>
  <navigator url="/pages/index/index" open-type="switchTab" hover-class="none">
    <button type="default" class="my-btn-default" hover-class="none">去逛逛</button>
  </navigator>
</view>

<view v-if="data_list.length == 0 && data_list_loding_status != 0">
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
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      data_list_loding_status: 1,
      data_list_loding_msg: '',
      data_bottom_line_status: false,
      data_list: [],
      swipe_index: null,
      total_price: '0.00',
      is_selected_all: false,
      buy_submit_disabled_status: true,
      // 是否展示型
      common_is_exhibition_mode: 0,
      common_is_exhibition_mode_btn_text: null,
      customer_service_tel: null
    };
  },

  components: {},
  props: {},

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.cart
    });
    this.init();
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.init();
  },

  methods: {
    init(e) {
      var user = app.globalData.get_user_cache_info(this, "init"); // 用户未绑定用户则转到登录页面

      var msg = user == false ? '授权用户信息' : '绑定手机号码';

      if (app.globalData.user_is_need_login(user)) {
        wx.showModal({
          title: '温馨提示',
          content: msg,
          confirmText: '确认',
          cancelText: '暂不',
          success: result => {
            if (result.confirm) {
              wx.navigateTo({
                url: "/pages/login/login?event_callback=init"
              });
            } else {
              this.setData({
                data_list_loding_status: 0,
                data_bottom_line_status: false,
                data_list_loding_msg: '请先' + msg
              });
            }
          }
        });
      } else {
        this.get_data();
      }
    },

    // 获取数据
    get_data() {
      this.setData({
        data_list_loding_status: 1,
        total_price: '0.00',
        is_selected_all: false,
        buy_submit_disabled_status: true
      });
      wx.request({
        url: app.globalData.get_request_url("index", "cart"),
        method: "POST",
        data: {},
        dataType: "json",
        success: res => {
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            var data = res.data.data; // 数据赋值

            this.setData({
              data_list: data.data,
              data_list_loding_status: data.data.length == 0 ? 0 : 3,
              data_bottom_line_status: true,
              data_list_loding_msg: '购物车空空如也',
              // 是否展示型
              common_is_exhibition_mode: data.common_is_exhibition_mode || 0,
              common_is_exhibition_mode_btn_text: data.common_is_exhibition_mode_btn_text || '立即咨询',
              customer_service_tel: data.customer_service_tel || null
            });
          } else {
            this.setData({
              data_list_loding_status: 2,
              data_bottom_line_status: false,
              data_list_loding_msg: res.data.msg
            });
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          wx.stopPullDownRefresh();
          this.setData({
            data_list_loding_status: 2,
            data_bottom_line_status: false,
            data_list_loding_msg: '服务器请求出错'
          });
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 数量输入事件
    goods_buy_number_blur(e) {
      var index = e.currentTarget.dataset.index || 0;
      var buy_number = parseInt(e.detail.value) || 1;
      this.goods_buy_number_func(index, buy_number);
    },

    // 数量操作事件
    goods_buy_number_event(e) {
      var index = e.currentTarget.dataset.index || 0;
      var type = parseInt(e.currentTarget.dataset.type) || 0;
      var temp_buy_number = parseInt(this.data_list[index]['stock']);

      if (type == 0) {
        var buy_number = temp_buy_number - 1;
      } else {
        var buy_number = temp_buy_number + 1;
      }

      this.goods_buy_number_func(index, buy_number);
    },

    // 数量处理方法
    goods_buy_number_func(index, buy_number) {
      var temp_data_list = this.data_list;
      var buy_min_number = parseInt(temp_data_list[index]['buy_min_number']) || 1;
      var buy_max_number = parseInt(temp_data_list[index]['buy_max_number']) || 0;
      var inventory = parseInt(temp_data_list[index]['inventory']);
      var inventory_unit = temp_data_list[index]['inventory_unit'];

      if (buy_number < buy_min_number) {
        buy_number = buy_min_number;

        if (buy_min_number > 1) {
          app.globalData.showToast('起购' + buy_min_number + inventory_unit);
          return false;
        }
      }

      if (buy_max_number > 0 && buy_number > buy_max_number) {
        buy_number = buy_max_number;
        app.globalData.showToast('限购' + buy_max_number + inventory_unit);
        return false;
      }

      if (buy_number > inventory) {
        buy_number = inventory;
        app.globalData.showToast('库存数量' + inventory + inventory_unit);
        return false;
      }

      if (temp_data_list[index]['stock'] == 1 && buy_number == 1) {
        return false;
      } // 更新数据库


      wx.request({
        url: app.globalData.get_request_url("stock", "cart"),
        method: "POST",
        data: {
          "id": temp_data_list[index]['id'],
          "goods_id": temp_data_list[index]['goods_id'],
          "stock": buy_number
        },
        dataType: "json",
        success: res => {
          wx.stopPullDownRefresh();

          if (res.data.code == 0) {
            temp_data_list[index]['stock'] = buy_number;
            this.setData({
              data_list: temp_data_list
            }); // 选择处理

            this.selected_calculate();
          } else {
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 收藏+删除
    goods_favor_delete(id, goods_id, type) {
      wx.request({
        url: app.globalData.get_request_url('favor', 'goods'),
        method: 'POST',
        data: {
          "id": goods_id,
          "is_mandatory_favor": 1
        },
        dataType: 'json',
        success: res => {
          if (res.data.code == 0) {
            this.cart_delete(id, type);
          } else {
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 移除操作事件
    cart_remove_event(e) {
      var id = e.currentTarget.dataset.id || null;
      var index = e.currentTarget.dataset.index || 0;
      var goods_id = e.currentTarget.dataset.goodsid || 0;
      var self = this;

      if (id !== null) {
        self.setData({
          swipe_index: index
        });
        wx.showActionSheet({
          itemList: ['加入收藏', '删除'],

          success(res) {
            if (res.tapIndex == 0) {
              self.goods_favor_delete(id, goods_id, 'favor');
            } else {
              self.cart_delete(id, 'delete');
            }
          }

        });
      } else {
        app.globalData.showToast("参数有误");
      }
    },

    // 购物车删除
    cart_delete(id, type) {
      wx.request({
        url: app.globalData.get_request_url('delete', 'cart'),
        method: 'POST',
        data: {
          "id": id
        },
        dataType: 'json',
        success: res => {
          if (res.data.code == 0) {
            var temp_data_list = this.data_list;
            temp_data_list.splice(this.swipe_index, 1);
            this.setData({
              data_list: temp_data_list,
              swipe_index: null,
              data_list_loding_status: temp_data_list.length == 0 ? 0 : this.data_list_loding_status
            });
            app.globalData.showToast(type == 'delete' ? '删除成功' : '收藏成功', 'success');
          } else {
            app.globalData.showToast(type == 'delete' ? '删除失败' : '收藏失败');
          }
        },
        fail: () => {
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    // 选中处理
    selectedt_event(e) {
      var type = e.currentTarget.dataset.type || null;

      if (type != null) {
        var temp_data_list = this.data_list;
        var temp_is_selected_all = this.is_selected_all;

        switch (type) {
          // 批量操作
          case 'all':
            temp_is_selected_all = temp_is_selected_all == true ? false : true;

            for (var i in temp_data_list) {
              if (temp_data_list[i]['is_error'] != 1) {
                temp_data_list[i]['selected'] = temp_is_selected_all;
              }
            }

            break;
          // 节点操作

          case 'node':
            var index = e.currentTarget.dataset.index || 0;

            if (temp_data_list[index]['is_error'] != 1) {
              temp_data_list[index]['selected'] = temp_data_list[index]['selected'] == true ? false : true;
            }

            break;
        }

        this.setData({
          data_list: temp_data_list,
          is_selected_all: temp_is_selected_all
        }); // 选择处理

        this.selected_calculate();
      }
    },

    // 选中计算
    selected_calculate() {
      var total_price = 0;
      var data_count = 0;
      var selected_count = 0;
      var temp_data_list = this.data_list;

      for (var i in temp_data_list) {
        if ((temp_data_list[i]['is_error'] || 0) == 0) {
          data_count++;
        }

        if ((temp_data_list[i]['selected'] || false) == true) {
          total_price += temp_data_list[i]['stock'] * temp_data_list[i]['price'];
          selected_count++;
        }
      }

      this.setData({
        total_price: total_price.toFixed(2),
        buy_submit_disabled_status: selected_count <= 0,
        is_selected_all: selected_count >= data_count
      });
    },

    // 结算
    buy_submit_event(e) {
      var selected_count = 0;
      var ids = [];
      var temp_data_list = this.data_list;

      for (var i in temp_data_list) {
        if ((temp_data_list[i]['selected'] || false) == true) {
          ids.push(temp_data_list[i]['id']);
          selected_count++;
        }
      }

      if (selected_count <= 0) {
        app.globalData.showToast("请选择商品");
        return false;
      } // 进入订单确认页面


      var data = {
        "buy_type": "cart",
        "ids": ids.join(',')
      };
      wx.navigateTo({
        url: '/pages/buy/buy?data=' + JSON.stringify(data)
      });
    },

    // 展示型事件
    exhibition_submit_event(e) {
      app.globalData.call_tel(this.customer_service_tel);
    }

  }
};
</script>
<style>
@import "./cart.css";
</style>