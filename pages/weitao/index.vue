<template>
	<view>
		<!-- 卡片开始 -->
		<view class="cu-card case no-card" v-for="(weitao_card,key) in weitao_data" :key="key">
			<view class="cu-item shadow">
				<swiper style="height:45vh"  class="square-dot image" :indicator-dots="true" :circular="true"
				 :autoplay="autoplay" interval="5000" duration="500">
				 <swiper-item class="swiper-item goods-video-submit" v-if="weitao_card.video">
					 <!-- 播放 or  暂停 -->
					 <view class="cu-video-play" @click="pause(key,weitao_card.video)" >
						 <image :src="weitao_card.video_play ? '/static/images/vpause.png':'/static/images/vplay.png'" mode="aspectFit"></image>
					 </view>
					 <!-- 全屏 -->
					<!-- <view class="cu-video-fullscreen" @click="fullScreen(key,weitao_card.video)" >
						 <image src="/static/images/full-screen.png" mode="aspectFit"></image>				 
					 </view> -->
					 <!-- 视频 -->
					  <video :id="'myVideo'+weitao_card.id"  :src="weitao_card.video" class="swiper-video" :controls="false"
							:show-center-play-btn="false" ></video>
				 </swiper-item>
					<swiper-item class="swiper-item" v-for="(img,index) in weitao_card.swiperList" :key="index" 
					@tap="goods_detail_images_view_event(index,weitao_card.swiperList)">
						<image :src="img" mode="aspectFill" ></image>
						<view class="cu-tag bg-black">{{index+1}}/{{weitao_card.swiperList.length}}
						</view>
					</swiper-item>
				</swiper>
				<!-- 商品轮播开始 -->
				<view class="cu-item margin-top">
					<goodsSwiper :datas="weitao_card" :config="config"  @change="swiperChange" @tapFun="tapFun"></goodsSwiper>
				</view>
				<!-- 商品轮播结束 -->
				<view class="cu-item" style="margin-top: 15rpx;">
					<view class="text-content">
						{{weitao_card.text_content}}
					</view>
				</view>
				<!-- 内容描述结束 -->
				<view class="cu-list menu-avatar">
					<view class="cu-item">
						<view class="cu-avatar round" :style="'background-image:url('+weitao_card.shop.shop_avatar+')'"></view>
						<view class="content flex-sub" style="left: 126rpx;">
							<view class="text-black text-sm">{{weitao_card.shop.shop_name}}</view>
							<view class="text-gray text-sm flex justify-between">
								{{weitao_card.pub_time}}
							</view>
						</view>
						<view class="text-gray text-sm right-feature">
								<button class="cu-btn block line-orange round margin-bottom enter-shop" @tap="sellerGoods(weitao_card.shop.shop_name,weitao_card.seller_id)">
									<text class="cuIcon-shopfill"></text> 进店</button>
								<!-- <text class="cuIcon-attentionfill margin-lr"></text> 10
								<text class="cuIcon-appreciatefill margin-lr"></text> 20 -->
								<!-- <text class="cuIcon-messagefill margin-lr-xs"></text> 30 -->
							</view>
						
					</view>
				</view>
			</view>
		</view>
		
		<!-- 卡片结束 -->
		<view class="load" >{{loadTxt}}</view>
	</view>
</template>

