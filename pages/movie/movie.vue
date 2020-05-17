<template>

	<view class="page">
		<!-- 视频播放 start -->
		<view class="player">
			<video id="myTrailer" :src="trailerInfo.trailer" :poster="trailerInfo.poster" class="movie" controls=""></video>
		</view>
		<!-- 视频播放 end -->

		<!-- 影片基本信息 start -->
		<view class="movie-info">
			<navigator :url="'../cover/cover?cover=' + trailerInfo.cover">
				<image :src="trailerInfo.cover" class="cover"></image>
			</navigator>
			<view class="movie-desc">
				<view class="title">{{trailerInfo.name}}</view>
				<view class="basic-info">{{trailerInfo.basicInfo}}</view>
				<view class="basic-info">{{trailerInfo.originalName}}</view>
				<view class="basic-info">{{trailerInfo.totalTime}}</view>
				<view class="basic-info">{{trailerInfo.releaseDate}}</view>
				<view class="score-block">
					<view class="big-score">
						<view class="score-words">综合评分：</view>
						<view class="movie-score">{{trailerInfo.score}}</view>
					</view>

					<view class="score-stars">
						<block v-if="trailerInfo.score >= 0">
							<trailerStars :innerScore="trailerInfo.score" showNum="0"></trailerStars>
						</block>
						<view class="prise-counts">
							{{trailerInfo.prisedCounts}} 人点赞
						</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 影片基本信息 end -->

		<view class="line-wapper">
			<view class="line"></view>
		</view>
		<!-- 剧情介绍 start -->
		<view class="plots-block">
			<view class="plots-title">剧情介绍</view>
			<view class="plots-desc">{{trailerInfo.plotDesc}}</view>
		</view>
		<!-- 剧情介绍 end -->
		
		<!-- 演职人员 start -->
		<view class="scroll-block">
			<view class="plots-title">演职人员</view>
			<scroll-view scroll-x class="scroll-list">
				<view class="actor-wapper" v-for="(director,staffIndex) in directorArray">
					<image
					:src="director.photo"
					class="single-actor"
					mode="aspectFill"
					@click="lookStaffs"
					:data-staffIndex="staffIndex"></image>
					<view class="actor-name">{{director.name}}</view>
					<view class="actor-role">{{director.actName}}</view>
				</view>
				<view class="actor-wapper" v-for="(actor,actorIndex) in actorArray">
					<image
					:src="actor.photo"
					class="single-actor"
					mode="aspectFill"
					@click="lookStaffs"
					:data-staffIndex="actorIndex + directorArray.length"></image>
					<view class="actor-name">{{actor.name}}</view>
					<view class="actor-role">{{actor.actName}}</view>
				</view>
			</scroll-view>
		</view>
		<!-- 演职人员 end -->
		
		<!-- 剧照 start -->
		<view class="scroll-block">
			<view class="plots-title">剧照</view>
			<scroll-view scroll-x class="scroll-list">
				<image 
				v-for="(img,imgIndex) in plotPicsArray"
				:src="img"
				class="plot-image"
				mode="aspectFill"
				@click="lookMe"
				:data-imgIndex="imgIndex"></image>
			</scroll-view>
		</view>
		<!-- 剧照 end -->
	</view>
</template>

<script>
	import trailerStars from "../../components/trailerStars.vue"
	export default {
		data() {
			return {
				trailerInfo: {},
				plotPicsArray: [], // 剧照
				directorArray: [], //导演列表
				actorArray: [] //演员列表
			}
		},
		//页面初次渲染完成，获得视频上下文对象
		onReady() {
			this.videoContext = uni.createVideoContext('myTrailer')
		},
		onHide() {
			//页面隐藏时，暂停播放
			this.videoContext.pause();
		},
		onShow() {
			//页面被再次显示时，继续播放
			if(this.videoContext){
				this.videoContext.play();
			}
		},
		onLoad(params) {
			//获取上一个页面传入的参数
			var trailerId = params.trailerId
			//获取预告片的详细信息；
			var that = this;

			var serverURL = that.serverURL;
			uni.request({
				url: serverURL + '/search/trailer/' + trailerId + '/?qq=lee82223437',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						//console.log(res.data.data)
						var trailerInfo = res.data.data
						that.trailerInfo = trailerInfo
						//把剧照的字符串转换为json array
						var plotPicsArray = JSON.parse(trailerInfo.plotPics);
						that.plotPicsArray = plotPicsArray
					}
				},
				complete: () => {
					uni.hideNavigationBarLoading();
					uni.hideLoading();
				}
			})
			//获取导演
			uni.request({
				url: serverURL + '/search/staff/' + trailerId + '/1' + '/?qq=lee82223437',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						//console.log(res.data.data)
						that.directorArray = res.data.data
					}
				},
				complete: () => {
					uni.hideNavigationBarLoading();
					uni.hideLoading();
				}
			})
			//获取演员
			uni.request({
				url: serverURL + '/search/staff/' + trailerId + '/2' + '/?qq=lee82223437',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						//console.log(res.data.data)
						that.actorArray = res.data.data
					}
				},
				complete: () => {
					uni.hideNavigationBarLoading();
					uni.hideLoading();
				}
			})
		},
		//小程序端的分享，分享到微信群或微信好友
		onShareAppMessage(res) {
			var that = this;
			return {
				title:that.trailerInfo.name,
				path:'/pages/movie/movie?trailerId=' + that.trailerInfo.id
			}
		},
		//监听导航栏的按钮
		onNavigationBarButtonTap(e) {
			var index = e.index;
			//console.log(index);
			
			var that = this;
			var trailerInfo = that.trailerInfo;
			var trailerId = trailerInfo.id;
			var trailerName = trailerInfo.name;
			var cover = trailerInfo.cover;
			var poster = trailerInfo.poster
			if(index == 0){
				uni.share({
				    provider: "weixin",
				    scene: "WXSenceTimeline",
				    type: 0,
				    href: "http://www.imovietrailer.com/#/pages/movie/movie?trailerId="+trailerId,
				    title: "NEXT超英预告：《"+ trailerName +"》",
				    summary: "NEXT超英预告：《"+ trailerName +"》",
				    imageUrl: cover,
				    success: function (res) {
				        console.log("success:" + JSON.stringify(res));
				    }
				});
			}
		},
		methods: {
			//图片预览
			lookMe(e){
				var that = this;
				var imgIndex = e.currentTarget.dataset.imgindex
				uni.previewImage({
					current:that.plotPicsArray[imgIndex],
					urls:that.plotPicsArray
				})
			},
			lookStaffs(e){
				var that = this;
				var staffIndex = e.currentTarget.dataset.staffindex;
				//拼接一个新数组，导演+演员
				var directorArray = that.directorArray;
				var actorArray = that.actorArray;
				var newStaffArray = [];
				newStaffArray = newStaffArray.concat(directorArray).concat(actorArray);
				var urls = [];
				for(var i = 0;i< newStaffArray.length; i++){
					var tempPhoto = newStaffArray[i].photo;
					urls.push(tempPhoto);
				}
				//图片预览函数
				uni.previewImage({
					current:urls[staffIndex],
					urls:urls
				})
				
			}
		},
		components: {
			trailerStars
		}
	}
</script>

<style>
	@import url("movie.css");
</style>
