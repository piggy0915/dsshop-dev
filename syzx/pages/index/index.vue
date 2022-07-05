<template>
	<view class="container">
		<!-- 引导添加小程序 -->
		<!-- #ifdef MP-WEIXIN -->
		<view class="guidance-my" v-if="guidanceMy">
			<view class="triangle-top"></view>
			<view @click="popupBoot()" class="bg-black padding-sm margin-top flex">
				<view><span @click.stop="setGuidanceMy" class="icon cuIcon-close text-gray"></span></view>
				<view class="flex-twice text-center">添加到我的小程序，<span class="text-bold">红包、优惠券不错过</span></view>
				<view><span class="icon cuIcon-right text-gray"></span></view>
			</view>
		</view>
		<view class="cu-modal" :class="modalName=='guidanceMy'?'show':''">
			<view class="guidance-modal">
				<view class="triangle-top"></view>
				<view class="title bg-red text-xl padding">点<image
						style="height: 60upx;position: relative;top:0;margin: 0 auto;" mode="heightFix"
						src="../../static/guidance-white.png"></image>添加小程序</view>
				<view class="list">
					<view class="padding text-left min-title">
						<span class="text-red">1、</span>
						点右上
						<image style="height: 60upx;position: relative;top:0;" mode="heightFix"
							src="../../static/guidance.png"></image>
						添加到“我的小程序”
					</view>
					<view>
						<image src="../../static/guidance-1.png" style="height: 100upx;" mode="aspectFit"></image>
					</view>
					<view class="padding text-left min-title">
						<span class="text-red">2、</span>
						回到微信首页，向下拉动
					</view>
					<view>
						<image src="../../static/guidance-2.png" style="height: 140upx;" mode="aspectFit"></image>
					</view>
					<view class="padding text-left min-title">
						<span class="text-red">3、</span>
						从“我的小程序”中，进入
					</view>
					<view>
						<image src="../../static/guidance-3.png" style="height: 140upx;" mode="aspectFit"></image>
					</view>
				</view>
				<view @tap="modalName = null" class="guidance-modal-close">
					<view class="cuIcon-roundclose"></view>
				</view>
			</view>

		</view>
		<official-account v-if="!wechat"></official-account>
		<!-- #endif -->
		<!-- 头部轮播 -->
		<view class="carousel-section">
			<!-- 标题栏和状态栏占位符 -->
			<view class="titleNview-placing"></view>
			<!-- 背景色区域 -->
			<view class="titleNview-background"></view>
			<swiper class="carousel" circular @change="swiperChange">
				<swiper-item v-for="(item, index) in carouselList" :key="index" class="carousel-item"
					@click="navToWwiperPage({item})">
					<image :src="item.resources.img" />
				</swiper-item>
			</swiper>
			<!-- 自定义swiper指示器 -->
			<view class="swiper-dots">
				<text class="num">{{swiperCurrent+1}}</text>
				<text class="sign">/</text>
				<text class="num">{{swiperLength}}</text>
			</view>
		</view>
		<!-- 分类 -->
		<view class="cate-section">
			<!--<view v-for="item in ctegory" :key="item.id" class="cate-item"
				@click="navTo('/pages/product/list?fid='+item.category.pid+'&sid='+item.pid+'&tid='+item.id)">
				<image v-if="item.resources" :src="item.resources.img | smallImage(80)" lazy-load
					style="padding:20rpx;"></image>
				<text>{{item.name}}</text>
			</view>-->
			<view v-for="(item, index) in ctegory" :key="index" class="cate-item"
				@click="item.url=='' ? navTo('/pages/product/list?tid=' + item.id) : navTo(item.url)">
				<image webp="true" :src="item.resources.img | smallImage(150) | webp" lazy-load></image>
				<text>{{item.name}} </text>
			</view>

		</view>

		<view class="ad-1">
			<!--<image v-if="adData.resources" :src="adData.resources.img" mode="scaleToFill" lazy-load  @click="navTo(adData.url)"></image>-->
			<image v-if="adData.resources" :src="adData.resources.img" mode="scaleToFill" lazy-load
				@click="navTo(adData.url)"></image>
		</view>
		<!-- 秒杀 -->
		<view class="seckill-section m-t" v-if="isSeckill && seckill.length">
			<navigator url="../seckill/list" hover-class="none" class="s-header">
				<image class="s-img" src="/static/temp/secskill-img.jpg" mode="widthFix"></image>
				<text class="tip">{{seckillActiveTime}}点场</text>
				<count-down-time :time="seckillTime" @end="endSeckillTime()"></count-down-time>
				<text class="yticon icon-you"></text>
			</navigator>
			<scroll-view class="floor-list" scroll-x>
				<view class="scoll-wrapper">
					<view v-for="(item, index) in seckill" :key="index" class="floor-item"
						@click="navToSeckillPage(item)">
						<image :src="item.resources.img | smallImage(250)" mode="aspectFill"></image>
						<text class="title clamp">{{item.name}}</text>
						<text class="price">￥{{item.price[0] | 1000}}</text>
					</view>
				</view>
			</scroll-view>
		</view>
		<!-- 拼团 -->
		<view v-if="isGroupPurchase && groupPurchase.length">
			<navigator url="../groupPurchase/list" hover-class="none" class="f-header m-t">
				<image src="/static/temp/h1.png"></image>
				<view class="tit-box">
					<text class="tit">精品团购</text>
					<text class="tit2">Boutique Group Buying</text>
				</view>
				<text class="yticon icon-you"></text>
			</navigator>
			<view class="group-section">
				<swiper class="g-swiper" :duration="500">
					<swiper-item class="g-swiper-item" v-for="(item, index) in groupPurchase" :key="index"
						v-if="index%2 === 0">
						<navigator :url="`/pages/product/detail?id=${item.good_id}`" hover-class="none"
							class="g-item left">
							<image :src="item.resources.img | smallImage(250)" mode="aspectFill"></image>
							<view class="t-box">
								<text class="title clamp">{{item.name}}</text>
								<view class="price-box">
									<text class="price">￥{{item.price[0] | 1000}}</text>
									<text class="m-price">￥{{item.originalPrice[0] | 1000}}</text>
								</view>

								<view class="pro-box">
									<view class="progress-box">
										<progress :percent="item.progress" activeColor="#fa436a" active
											stroke-width="6" />
									</view>
									<text>{{item.number}}人成团</text>
								</view>
							</view>
						</navigator>
						<template v-if="groupPurchase[index+1]">
							<navigator :url="`/pages/product/detail?id=${groupPurchase[index+1].good_id}`"
								hover-class="none" class="g-item right">
								<image :src="groupPurchase[index+1].resources.img" mode="aspectFill"></image>
								<view class="t-box">
									<text class="title clamp">{{groupPurchase[index+1].name}}</text>
									<view class="price-box">
										<text class="price">￥{{groupPurchase[index+1].price | 1000}}</text>
										<text class="m-price">￥{{groupPurchase[index+1].originalPrice | 1000}}</text>
									</view>
									<view class="pro-box">
										<view class="progress-box">
											<progress :percent="groupPurchase[index+1].progress" activeColor="#fa436a"
												active stroke-width="6" />
										</view>
										<text>{{groupPurchase[index+1].number}}人成团</text>
									</view>
								</view>
							</navigator>
						</template>
					</swiper-item>

				</swiper>
			</view>
		</view>
		<!-- 为你推荐 -->
		<view class="f-header m-t">
			<image src="/static/temp/h1.png"></image>
			<view class="tit-box">
				<text class="tit">为你推荐</text>
				<text class="tit2">Recommend To You</text>
			</view>
		</view>

		<view class="guess-section">
			<view v-for="(item, index) in goodsList" :key="index" class="guess-item" @click="navToDetailPage(item)">
				<view class="image-wrapper">
					<image :src="item.resources.img | smallImage(250)" mode="aspectFill" lazy-load></image>
				</view>
				<text class="title clamp">{{item.name}}</text>
				<text class="price">￥{{item.order_price | 1000}}</text>
			</view>
		</view>


	</view>
