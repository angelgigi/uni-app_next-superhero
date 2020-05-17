<template>
	<view class="page page-fill">

		<view class="page-block info-list">
			<!-- 头像 -->

			<view class="item-wapper face-line-upbottom" @click="operator">
				<view class="info-words">头像</view>

				<view class="right-wapper">
					<image :src="globalUser.faceImage" class="face"></image>
					<view class="arrow-block">
						<image src="../../static/icos/left-gray-arrow.png" class="arrow-ico"></image>
					</view>
				</view>
			</view>

			<view class="line-top">
				<view class="line"></view>
			</view>

			<!-- 昵称 -->
			<view class="item-wapper" @click="modifyNickname">
				<view class="info-words">昵称</view>

				<view class="right-wapper">
					<view class="gray-fields">
						{{globalUser.nickname}}
					</view>
					<view class="arrow-block">
						<image src="../../static/icos/left-gray-arrow.png" class="arrow-ico"></image>
					</view>
				</view>
			</view>

			<view class="line-top">
				<view class="line"></view>
			</view>

			<!-- 生日 -->
			<view class="item-wapper" @click="modifyBirthday">
				<view class="info-words">生日</view>

				<view class="right-wapper">
					<view class="gray-fields">
						{{globalUser.birthday}}
					</view>
					<view class="arrow-block">
						<image src="../../static/icos/left-gray-arrow.png" class="arrow-ico"></image>
					</view>
				</view>
			</view>

			<view class="line-top">
				<view class="line"></view>
			</view>

			<!-- 性别 -->
			<view class="item-wapper" @click="modifySex">
				<view class="info-words">性别</view>

				<view class="right-wapper">
					<view class="gray-fields">
						<view v-if="globalUser.sex == 1">
							男
						</view>
						<view v-else-if="globalUser.sex == 0">
							女
						</view>
						<view v-else >未选择</view>
					</view>
					<view class="arrow-block">
						<image src="../../static/icos/left-gray-arrow.png" class="arrow-ico"></image>
					</view>
				</view>
			</view>

		</view>
		
		<view class="footer-wapper">
			<view class="footer-words" @click="cleanStorage">
				清理缓存
			</view>
			<view class="footer-words" style="margin-top: 10upx;" @click="logout">
				退出登录
			</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				globalUser:{}
			}
		},
		onShow() {
			var that = this;
			var globalUser = that.getGlobalUser("globalUser");
			that.globalUser = globalUser;
		},
		methods: {
			cleanStorage(){
				uni.clearStorageSync();
				uni.showToast({
					title:"清理缓存成功",
					mask:false,
					duration:1500
				})
			},
			logout(){
				var globalUser = this.getGlobalUser("globalUser")
				var serverURL = this.serverURL
				uni.request({
					url:serverURL + '/user/logout?userId=' + globalUser.id +'&qq=lee82223437',
					method:"POST",
					success: (res) => {
						if(res.data.status == 200){
							//退出成功
							uni.removeStorageSync("globalUser")
							uni.switchTab({
								url:"../index/index"
							})
						}else{
							return res.errMsgs
						}
					}
				})
			}
		}
	}
</script>

<style>
	@import url("meInfo.css");
</style>
