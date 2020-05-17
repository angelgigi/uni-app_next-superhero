<template>
	<view class="page">
		<view class="search-block">
			<view class="search-ico-wapper">
				<image src="../../static/icos/search.png" class="search-ico"></image>
			</view>
			<input placeholder="搜索预告" maxlength="10" class="search-text" confirm-type="search" @confirm="searchMe" />
		</view>
		<view class="movie-list page-block">
			<view class="movie-wapper" v-for="trailer in trailerList">
				<image :src="trailer.cover"  :data-trailerId="trailer.id" @click="showTrailer" class="poster"></image>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				trailerList: [],
				keywords: "", //搜索的关键字
				page: 1, //当前第几页
				totalPages: 1 //总页数

			}
		},
		onReachBottom() {
			var that = this;

			var page = that.page + 1; //查询下一页面，当前页数累加1
			var keywords = that.keywords; //获取当前页面中data里的搜索值
			var totalPages = that.totalPages; //获取总页数

			//如果当前需要分页的分页数和总页数相等，就不分页
			if (page > totalPages) {
				return;
			}
			that.pagetrailerList(keywords, page, 15)
		},
		methods: {
			showTrailer(e){
				var trailerId = e.currentTarget.dataset.trailerid;
				// 页面跳转接口api
				uni.navigateTo({
					url:"../movie/movie?trailerId="+trailerId
				})
			},
			pagetrailerList(keywords, page, pageSize) {
				var that = this;
				uni.showLoading({
					mask: true,
					title: "请稍后..."
				})
				uni.showNavigationBarLoading()
				var serverURL = that.serverURL;
				uni.request({
					url: serverURL + '/search/list?qq=lee82223437&keywords=' + keywords + '&page= ' + page + '&pageSize=' + pageSize,
					method: "POST",
					success: (res) => {
						if (res.data.status == 200) {
							
							var tempList = res.data.data.rows
							that.trailerList = that.trailerList.concat(tempList);
							that.totalPages = res.data.data.total; //获取总页数
							that.page = page; //覆盖当前页面里的page
						}
					},
					complete: () => {
						uni.hideNavigationBarLoading();
						uni.hideLoading();
					}
				})
			},
			searchMe(e) {
				var that = this;

				//获取搜索内容
				var value = e.detail.value;
				that.keywords = value;
				that.trailerList = []
				that.pagetrailerList(value, 1, 15)

			}
		},
		onLoad() {
			var that = this;
			uni.showLoading({
				mask: true,
				title: "请稍后..."
			})
			uni.showNavigationBarLoading()
			var serverURL = that.serverURL;
			uni.request({
				url: serverURL + '/search/list?qq=lee82223437&keywords=&page=&pageSize=',
				method: "POST",
				success: (res) => {
					if (res.data.status == 200) {
						
						var trailerList = res.data.data.rows
						this.trailerList = trailerList
					}
				},
				complete: () => {
					uni.hideNavigationBarLoading();
					uni.hideLoading();
				}
			})
		}
	}
</script>

<style>
	@import url("search.css");
</style>
