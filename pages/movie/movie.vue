<template>
	<view class="">
		<view class="movie" v-if="flag">
			<view class="movieTrailer">
				<video :src="carouselData.trailer" v-if="carouselData.poster != null && carouselData.poster != undefined && carouselData.poster != ''"
				 :poster="carouselData.poster" class="movieTrailerSource" controls></video>
			</view>

			<view class="movieInfo">
				<view class="moviePoster" @click="bigPoster">
					<image :src="carouselData.cover" mode=""></image>
				</view>
				<view class="detail">
					<view class="detail-title">
						{{carouselData.name}}
					</view>
					<view class="detail-box">
						<view class="detail-basicInfo">
							{{carouselData.basicInfo}}
						</view>
						<view class="detail-originalName">
							{{carouselData.originalName}}
						</view>
						<view class="detail-releaseDate">
							{{carouselData.releaseDate}}
						</view>
					</view>
					<view class="detail-score page-block">
						<view class="score-box">
							<view class="score-text">
								综合评分
							</view>
							<view class="score-num">
								{{carouselData.score}}
							</view>
						</view>
						<view class="score-star">
							<view v-if="carouselData.score >= 0" class="">
								<trailerStarts :innerScore="carouselData.score" showNum="0"></trailerStarts>
							</view>
							<view class="score-prisedCounts">
								{{carouselData.prisedCounts}} 人点赞
							</view>
						</view>
					</view>
				</view>
			</view>

			<view class="line-wrapper">
				<view class="line">

				</view>
			</view>

			<view class="conten-box">

				<view class="plotDesc-box">
					<view class="plotDesc-title sec-title">
						剧情介绍
					</view>
					<view class="plotDesc-text">
						{{carouselData.plotDesc}}
					</view>
				</view>

				<view class="staff-box">
					<view class="staff-title sec-title">
						演职人员
					</view>
					<scroll-view class="staff-pic-wrapper" scroll-x="true" scroll-with-animation="true">
						<view v-for="(item, index) in directorArray" :key="index" class="director-single">
							<view :style="'background:url(' + item.photo + ') no-repeat center center / cover;'">
								<image :src="item.photo" class="director-pic" style="opacity: 0;"></image>
							</view>
							<view class="directorName">
								{{item.name}}
							</view>
							<view class="directorActName">
								{{item.actName}}
							</view>
						</view>
						<view v-for="(item, index) in actorArray" :key="index" class="actor-single">
							<view :style="'background:url(' + item.photo + ') no-repeat center center / cover;'">
								<image :src="item.photo" class="actor-pic" style="opacity: 0;"></image>
							</view>
							<view class="actorName">
								{{item.name}}
							</view>
							<view class="actorActName">
								{{item.actName}}
							</view>
						</view>
					</scroll-view>
				</view>

				<view class="plotPics-box">
					<view class="plotPics-title sec-title">
						剧照
					</view>
					<scroll-view class="plotPics-pic-wrapper" scroll-x="true" scroll-with-animation="true">
						<view v-for="(item, index) in carouselData.plotPics" :key="index" :data-imgIndex="index" @click="bigpLotPics(index)"
						 :style="'background:url(' + item + ') no-repeat center center / cover;'" class="plotPics-single">
							<image :src="item" class="plotPics-pic" style="opacity: 0;"></image>
						</view>
					</scroll-view>
				</view>

			</view>
		</view>

		<view class="errorTip" v-if="!flag">
			{{msg}}
		</view>
	</view>
</template>

<script>
	import common from "../../common/common.js"
	import trailerStarts from "../../components/trailerStarts.vue"

	export default {
		data() {
			return {
				carouselData: {},
				directorArray: [],
				actorArray: [],
				msg: '',
				flag: false
			}
		},
		onLoad(params) {
			var serverUrl = common.serverUrl
			var trailerId = params.trailerId

			// trailerId = "dc-1004"

			// 查询对应影片信息 
			uni.request({
				url: serverUrl + '/search?trailerid=' + trailerId,
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					// console.log(res);

					// 获取真实数据之前，务必判断状态是否为200
					if (res.data.status == 200) {
						// var flag = JSON.stringify(this.carouselData) == "{}"
						if (res.data.msg == "OK") {
							this.flag = true
							var carouselData = res.data.data[0];
							carouselData.plotPics = JSON.parse(carouselData.plotPics)
							this.carouselData = carouselData;
						} else {
							this.flag = false
							this.msg = "暂时没有资源信息"
						}

					}
				}
			});

			// 查询导演 演员信息
			uni.request({
				url: 'https://www.imovietrailer.com/superhero/search/staff/' + trailerId + '/1?qq=lee70539983',
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					// 获取真实数据之前，务必判断状态是否为200
					if (res.data.status == 200) {
						this.directorArray = res.data.data;
					}
				}
			});
			uni.request({
				url: 'https://www.imovietrailer.com/superhero/search/staff/' + trailerId + '/2?qq=lee70539983',
				method: "POST",
				header: {
					'content-type': 'application/x-www-form-urlencoded'
				},
				success: (res) => {
					// 获取真实数据之前，务必判断状态是否为200
					if (res.data.status == 200) {
						this.actorArray = res.data.data;
					}
					console.log(this.actorArray, this.directorArray)
				}
			});

		},
		methods: {
			bigpLotPics: function(index) {
				var me = this;
				// var imgIndex = e.currentTarget.dataset.imgindex
				var imgIndex = index

				uni.previewImage({
					current: me.carouselData.plotPics[imgIndex],
					urls: me.carouselData.plotPics
				})
			},
			bigPoster: function() {
				var me = this;
				var arr = [me.carouselData.cover]

				uni.previewImage({
					current: arr[0],
					urls: arr
				})
			}
		},
		components: {
			trailerStarts
		}
	}
</script>

<style>
	@import url("./movie.css");
</style>
