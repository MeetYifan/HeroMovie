<template>
	<view class="page">

		<!-- 轮播图 -->
		<swiper :indicator-dots="true" :circular="true" :autoplay="true" :interval="3000" :duration="1000" class="carouselBox">
			<swiper-item v-for="(item, index) in carouselList" :key="index">
				<navigator open-type="navigate" class="swiper-navigator" :url="'../movie/movie?trailerId=' + item.movieId">
					<image :src="item.image" class="carouselImg"></image>
				</navigator>
			</swiper-item>
			<!-- 			<swiper-item>
				<image src="../../static/carousel/spiderman.png" class="carouselImg"></image>
			</swiper-item> -->
		</swiper>

		<!-- 热门超级英雄海报 -->
		<view class="page-block block-wrapper">

			<view class="sec-title-wapper">
				<image src="../../static/icos/hot.png" class="sec-ico"></image>
				<view class="sec-title">
					热门超英
				</view>
			</view>

			<scroll-view class="poster-content-wrapper" scroll-x="true" scroll-left="120" scroll-with-animation="true">
				<view class="single-poster">
					<view class="scroll-view-item" v-for="(item, index) in hotPosterList" :key="index">
						<navigator open-type="navigate" :url="'../movie/movie?trailerId=' + item.id">
							<image :src="item.cover" class="moviePoster"></image>
							<view class="movieName">{{item.name}}</view>
							<!-- 组件使用：3.使用 -->
							<trailerStarts :innerScore="item.score" showNum="1"></trailerStarts>
						</navigator>
					</view>
				</view>
			</scroll-view>

		</view>

		<!-- 热门超级英雄预告 -->
		<view class="page-block block-wrapper">

			<view class="sec-title-wapper">
				<image src="../../static/icos/interest.png" class="sec-ico"></image>
				<view class="sec-title">
					热门预告
				</view>
			</view>

			<view class="trailer-content-wrapper">
				<video 
				v-for="(item, index) in hotTrailerList" 
				:key="index" 
				v-if="item.poster != null && item.poster != undefined && item.poster != ''"
				:src="item.trailer"
				:poster="item.poster" 
				class="single-trailer"
				 controls>
				 <!-- v-if 是为了避免资源加载延迟，而造成页面错位 -->
				 </video>
			</view>

		</view>

		<!-- 猜你喜欢 -->
		<view class="page-block block-wrapper">

			<view class="sec-title-wapper">
				<image src="../../static/icos/guess-u-like.png" class="sec-ico"></image>
				<view class="sec-title">
					猜你喜欢
				</view>
			</view>

			<view class="recommend-content-wrapper">

				<view v-for="(item, index) in recommendList" :key="index" class="single-recommend">
					<view class="recommend-left">
						<navigator open-type="navigate" :url="'../movie/movie?trailerId=' + item.id">
							<image :src="item.cover" class="recommend-img"></image>
						</navigator>
					</view>
					<view class="recommend-center">
						<view class="recommend-name">
							{{item.name}}
						</view>
						<trailerStarts :innerScore="item.score" showNum="1"></trailerStarts>
						<view class="recommend-basicInfo">
							{{item.basicInfo}}
						</view>
						<view class="recommend-releaseDate">
							{{item.releaseDate}}
						</view>
					</view>
					<view class="recommend-right">
						<image src="../../static/icos/praise.png" class="recommend-zan-img" @click="praiseMe(index)"></image>
						<view class="recommend-zan-text">
							赞一下
						</view>
						<view :animation="animationDataArr[index]" class="recommend-zan-text animation-opacity">
							+1
						</view>
					</view>
				</view>

			</view>

		</view>

	</view>
</template>

<script>
	import common from "../../common/common.js"
	// 使用组件：引入-注册-使用
	// 组件使用：1.引入
	import trailerStarts from "../../components/trailerStarts.vue"

	export default {
		data() {
			return {
				carouselList: [],
				hotPosterList: [],
				hotTrailerList: [],
				recommendList: [],
				animationData: {},
				animationDataArr: [{}, {}, {}, {}, {}]
			}
		},
		onUnload() {
			// 清除动画数据
			this.animation = {}
		},
		onPullDownRefresh() {
			this.refresh()
		},
		onLoad() {

			// #ifdef APP-PLUS || MP-WEIXIN
			// 创建一个临时动画对象
			var animation = uni.createAnimation({

			})
			this.animation = animation
			// #endif

			var serverUrl = common.serverUrl

			// 请求轮播图数据
			uni.request({
				url: serverUrl + '/index/carousel/list',
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					// console.log(res);

					// 获取真实数据之前，务必判断状态是否为200
					if (res.data.status == 200) {
						var carouselList = res.data.data;
						this.carouselList = carouselList;
					}
				}
			});

			// 请求热门超英海报
			uni.request({
				url: serverUrl + '/index/movie/hot?type=superhero',
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					if (res.data.status == 200) {
						var hotPosterList = res.data.data;
						this.hotPosterList = hotPosterList;
					}
					// console.log(this.hotPosterList)
				}
			});

			// 请求热门超英预告 
			uni.request({
				url: serverUrl + '/index/movie/hot?type=trailer',
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					if (res.data.status == 200) {
						var hotTrailerList = res.data.data;
						this.hotTrailerList = hotTrailerList;
					}
				}
			});

			this.refresh()

		},
		methods: {
			refresh: function() {
				var that = this
				var serverUrl = common.serverUrl

				uni.showLoading({
					title: '请求资源中...',
					mask: true
				});
				uni.showNavigationBarLoading()

				// 请求推荐
				uni.request({
					url: serverUrl + '/index/guessULike',
					method: "POST",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: (res) => {
						if (res.data.status == 200) {
							var recommendList = res.data.data;
							that.recommendList = recommendList;
						}
					},
					complete() {
						uni.stopPullDownRefresh()
						uni.hideLoading()
						uni.hideNavigationBarLoading()
					}
				});
			},

			praiseMe: function(index) {
				// #ifdef APP-PLUS || MP-WEIXIN
				// 构建动画数据
				this.animation.translateX(-20).translateY(-60).opacity(1).step({
					duration: 600,
				})

				// 导出动画数据
				this.animationData = this.animation
				this.animationDataArr[index] = this.animationData.export()

				setTimeout(function() {
					this.animation.translateX(0).translateY(0).opacity(0).step({
						duration: 0,
					})
					this.animationData = this.animation
					this.animationDataArr[index] = this.animationData.export()
				}.bind(this), 600);
				// #endif
			}
		},
		components: {
			// 组件使用：2.注册
			trailerStarts
		}
	}
</script>

<style>
	@import url("index.css");
</style>
