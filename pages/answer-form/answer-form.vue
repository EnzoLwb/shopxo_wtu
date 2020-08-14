<template>
<view>
<form @submit="formSubmit">
    <view class="form-input bg-white spacing-mb">
        <input type="text" class="wh-auto" name="name" placeholder="联系人" maxlength="30"></input>
    </view>
    <view class="form-input bg-white spacing-mb">
        <input type="number" class="wh-auto" name="tel" placeholder="联系电话" maxlength="15"></input>
    </view>
    <view class="form-input bg-white spacing-mb">
        <textarea name="content" class="content-textarea" maxlength="160" placeholder="请详细描述问题，我们将尽快为您解答！"></textarea>
    </view>

    <view class="bottom-btn-box fixed">
        <button type="default" formType="submit" class="my-btn-default" hover-class="none" @tap="submit_event" :loading="form_submit_loading" :disabled="form_submit_loading">提交</button>
    </view>
</form>
</view>
</template>

<script>
const app = getApp();

export default {
  data() {
    return {
      form_submit_loading: false
    };
  },

  components: {},
  props: {},

  onLoad() {},

  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.data.common_pages_title.answer_form
    });
    this.init();
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
      }
    },

    /**
     * 表单提交
     */
    formSubmit(e) {
      // 数据验证
      var validation = [{
        fields: 'name',
        msg: '请填写联系人'
      }, {
        fields: 'tel',
        msg: '请填写联系电话'
      }, {
        fields: 'content',
        msg: '请填写内容'
      }];

      if (app.globalData.fields_check(e.detail.value, validation)) {
        wx.showLoading({
          title: '提交中...'
        });
        this.setData({
          form_submit_loading: true
        }); // 网络请求

        wx.request({
          url: app.globalData.get_request_url('add', 'answer'),
          method: 'POST',
          data: e.detail.value,
          dataType: 'json',
          header: {
            'content-type': 'application/x-www-form-urlencoded'
          },
          success: res => {
            wx.hideLoading();

            if (res.data.code == 0) {
              app.globalData.showToast(res.data.msg, "success");
              setTimeout(function () {
                wx.redirectTo({
                  url: "/pages/user-answer-list/user-answer-list"
                });
              }, 2000);
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
            app.globalData.showToast('服务器请求出错');
          }
        });
      }
    }

  }
};
</script>
<style>
@import "./answer-form.css";
</style>