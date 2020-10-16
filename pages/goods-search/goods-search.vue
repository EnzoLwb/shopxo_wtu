<template>
<view>
<!-- 排序 -->
<view class="nav-sort oh">
  <view class="nav-sort-content">
    <block v-for="(item, index) in search_nav_sort_list" :key="index">
      <view class="item tc fl" :data-index="index" @tap="nav_sort_event">
        <text class="cr-666">{{item.name}}</text>
        <image v-if="(item.icon || null) != null" class="icon" :src="'/static/images/search-' + item.icon + '-icon.png'" mode="aspectFill"></image>
      </view>
    </block>
  </view>
  <image class="screening-submit" src="/static/images/search-submit-icon.png" mode="aspectFill" @tap="popup_form_event_show"></image>
</view>

<!-- 列表 -->
<scroll-view scroll-y="true" class="scroll-box" @scrolltolower="scroll_lower" lower-threshold="30">
  <view class="data-list">
    <view v-for="(item, index) in data_list" :key="index" class="items bg-white">
      <navigator :url="'/pages/goods-detail/goods-detail?goods_id=' + item.id" hover-class="none">
        <image :src="item.images" mode="aspectFit"></image>
        <view class="base">
          <view class="single-text">{{item.title}}</view>
          <view class="price">
            <text class="sales-price">￥{{item.min_price}}</text>
          </view>
        </view>
      </navigator>
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

<!-- 筛选条件 popup -->
<component-popup :prop-show="is_show_popup_form" prop-position="right" @onclose="popup_form_event_close">
  <form @submit="form_submit_event" class="popup-form oh bg-white">
    <view class="item oh screening-price">
      <view class="title cr-666">价格区间(元)
      </view>
      <input type="digit" placeholder="最低价" name="min_price" class="br fl"></input>
      <view class="cr-888 fl tc separator">~
      </view>
      <input type="digit" placeholder="最高价" name="max_price" class="br fl"></input>
    </view>
    <view class="item keywords">
      <view class="title cr-666">关键字</view>
      <input type="text" placeholder="关键字" name="keywords" :value="(params.keywords || '')" class="br"></input>
    </view>
    <button formType="submit" class="bg-main form-submit wh-auto" :disabled="popup_form_loading_status" hover-class="none">确认</button>
  </form>
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
      data_bottom_line_status: false,
      data_list: [],
      data_page_total: 0,
      data_page: 1,
      params: null,
      post_data: {},
      is_show_popup_form: false,
      popup_form_loading_status: false,
      search_nav_sort_list: [{
        name: "综合",
        field: "default",
        sort: "asc",
        "icon": null
      }, {
        name: "销量",
        field: "sales_count",
        sort: "asc",
        "icon": "default"
      }, {
        name: "价格",
        field: "min_price",
        sort: "asc",
        "icon": "default"
      }],
      data_total: ""
    };
  },

  components: {
    componentPopup
  },
  props: {},

  onLoad(params) {
    this.setData({
      params: params,
      post_data: params
    });
    this.init();
  },

  onShow() {
		if (this.post_data['shop_name']){
			//设置title
			wx.setNavigationBarTitle({
				title: this.post_data['shop_name']
			});
		}else{
			wx.setNavigationBarTitle({
				title: app.globalData.common_pages_title.goods_search
			});
		}
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.setData({
      data_page: 1
    });
    this.get_data_list(1);
  },

  methods: {
    // 初始化
    init() {
      // 获取数据
      this.get_data_list();
    },

    // 搜索
    search_event() {
      this.setData({
        data_list: [],
        data_page: 1
      });
      this.get_data_list(1);
    },

    // 获取数据列表
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

      var params = this.params;
      var post_data = this.post_data;
      post_data['page'] = this.data_page;
      post_data['category_id'] = params['category_id'] || 0; // 获取数据

      wx.request({
        url: app.globalData.get_request_url("index", "search"),
        method: "POST",
        data: post_data,
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

              if (this.data_page <= 1) {
                this.setData({
                  data_list: [],
                  data_bottom_line_status: false
                });
              }
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

    // 搜索条件
    form_submit_event(e) {
      this.setData({
        post_data: e.detail.value,
        data_page: 1
      });
      this.popup_form_event_close();
      this.get_data_list(1);
    },

    // 筛选条件关闭
    popup_form_event_close(e) {
      this.setData({
        is_show_popup_form: false
      });
    },

    // 筛选条件开启
    popup_form_event_show(e) {
      this.setData({
        is_show_popup_form: true
      });
    },

    // 筛选
    nav_sort_event(e) {
      var index = e.currentTarget.dataset.index || 0;
      var temp_post_data = this.post_data;
      var temp_search_nav_sort = this.search_nav_sort_list;
      var temp_sort = temp_search_nav_sort[index]['sort'] == 'desc' ? 'asc' : 'desc';

      for (var i in temp_search_nav_sort) {
        if (i != index) {
          if (temp_search_nav_sort[i]['icon'] != null) {
            temp_search_nav_sort[i]['icon'] = 'default';
          }

          temp_search_nav_sort[i]['sort'] = 'desc';
        }
      }

      temp_search_nav_sort[index]['sort'] = temp_sort;

      if (temp_search_nav_sort[index]['icon'] != null) {
        temp_search_nav_sort[index]['icon'] = temp_sort;
      }

      temp_post_data['order_by_field'] = temp_search_nav_sort[index]['field'];
      temp_post_data['order_by_type'] = temp_sort;
      this.setData({
        post_data: temp_post_data,
        search_nav_sort_list: temp_search_nav_sort,
        data_page: 1
      });
      this.get_data_list(1);
    }

  }
};
</script>
<style>
@import "./goods-search.css";
</style>