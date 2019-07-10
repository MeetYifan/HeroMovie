<template name="trailerStars">
	<view>
		<view class="movieScore" style="display: flex;">
			<image v-for="(yel, index) in yellowScore" src="/static/icos/star-yellow.png" :key="index" class="starIcon"></image>
			<image v-for="(gra, index) in grayScore" src="/static/icos/star-gray.png" :key="index" class="starIcon"></image>
			<view v-if="showNum == 1" class="scoreNum">{{innerScore}}</view>
		</view>
	</view>
</template>

<script>
	export default {

		name: "trailerStars",

		data() {
			return {
				yellowScore: 0,
				grayScore: 5
			};
		},
		
		props: {
			innerScore: 0, // 定义传入的分数
			showNum: 0     // 是否显示，1为显示，0为不显示
		},
		
		// 生命周期，组件创建完成后被调用
		created() {
			// console.log(this.showNum)
			// console.log(this.innerScore)
			
			var tempScore = 0
			if(this.innerScore != null && this.innerScore != undefined && this.innerScore != ''){
				tempScore = this.innerScore
			}
			
			var yellowScore = parseInt(tempScore / 2)
			var grayScore = 5 - yellowScore
			
			
			this.yellowScore = yellowScore
			this.grayScore = grayScore
			
		}
		
	}
</script>

<style>
	.movieScore {
		width: 200upx;
		display: flex;
		flex-direction: row;
		padding: 6upx 0;
		align-items: center;
	}

	.starIcon {
		width: 20upx;
		height: 20upx;
	}

	.scoreNum {
		font-size: 20upx;
		color: grey;
		margin-left: 8upx;
	}
</style>
