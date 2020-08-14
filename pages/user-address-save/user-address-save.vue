<template>
<view>
<form @submit="form_submit">
    <view class="person-box bg-white">
        <view class="addressee br-b oh">
            <input class="addressee-name fl br-r" name="name" type="text" :value="name" placeholder="姓名"></input>
            <input class="addressee-phone fl" name="tel" type="number" :value="tel" placeholder="手机号"></input>
        </view>

        <!-- 地区选择 -->
        <view class="select-address br-b oh">
            <view class="section fl br-r">
                <picker name="province" @change="select_province" :value="province_value" :range="province_list" range-key="name">
                    <view :class="'name ' + ((province_value == null) ? 'cr-888' : '' )">{{province_list[province_value].name || default_province}}</view>
                </picker>
            </view>
            <view class="section fl br-r">
                <picker name="city" @change="select_city" :value="city_value" :range="city_list" range-key="name">
                    <view :class="'name ' + ((city_value == null) ? 'cr-888' : '' )">{{city_list[city_value].name || default_city}}</view>
                </picker>
            </view>
            <view class="section fl">
                <picker name="county" @change="select_county" :value="county_value" :range="county_list" range-key="name">
                    <view :class="'name ' + ((county_value == null) ? 'cr-888' : '' )">{{county_list[county_value].name || default_county}}</view>
                </picker>
            </view>
        </view>
        <!-- end地区选择 -->

        <input name="address" class="addressee-address" type="text" :value="address" placeholder="详细地址"></input>
    </view>

    <button class="submit-fixed submit-bottom" type="default" formType="submit" hover-class="none">保存</button>
</form>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      province_list: [],
      city_list: [],
      county_list: [],
      province_id: null,
      city_id: null,
      county_id: null,
      is_default: 0,
      default_province: "请选择省",
      default_city: "请选择市",
      default_county: "请选择区/县",
      province_value: null,
      city_value: null,
      county_value: null,
      params: null,
      name: "",
      tel: "",
      address: ""
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
    if ((this.params.id || null) == null) {
      var title = app.globalData.common_pages_title.user_address_save_add;
    } else {
      var title = app.globalData.common_pages_title.user_address_save_edit;
    }

    wx.setNavigationBarTitle({
      title: title
    });
    this.init();
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
        // 获取地址数据
        if ((this.params.id || null) != null) {
          this.get_user_address();
        } // 获取省


        this.get_province_list();
      }
    },

    //   获取用户地址
    get_user_address() {
      var self = this; // 加载loding

      wx.showLoading({
        title: "加载中..."
      });
      wx.request({
        url: app.globalData.get_request_url("detail", "useraddress"),
        method: "POST",
        data: self.params,
        dataType: "json",
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        success: res => {
          wx.hideLoading();

          if (res.data.code == 0) {
            var data = res.data.data;
            self.setData({
              name: data.name,
              tel: data.tel,
              address: data.address,
              province_id: data.province,
              city_id: data.city,
              county_id: data.county,
              is_default: data.is_default || 0
            });
            self.get_city_list();
            self.get_county_list();
            setTimeout(function () {
              self.init_value();
            }, 500);
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

    // 获取选择的省市区
    get_province_list() {
      var self = this;
      wx.request({
        url: app.globalData.get_request_url("index", "region"),
        method: "POST",
        data: {},
        dataType: "json",
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        success: res => {
          if (res.data.code == 0) {
            var data = res.data.data;
            self.setData({
              province_list: data
            });
          } else {
            app.globalData.showToast(res.data.msg);
          }
        },
        fail: () => {
          app.globalData.showToast("服务器请求出错");
        }
      });
    },

    get_city_list() {
      var self = this;

      if (self.province_id) {
        wx.request({
          url: app.globalData.get_request_url("index", "region"),
          method: "POST",
          data: {
            pid: self.province_id
          },
          dataType: "json",
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            if (res.data.code == 0) {
              var data = res.data.data;
              self.setData({
                city_list: data
              });
            } else {
              app.globalData.showToast(res.data.msg);
            }
          },
          fail: () => {
            app.globalData.showToast("服务器请求出错");
          }
        });
      }
    },

    get_county_list() {
      var self = this;

      if (self.city_id) {
        // 加载loding
        wx.request({
          url: app.globalData.get_request_url("index", "region"),
          method: "POST",
          data: {
            pid: self.city_id
          },
          dataType: "json",
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            if (res.data.code == 0) {
              var data = res.data.data;
              self.setData({
                county_list: data
              });
            } else {
              app.globalData.showToast(res.data.msg);
            }
          },
          fail: () => {
            app.globalData.showToast("服务器请求出错");
          }
        });
      }
    },

    select_province(e) {
      if (e.detail.value >= 0) {
        var value = e.detail.value,
            data = this.province_list[value];
        this.setData({
          province_value: value,
          province_id: data.id,
          city_value: null,
          county_value: null,
          city_id: null,
          county_id: null
        });
        this.get_city_list();
      }
    },

    select_city(e) {
      if (e.detail.value >= 0) {
        var value = e.detail.value,
            data = this.city_list[value];
        this.setData({
          city_value: value,
          city_id: data.id,
          county_value: null,
          county_id: null
        });
        this.get_county_list();
      }
    },

    select_county(e) {
      if (e.detail.value >= 0) {
        var value = e.detail.value,
            data = this.county_list[value];
        this.setData({
          county_value: value,
          county_id: data.id
        });
      }
    },

    init_value() {
      var province_value = this.get_init_value("province_list", "province_id"),
          city_value = this.get_init_value("city_list", "city_id"),
          county_value = this.get_init_value("county_list", "county_id");
      this.setData({
        province_value: province_value,
        city_value: city_value,
        county_value: county_value
      });
    },

    get_init_value(list, id) {
      var data = this[list],
          data_id = this[id],
          value;
      data.forEach((d, i) => {
        if (d.id == data_id) {
          value = i;
          return false;
        }
      });
      return value;
    },

    form_submit(e) {
      var self = this,
          data = self; // 表单数据

      var form_data = e.detail.value; // 数据校验

      var validation = [{
        fields: "name",
        msg: "请填写姓名"
      }, {
        fields: "tel",
        msg: "请填写手机号"
      }, {
        fields: "province",
        msg: "请选择省份"
      }, {
        fields: "city",
        msg: "请选择城市"
      }, {
        fields: "county",
        msg: "请选择区县"
      }, {
        fields: "address",
        msg: "请填写详细地址"
      }];
      form_data["province"] = data.province_id;
      form_data["city"] = data.city_id;
      form_data["county"] = data.county_id;
      form_data["id"] = self.params.id || 0;
      form_data["is_default"] = self.is_default || 0;

      if (app.globalData.fields_check(form_data, validation)) {
        // 加载loding
        wx.showLoading({
          title: "处理中..."
        });
        wx.request({
          url: app.globalData.get_request_url("save", "useraddress"),
          method: "POST",
          data: form_data,
          dataType: "json",
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            wx.hideLoading();

            if (res.data.code == 0) {
              app.globalData.showToast(res.data.msg, "success");
              setTimeout(function () {
                wx.navigateBack();
              }, 1000);
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

  }
};
</script>
<style>
@import "./user-address-save.css";
</style>