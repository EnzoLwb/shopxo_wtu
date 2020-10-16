<template>
    <view class="page">
        <video class="video" id="demoVideo" :controls="false" :enable-progress-gesture="false" :show-center-play-btn="disable" src="https://img.cdn.aliyun.dcloud.net.cn/guide/uniapp/%E7%AC%AC1%E8%AE%B2%EF%BC%88uni-app%E4%BA%A7%E5%93%81%E4%BB%8B%E7%BB%8D%EF%BC%89-%20DCloud%E5%AE%98%E6%96%B9%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B@20181126-lite.m4v">
            <cover-view class="controls-title">简单的自定义 controls</cover-view>
            <cover-image class="controls-play img" @click="play" src="/static/images/vplay.png"></cover-image>
            <cover-image class="controls-pause img" @click="pause" src="/static/images/vpause.png"></cover-image>
        </video>
    </view>
</template>
<script>
    export default {
        data() {
            return {}
        },
        mounted() {
            this.videoCtx = uni.createVideoContext('demoVideo')
        },
        methods: {
            play(event) {
                this.videoCtx.play();
                uni.showToast({
                    title: '开始播放',
                    icon: 'none'
                });
            },
            pause(event) {
                this.videoCtx.pause();
                uni.showToast({
                    title: '暂停播放',
                    icon: 'none'
                });
            }
        }
    }
</script>
<style>
    .page {
        display: flex;
        justify-content: center;
    }

    .video {
        position: relative;
    }

    cover-view,
    cover-image {
        display: inline-block;
    }

    .img {
        position: absolute;
        width: 100rpx;
        height: 100rpx;
        top: 50%;
        margin-top: -50rpx;
    }

    .controls-play {
        left: 50rpx;
    }

    .controls-pause {
        right: 50rpx;
    }

    .controls-title {
        width: 100%;
        text-align: center;
        color: #FFFFFF;
    }
</style>