<template>
	<view>
		<!-- 卡片开始 -->
		<view class="cu-card case no-card" >
			<view class="cu-item shadow">
				<swiper class="square-dot image" :indicator-dots="true" :circular="true"
				 :autoplay="autoplay" interval="5000" duration="500">
				 <swiper-item class="swiper-item goods-video-submit" v-if="video">
					 <!-- 播放 or  暂停 -->
					 <view class="cu-video-play" @click="pause()" >
						 <image 
						 :src="video_play ? '/static/images/vpause.png':'/static/images/vplay.png'" mode="aspectFit"></image>
					 </view>
					 <!-- 全屏 -->
					 <view class="cu-video-fullscreen" @click="fullScreen()" >
						 <image src="/static/images/full-screen.png" mode="aspectFit"></image>				 
					 </view>
					 <!-- 视频 -->
					  <video id="myVideo"  :src="video" class="swiper-video" :controls="false"
							:show-center-play-btn="false" ></video>
				 </swiper-item>
					<swiper-item class="swiper-item" v-for="(item,index) in swiperList" :key="index">
						<image :src="item.url" mode="aspectFill"></image>
						<view class="cu-tag bg-black">{{index+1}}/{{swiperList.length}}
						</view>
					</swiper-item>
				</swiper>
				<view class="cu-list menu-avatar">
					<view class="cu-item">
						<view class="cu-avatar round lg" style="background-image:url(https://ossweb-img.qq.com/images/lol/web201310/skin/big10006.jpg);"></view>
						<view class="content flex-sub">
							<view class="text-black">正义天使 凯尔</view>
							<view class="text-gray text-sm flex justify-between">
								十天前
								<view class="text-gray text-sm">
									<text class="cuIcon-attentionfill margin-lr-xs"></text> 10
									<text class="cuIcon-appreciatefill margin-lr-xs"></text> 20
									<text class="cuIcon-messagefill margin-lr-xs"></text> 30
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		
		<!-- 卡片结束 -->
	</view>
</template>

<script>
const app = getApp();
export default {
  data() {
    return {
      form_submit_loading: false,
      autoplay: false,// 视频是否播放
      controls: false,// 全屏下显示下边栏
			towerStart: 0,
			direction: '',
			video:'https://shopxo.bachashu.com/static/upload/video/goods/2019/01/14/1547458876723311.mp4',
			video_play:false,
			swiperList: [
				{
					id: 1,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big37006.jpg',
				}, 
				{
					id: 2,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big39000.jpg'
				}, 
				{
					id: 3,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big10001.jpg'
				}, 
				{
					id: 4,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big25011.jpg'
				}, 
				{
					id: 5,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big21016.jpg'
				}, 
				{
					id: 6,
					type: 'image',
					url: 'https://ossweb-img.qq.com/images/lol/web201310/skin/big99008.jpg'
				},
			],
			
    };
  },

  components: {
		
	},

  onLoad() {
		this.TowerSwiper('swiperList');
	},
	onReady: function (res) {
        this.videoContext = uni.createVideoContext('myVideo')
  },
  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.weitao_index
    });
    // this.init();
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
		//轮播图
		// 初始化towerSwiper
		TowerSwiper(name) {
			let list = this[name];
			for (let i = 0; i < list.length; i++) {
				list[i].zIndex = parseInt(list.length / 2) + 1 - Math.abs(i - parseInt(list.length / 2))
				list[i].mLeft = i - parseInt(list.length / 2)
			}
			this.swiperList = list
		},
		
		// towerSwiper触摸开始
		TowerStart(e) {
			this.towerStart = e.touches[0].pageX
		},
		
		// towerSwiper计算方向
		TowerMove(e) {
			this.direction = e.touches[0].pageX - this.towerStart > 0 ? 'right' : 'left'
		},
		
		// towerSwiper计算滚动
		TowerEnd(e) {
			let direction = this.direction;
			let list = this.swiperList;
			if (direction == 'right') {
				let mLeft = list[0].mLeft;
				let zIndex = list[0].zIndex;
				for (let i = 1; i < this.swiperList.length; i++) {
					this.swiperList[i - 1].mLeft = this.swiperList[i].mLeft
					this.swiperList[i - 1].zIndex = this.swiperList[i].zIndex
				}
				this.swiperList[list.length - 1].mLeft = mLeft;
				this.swiperList[list.length - 1].zIndex = zIndex;
			} else {
				let mLeft = list[list.length - 1].mLeft;
				let zIndex = list[list.length - 1].zIndex;
				for (let i = this.swiperList.length - 1; i > 0; i--) {
					this.swiperList[i].mLeft = this.swiperList[i - 1].mLeft
					this.swiperList[i].zIndex = this.swiperList[i - 1].zIndex
				}
				this.swiperList[0].mLeft = mLeft;
				this.swiperList[0].zIndex = zIndex;
			}
			this.direction = ""
			this.swiperList = this.swiperList
		},
		// 视频播放
		pause() {
			if(this.video_play){
				this.video_play = false
				this.videoContext.pause()
			}else{
				this.video_play = true
				this.videoContext.play()
			}
			
		},
		fullScreen() {
			console.log("requestFullScreen")
			this.videoContext.requestFullScreen()
		},

  },
};
</script>

<style>
	@import "/colorui/main.css";
	@import "/colorui/icon.css";
	@import "/colorui/simple.css";
	.tower-swiper .tower-item {
		transform: scale(calc(0.5 + var(--index) / 10));
		margin-left: calc(var(--left) * 100upx - 150upx);
		z-index: var(--index);
	}
	.swiper-item,swiper {
	    height: 50vh !important;
	    display: block;
	}
	/* 视频 */
	.cu-card.case .image .cu-video-play {
		position: absolute;
		left: 40rpx;
		bottom: 32rpx;
		width: 50rpx;
		height: 60rpx;
		z-index: 99;
	}
	.cu-card.case .image .cu-video-fullscreen {
		position: absolute;
		right: 40rpx;
		bottom: 32rpx;
		width: 50rpx;
		height: 50rpx;
		z-index: 99;
	}
</style>
