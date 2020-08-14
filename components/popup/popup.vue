<template>
<view class="am-popup" v-bind:class="{ 'am-popup-show': propShow, 'animation': propAnimation }"
		:disable-scroll="propDisablescroll || true">
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
  watch: {
	  propShow(val){
		  console.log(val)
	  }
  },
  mounted() {
  	console.log(this.propShow)
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