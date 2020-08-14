<template>
<view v-if="propNumber > 0" class="am-badge">
  <view :class="'am-badge-text ' + ((propNumber > 99) ? 'am-badge-text-max' : '')">
    <text>{{(propNumber > 99) ? '99+' : propNumber}}</text>
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
    propNumber: Number
  },
  methods: {}
};
</script>
<style>
@import "./badge.css";
</style>