</template>

<script>
	import Good from '../../api/good'
	import Banner from '../../api/banner'
	import moment from 'moment'
	import {
		getList as seckillList
	} from '@/api/seckill'
	import {
		getList as groupPurchaseList
	} from '@/api/groupPurchase'
	import CountDownTime from '@/pages/seckill/components/CountDownTime';
	import {
		verifyPlugin
	} from '@/api/plugin'
	export default {
		components: {
			CountDownTime
		},
		data() {
			return {
				modalName: null,
				wechat: null,
				guidanceMy: false,
				titleNViewBackground: '',
				swiperCurrent: 0,
				swiperLength: 0,
				carouselList: [],
				goodsList: [],
				adData: {},
				ctegory: [],
				isSeckill: false,
				seckill: [],
				seckillTime: 0,
				seckillActiveTime: '',
				isGroupPurchase: false,
				groupPurchase: []
			};
		},

		onLoad() {
			this.loadData()
			// #ifdef MP-WEIXIN 
			this.wechat = uni.getStorageSync('dsshopUserInfo').wechat
			// #endif
			if (!uni.getStorageSync('applyDsshopGuidanceMy')) {
				this.guidanceMy = true
			}
		},
		onShow() {
			getApp().showDsshopCartNumber()
			this.getVerifyPlugin()
		},
		methods: {
			/**
			 * 请求数据
			 */
			async loadData() {
				const that = this
				// 轮播
				await Banner.getList({
					limit: 7,
					type: 1,
					state: 0,
					sort: '+sort'
				}, function(res) {
					that.carouselList = res.data
					that.swiperLength = res.data.length
				})
				// 首页广告
				/*
				await Banner.getList({
					type: 1,
					limit: 1,
					state: 0,
					sort: '+sort'
				}, function(res) {
					that.adData = res.data[0]
				})
				*/
				await Banner.getList({
					id: 14,
					state: 0,
					sort: '+sort'
				}, function(res) {
					that.adData = res.data[3]
				})

				// 推荐商品
				await Good.getList({
					limit: 10,
					is_recommend: 1,
				}, function(res) {
					that.goodsList = res.data
				})
				// 推荐分类
				await Good.goodCategory({
					is_recommend: 1
				}, function(res) {
					//that.ctegory = res
					that.ctegory = res['fir']
				})
			},
			//轮播图切换修改背景色
			swiperChange(e) {
				const index = e.detail.current;
				this.swiperCurrent = index;
			},
			//轮播跳转
			navToWwiperPage(item) {
				if (item.item.url) {
					uni.navigateTo({
						url: item.item.url
					})
				}
			},
			//详情页
			navToDetailPage(item) {
				//测试数据没有写id，用title代替
				let id = item.id;
				uni.navigateTo({
					url: `/pages/product/detail?id=${id}`
				})
			},
			// 秒杀详情页
			navToSeckillPage(item) {
				let id = item.good_id;
				uni.navigateTo({
					url: `/pages/product/detail?id=${id}`
				})
			},
			//跳转
			navTo(url) {
				if (url) {
					uni.navigateTo({
						url
					})
				}
			},
			// #ifdef MP-WEIXIN
			//弹出引导页
			popupBoot() {
				this.modalName = 'guidanceMy'
				this.guidanceMy = false
				uni.setStorageSync('applyDsshopGuidanceMy', true)
			},
			// 引导添加小程序
			setGuidanceMy() {
				this.guidanceMy = false
				uni.setStorageSync('applyDsshopGuidanceMy', true)
			},
			// #endif
			getVerifyPlugin() {
				const that = this
				verifyPlugin(['seckill', 'groupPurchase'], function(res) {
					if (res.seckill) {
						that.isSeckill = true
						that.getSeckill()
					}
					if (res.groupPurchase) {
						that.isGroupPurchase = true
						that.getGroupPurchase()
					}
				})
			},
			// 获取拼团列表
			getGroupPurchase() {
				let that = this
				groupPurchaseList({
					limit: 10,
					state: 1
				}, function(res) {
					that.groupPurchase = res.data
				})
			},
			// 获取秒杀列表
			getSeckill() {
				let time = moment().format('YYYY-MM-DD HH:00:00')
				let that = this
				if (moment().format('HH') % 2 !== 0) {
					time = moment().subtract(1, 'hour').format('YYYY-MM-DD HH:00:00')
				}
				this.seckillActiveTime = moment(time, "YYYY-MM-DD HH:00:00").format('HH')
				this.seckillTime = (moment(time, "YYYY-MM-DD HH:00:00").add(2, 'hour') - moment().valueOf()) / 1000
				seckillList({
					limit: 10,
					time: time,
					sort: '-id',
					state: 1
				}, function(res) {
					that.seckill = res.data
				})
			},
			// 秒杀倒计时结束
			endSeckillTime() {
				this.getSeckill()
			}
		},
		// #ifndef MP
		// 标题栏input搜索框点击
		onNavigationBarSearchInputClicked: async function(e) {
			this.$api.msg('点击了搜索框');
		},
		//点击导航栏 buttons 时触发
		onNavigationBarButtonTap(e) {
			const index = e.index;
			if (index === 0) {
				this.$api.msg('点击了扫描');
			} else if (index === 1) {
				// #ifdef APP-PLUS
				const pages = getCurrentPages();
				const page = pages[pages.length - 1];
				const currentWebview = page.$getAppWebview();
				currentWebview.hideTitleNViewButtonRedDot({
					index
				});
				// #endif
				uni.navigateTo({
					url: '/pages/notice/notice'
				})
			}
		}
		// #endif
	}
</script>

<style lang="scss">
	@import "./scss/index";
</style>
