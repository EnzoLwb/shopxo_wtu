<template>
<view>
<view v-if="user != null" class="content">
  <form @submit="formSubmit">
    <input type="number" placeholder="输入手机号码" maxlength="11" name="mobile" @input="bind_key_input" class="mobile"></input>
    <view class="code clearfix">
      <input type="number" placeholder="验证码" maxlength="4" name="verify" class="verify"></input>
      <button type="default" hover-class="none" size="mini" :loading="verify_loading" :disabled="verify_disabled" @tap="verify_send" :class="'verify-sub ' + (verify_disabled ? 'sub-disabled' : '')">{{verify_submit_text}}</button>
    </view>
    <button type="default" formType="submit" hover-class="none" plain :loading="form_submit_loading" :disabled="form_submit_loading" :class="'submit ' + (form_submit_loading ? 'my-btn-gray' : 'my-btn-default')">确认绑定</button>
  </form>
</view>

<view v-if="user == null" class="user-login tc">
  <view class="cr-888 fs-12">确认登录授权，为您提供更优质的服务</view>
  <button type="primary" size="mini" open-type="getUserInfo" @getuserinfo="get_user_info_event">授权登录</button>
</view>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      params: null,
      user: null,
      mobile: null,
      verify_submit_text: '获取验证码',
      verify_loading: false,
      verify_disabled: false,
      form_submit_loading: false,
      verify_time_total: 60,
      temp_clear_time: null
    };
  },

  components: {},
  props: {},

  /**
   * 页面加载初始化
   */
  onLoad(option) {
    // 设置用户信息
    this.setData({
      params: option,
      user: app.globalData.get_user_cache_info() || null
    }); // 标题设置

    wx.setNavigationBarTitle({
      title: this.user == null ? '授权用户信息' : '手机绑定'
    });
  },

  methods: {
    /**
     * 登录授权事件
     */
    get_user_info_event(e) {
      this.user_auth_code(null, null, e.detail);
    },

    /**
     * 用户授权
     * object     回调操作对象
     * method     回调操作对象的函数
     * auth_data  授权数据
     */
    user_auth_code(object, method, auth_data) {
      // 请求授权接口
      var self = this;
      wx.getSetting({
        success(res) {
          if (!res.authSetting['scope.userInfo']) {
            self.setData({
              user: null
            });
          } else {
            app.globalData.user_auth_login(self, 'user_auth_back_event', auth_data);
          }
        },

        fail: e => {
          app.globalData.showToast("授权校验失败");
        }
      });
    },

    /**
     * 授权返回事件
     */
    user_auth_back_event() {
      var user = app.globalData.get_user_cache_info();
      this.setData({
        user: user || null
      });

      if (app.globalData.user_is_need_login(user) == false) {
        wx.navigateBack();
      }
    },

    /**
     * 输入手机号码事件
     */
    bind_key_input(e) {
      this.setData({
        mobile: e.detail.value
      });
    },

    /**
     * 短信验证码发送
     */
    verify_send() {
      // 数据验证
      var validation = [{
        fields: 'mobile',
        msg: '请填写手机号码'
      }];

      if (app.globalData.fields_check(this, validation)) {
        // 网络请求
        var self = this;
        wx.showLoading({
          title: '发送中...'
        });
        this.setData({
          verify_submit_text: '发送中',
          verify_loading: true,
          verify_disabled: true
        });
        wx.request({
          url: app.globalData.get_request_url('regverifysend', 'user'),
          method: 'POST',
          data: {
            mobile: this.mobile
          },
          dataType: 'json',
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            wx.hideLoading();

            if (res.data.code == 0) {
              this.setData({
                verify_loading: false
              });
              var temp_time = this.verify_time_total;
              this.temp_clear_time = setInterval(function () {
                if (temp_time <= 1) {
                  clearInterval(self.temp_clear_time);
                  self.setData({
                    verify_submit_text: '获取验证码',
                    verify_disabled: false
                  });
                } else {
                  temp_time--;
                  self.setData({
                    verify_submit_text: '剩余 ' + temp_time + ' 秒'
                  });
                }
              }, 1000);
            } else {
              this.setData({
                verify_submit_text: '获取验证码',
                verify_loading: false,
                verify_disabled: false
              });
              app.globalData.showToast(res.data.msg);
            }
          },
          fail: () => {
            wx.hideLoading();
            this.setData({
              verify_submit_text: '获取验证码',
              verify_loading: false,
              verify_disabled: false
            });
            app.globalData.showToast("服务器请求出错");
          }
        });
      }
    },

    /**
     * 表单提交
     */
    formSubmit(e) {
      // 邀请人参数
      var params = wx.getStorageSync(app.globalData.data.cache_launch_info_key) || null; // 数据验证

      var validation = [{
        fields: 'mobile',
        msg: '请填写手机号码'
      }, {
        fields: 'verify',
        msg: '请填写验证码'
      }, {
        fields: 'weixin_openid',
        msg: '授权id不能为空'
      }];
      e.detail.value['weixin_openid'] = this.user.weixin_openid;
      e.detail.value['nickname'] = this.user.nickname;
      e.detail.value['avatar'] = this.user.avatar;
      e.detail.value['province'] = this.user.province;
      e.detail.value['city'] = this.user.city;
      e.detail.value['gender'] = this.user.gender;
      e.detail.value['weixin_unionid'] = this.user.weixin_unionid || '';
      e.detail.value['app_type'] = 'weixin';
      e.detail.value['referrer'] = params == null ? this.user.referrer || 0 : params.referrer || 0;

      if (app.globalData.fields_check(e.detail.value, validation)) {
        wx.showLoading({
          title: '处理中...'
        });
        this.setData({
          form_submit_loading: true
        }); // 网络请求

        wx.request({
          url: app.globalData.get_request_url('reg', 'user'),
          method: 'POST',
          data: e.detail.value,
          dataType: 'json',
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            wx.hideLoading();

            if (res.data.code == 0 && (res.data.data || null) != null) {
              clearInterval(this.temp_clear_time);
              app.globalData.showToast(res.data.msg, 'success');
              wx.setStorage({
                key: app.globalData.data.cache_user_info_key,
                data: res.data.data
              });
              var event_callback = this.params.event_callback || null;
              setTimeout(function () {
                // 触发回调函数
                if (event_callback != null) {
                  getCurrentPages()[getCurrentPages().length - 2][event_callback]();
                }

                wx.navigateBack();
              }, 1000);
            } else {
              this.setData({
                form_submit_loading: false
              });
              app.globalData.showToast(res.data.msg);
            }
          },
          fail: () => {
            wx.hideLoading();
            this.setData({
              form_submit_loading: false
            });
            app.globalData.showToast("服务器请求出错");
          }
        });
      }
    }

  }
};
</script>
<style>
@import "./login.css";
</style>