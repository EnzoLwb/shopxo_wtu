<template>
<view :class="'am-popup ' + propClassname || '' + ' ' + ((propShow || false) ? 'am-popup-show' : '') + ' ' + ( (propAnimation || true) ? 'animation': '' )" :disable-scroll="propDisablescroll || true">
  <view class="am-popup-mask" v-if="propMask || true" @tap="onMaskTap"></view>
  <view :class="'am-popup-content am-popup-' + propPosition || 'bottom'">
    <slot></slot>
  </view>
</view>
</template>

<script>

export default {
  data() {
    return {};
  },

  components: {},
  props: {
    propClassname: String,
    propShow: Boolean,
    propPosition: String,
    propMask: Boolean,
    propAnimation: Boolean,
    propDisablescroll: Boolean
  },
  methods: {
    onMaskTap: function onMaskTap() {
      this.$emit('onclose', {
        detail: {}
      }, {});
    }
  }
};
</script>
<style>
@import "./popup.css";
</style>