<script>
import goodsSwiper from "@/components/pyh-goodsSwiper/pyh-goodsSwiper.vue"
const app = getApp();
export default {
  data() {
    return {
      form_submit_loading: false,
      autoplay: false,// 视频是否播放
      controls: false,// 全屏下显示下边栏
			towerStart: 0,
			direction: '',
			videoContext:[],
			data_list_loding_msg: '',
			page: 1,
			page_size: 10,
			data_bottom_line_status: false,
			data_list_loding_status: 1,
			config:{
				more:true,//是否显示更多
				autoplay:false,
				shadow:false,//是否显示两边渐变
				multiple:1,
				// hideMargin:true,是否显示右边距
				// current:0,
				// interval:5000,
				// duration:500,//app-nvue不支持
				// circular:true,//如果有边距不推荐使用，效果不佳
			},
			weitao_data:[],
			isGet:true,
			getBottom:false,
			loadTxt:'',
			showLoad:true,
			common_seller:app.globalData.common_seller,
    };
  },

  components: {
		goodsSwiper
	},
  // 下拉刷新
  onPullDownRefresh() {
		this.page = 1
		this.isGet=true,
		this.getBottom=false,
		this.loadTxt='',
		this.showLoad=true,
    this.get_data_list();
  },
  onLoad() {
		// for (let i = 0; i < this.weitao_data.length; i++) {
		// 	this.TowerSwiper(i);
		// }
		
	},
  onShow() {
    wx.setNavigationBarTitle({
      title: app.globalData.common_pages_title.weitao_index
    });
    this.init();
  },
	onReachBottom() {
		this.get_data_list()
	},
  methods: {
    // 初始化
    init() {
      var user = app.globalData.get_user_cache_info(this, "init"),
          self = this; // 用户未绑定用户则转到登录页面
      
      var msg = user == false ? '授权用户信息' : '绑定手机号码';
      
      if (app.globalData.user_is_need_login(user)) {
        wx.showModal({
          title: '温馨提示',
          content: msg,
          confirmText: '确认',
          cancelText: '暂不',
          success: result => {
            wx.stopPullDownRefresh();
      
            if (result.confirm) {
              wx.navigateTo({
                url: "/pages/login/login?event_callback=init"
              });
            }
						
          }
        })
				return false
			}
			this.get_data_list();
    },
		// 获取数据
		get_data_list() {
			if(!this.isGet || this.getBottom){
				return;
			}
			this.isGet=false;
		  var self = this;
		  wx.showLoading({
		    title: "加载中..."
		  });
		  if (self.weitao_data.length <= 0) {
		    self.setData({
		      data_list_loding_status: 1
		    });
		  }
		  wx.request({
		    url: app.globalData.get_request_url("index", "weitao"),
		    method: "POST",
		    data: {
					page:self.page,
				},
		    dataType: "json",
		    success: res => {
		      wx.hideLoading();
		      wx.stopPullDownRefresh();
		      if (res.data.code == 0) {
		        var data = res.data.data;
		        var status = (data.data || []).length > 0;
						
						if(data.data.length == 0){
							console.log('没数据了')
							self.getBottom = true
						}
						if(data.data.length < self.page_size){
							self.loadTxt="没数据了~";
							self.showLoad = false;
						}else if(self.getBottom){
							self.loadTxt="没数据了~";
							self.showLoad = false;
						}else{
							self.loadTxt="上拉/点击加载更多";
							self.page++;
							self.isGet=true;
						}
						//设置视频和店家
						for (let i = 0; i < data.data.length; i++) {
							if(!data.data[i].shop){
								data.data[i].shop = self.common_seller
							}
							if(data.data[i].video!=""){
								self.videoContext[data.data[i].video] = uni.createVideoContext("myVideo"+data.data[i].id)
							}
						}
						self.weitao_data = [...self.weitao_data,...data.data]
		        this.setData({
		          data_list_loding_msg: '',
		          data_list_loding_status: status ? 3 : 0,
		          data_bottom_line_status: status
		        }); 
		      
					     
		      } else {
		        self.setData({
		          data_bottom_line_status: false,
		          data_list_loding_status: 2,
		          data_list_loding_msg: res.data.msg
		        });
		        app.globalData.showToast(res.data.msg);
		      }
		    },
		    fail: () => {
		      wx.hideLoading();
		      wx.stopPullDownRefresh();
		      self.setData({
		        data_bottom_line_status: false,
		        data_list_loding_status: 2,
		        data_list_loding_msg: '服务器请求出错'
		      });
		      app.globalData.showToast("服务器请求出错");
		    }
		  });
		},
		goods_detail_images_view_event(key,urls) {
		  if (key != null) {
		    wx.previewImage({
		      current: key,
		      urls: urls
		    });
		  }
		},
		//轮播图
		// 初始化towerSwiper
		TowerSwiper(key) {
			let list = this.weitao_data[key]['swiperList'];
			for (let i = 0; i < list.length; i++) {
				list[i].zIndex = parseInt(list.length / 2) + 1 - Math.abs(i - parseInt(list.length / 2))
				list[i].mLeft = i - parseInt(list.length / 2)
			}
			this.weitao_data[key]['swiperList'] = list
		},
		
		// 视频播放
		pause(key,video_path) {
			if(this.weitao_data[key].video_play){
				this.weitao_data[key].video_play = false
				this.videoContext[video_path].pause()
			}else{
				this.weitao_data[key].video_play = true
				this.videoContext[video_path].play()
			}
			
		},
		fullScreen(key,video_path) {
			console.log("requestFullScreen")
			this.videoContext[video_path].requestFullScreen()
		},
		//商品轮播
		swiperChange(current){
			console.log(current);
		},
		tapFun(e){
			console.log(e.type,e.id,e.index);
			wx.navigateTo({
				url: '/pages/goods-detail/goods-detail?goods_id=' + e.id
			});
		},
		//跳转到商家页面
		sellerGoods(shop_name,admin_id){
			wx.navigateTo({
				url: '/pages/goods-search/goods-search?shop_name='+shop_name+'&admin_id=' + admin_id
			});
		},
  },
};
</script>

<style>
	@import "/colorui/main.css";
	@import "/colorui/icon.css";
	@import "/colorui/simple.css";
	.cu-avatar {
		width: 74rpx;
		height: 74rpx;
	}
	.tower-swiper .tower-item {
		transform: scale(calc(0.5 + var(--index) / 10));
		margin-left: calc(var(--left) * 100upx - 150upx);
		z-index: var(--index);
	}
	.right-feature{
		display: inline-block;float: left;margin-right:40rpx ;
	}
	.enter-shop{
		width: 140rpx;
		margin: 10rpx 0 20rpx 40rpx;
		font-size:24rpx ;height: 54rpx;
	}
/* 	.swiper-item,swiper {
	    height: 50vh !important;
	    display: block;
	} */
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
	.text-content{
		padding: 0 30rpx 0;
		/* max-height: 6.4em; */
		overflow: hidden;
		font-size: 26rpx;
		/* margin-bottom: 20rpx; */
		color: #607D8B;
	}
	.load{
		line-height: 80upx;
		text-align: center;
		font-size: 26upx;
		color: #999;
		padding-bottom: 20upx;
	}
</style>
