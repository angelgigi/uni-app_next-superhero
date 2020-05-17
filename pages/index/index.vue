<template>
	<view class="page">
		<!-- 轮播图 start -->
		<swiper class="carousel" :indicator-dots="true" autoplay="autoplay">
			<swiper-item v-for="carousel in carouselList">
				<image :src="carousel.image" class="carousel"></image>
			</swiper-item>
		</swiper>
		<!-- 轮播图 end -->
		<!-- 热门超英 start -->
		<view class="page-block super-hot">
			<view class="hot-title-wapper">
				<image src="../../static/icos/hot.png" class="hot-ico"></image>
				<view class="hot-title">热门超英</view>
			</view>
		</view>
		<scroll-view scroll-x="true" class="page-block hot">
			<view class="single-poster" v-for="superhero in hotSuperheroList">
				<view class="poster-wapper">
					<navigator open-type="navigate" :url="'../movie/movie?trailerId=' + superhero.id">
						<image :src="superhero.cover" class="poster"></image>
					</navigator>
					<view class="movie-name">{{superhero.name}}</view>
					<trailerStars :innerScore="superhero.score" showNum="1"></trailerStars>
				</view>
			</view>
		</scroll-view>
		<!-- 热门超英 end -->
		<!-- 热门预告 start -->
		<view class="page-block super-hot">
			<view class="hot-title-wapper">
				<image src="../../static/icos/interest.png" class="hot-ico"></image>
				<view class="hot-title">热门预告</view>
			</view>
		</view>
		<view class="page-block hot-movies">
			<video v-for="trailer in hotTrailerList" :id="trailer.id" :data-playingIndex="trailer.id" @play="meIsPlaying" :src="trailer.trailer"
			 :poster="trailer.poster" class="hot-movie-single" controls></video>
		</view>
		<!-- 热门预告 end -->

		<!-- 猜你喜欢 start -->
		<view class="page-block super-hot">
			<view class="hot-title-wapper">
				<image src="../../static/icos/guess-u-like.png" class="hot-ico"></image>
				<view class="hot-title">猜你喜欢</view>
			</view>
		</view>
		<view class="page-block guess-u-like">
			<view class="single-like-movie" v-for="(guess,gIndex) in guessULikeList">
				<image :src="guess.cover" class="like-movie"></image>
				<view class="movie-desc">
					<view class="movie-title">{{guess.name}}</view>
					<trailerStars :innerScore="guess.score" showNum="0"></trailerStars>
					<view class="movie-info">{{guess.basicInfo}}</view>
					<view class="movie-info">{{guess.releaseDate}}</view>
				</view>
				<view class="movie-oper" :data-gIndex="gIndex" @click="praiseMe">
					<image src="../../static/icos/praise.png" class="praise-ico"></image>
					<view class="praise-me">点赞</view>
					<view :animation="animationDataArr[gIndex]" class="praise-me animation-opacity">+1</view>
				</view>
			</view>
		</view>
		<!-- 猜你喜欢 end -->
	</view>
</template>

<script>
	import trailerStars from "../../components/trailerStars.vue"
	
	export default {
		data() {
			return {
				carouselList: [],
				hotSuperheroList: [],
				hotTrailerList: [],
				guessULikeList: [],
				animationData: {},
				animationDataArr: [{}, {}, {}, {}, {}]

			}
		},
		onUnload() {
			//页面卸载的时候，清除动画数据
			this.animationData = {};
			this.animationDataArr = [{}, {}, {}, {}, {}]
		},
		onPullDownRefresh() {
			this.refresh()
		},
		onHide() {
			if (this.videoContext) {
				this.videoContext.pause()
			}
		},
		onLoad() {
			var that = this;
			//在页面创建的时候，创建一个临时动画对象
			// #ifdef APP-PLUS || MP-WEIXIN
			this.animation = uni.createAnimation()
			// #endif
			//通过挂载到main.js中获取服务器的地址，作为全局变量
			var serverURL = that.serverURL;
			//请求轮播图数据
			uni.request({
				url: serverURL + '	',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						var carouselList = res.data.data
						that.carouselList = carouselList
					}

				}
			});
			//请求热门超英
			uni.request({
				url: serverURL + '/index/movie/hot?type=superhero&qq=122212489',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						console.log(res.data)
						var hotSuperheroList = res.data.data
						this.hotSuperheroList = hotSuperheroList
					}
				},

			})
			//请求超英预告
			uni.request({
				url: serverURL + '/index/movie/hot?type=trailer&qq=122212489',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						console.log(res.data)
						var hotTrailerList = res.data.data
						this.hotTrailerList = hotTrailerList
					}

				}
			})
			//查询猜你喜欢
			that.refresh()
		},
		methods: {
			//播放一个视频的时候，需要暂停其他正在播放的视频(通过判断是否是当前id)
			meIsPlaying(e) {
				var that = this;
				var trailerId = "";
				if (e) {
					trailerId = e.currentTarget.dataset.playingindex
				}
				console.log(trailerId)
				var hotTrailerList = that.hotTrailerList;
				for (var i = 0; i < hotTrailerList.length; i++) {
					var tempId = hotTrailerList[i].id;
					if (tempId != trailerId) {
						uni.createVideoContext(tempId).pause()
					}
				}
			},
			refresh() {
				var that = this;
				uni.showLoading({
					mask: true
				})
				uni.showNavigationBarLoading()
				var serverURL = that.serverURL;
				uni.request({
					url: serverURL + '/index/guessULike?qq=122212489',
					method: "POST",
					success: (res) => {
						console.log(res)
						if (res.data.status == 200) {
							var guessULikeList = res.data.data
							this.guessULikeList = guessULikeList
						}
					},
					complete: () => {
						uni.hideNavigationBarLoading();
						uni.hideLoading();
						uni.stopPullDownRefresh();
					}
				})
			},
			//实现点赞动画效果
			praiseMe(e) {
				// #ifdef APP-PLUS || MP-WEIXIN
				var gIndex = e.currentTarget.dataset.gindex;
				//构建动画数据，并且通过step来表示这组动画的完成
				this.animation.translateY(-60).opacity(1).step({
					duration: 400
				});
				//导出动画数据到view组件，实现组建的动画效果
				this.animationData = this.animation
				this.animationDataArr[gIndex] = this.animationData.export()
				//还原动画
				setTimeout(function() {
					this.animation.translateY(0).opacity(0).step({
						duration: 0
					});
					this.animationData = this.animation
					this.animationDataArr[gIndex] = this.animationData.export()
				}.bind(this), 500);
				// #endif
			}

		},
		components: {
			trailerStars
		}
	}

</script>

<style>
	@import url("index.css");
</style>
