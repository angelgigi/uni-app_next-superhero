<template>
	<view class="body">
		<form @submit="formSubmit">
			<view class="face-wapper">
				<image src="../../static/icos/default-face.png" class="face"></image>
			</view>

			<view class="info-wapper">
				<label class="words-lbl">账号</label>
				<input name="username" type="text" value="" class="input" placeholder="请输入用户名" placeholder-class="graywords" />
			</view>

			<view class="info-wapper" style="margin-top: 40upx;">
				<label class="words-lbl">密码</label>
				<input name="password" type="text" value="" password="true" class="input" placeholder="请输入密码" placeholder-class="graywords" />
			</view>

			<button type="primary" form-type="submit" style="margin-top: 60upx;width: 90%;">注册/登录</button>
		</form>
		<!-- 第三方登录H5不支持，所以隐藏掉 -->
		<!-- #ifndef H5 -->
		<view class="third-wapper">

			<view class="third-line">
				<view class="single-line">
					<view class="line"></view>
				</view>

				<view class="third-words">第三方账号登录</view>

				<view class="single-line">
					<view class="line"></view>
				</view>
			</view>

			<view class="third-icos-wapper">
				<!-- 5+app 用qq/微信/微博 登录 小程序用微信小程序登录 h5不支持 -->
				<!-- #ifdef APP-PLUS -->
				<image src="../../static/icos/weixin.png" data-logintype="weixin" @click="appOAuthLogin" class="third-ico"></image>
				<image src="../../static/icos/QQ.png" data-logintype="qq" @click="appOAuthLogin" class="third-ico" style="margin-left: 80upx;"></image>
				<image src="../../static/icos/weibo.png" data-logintype="sinaweibo" @click="appOAuthLogin" class="third-ico" style="margin-left: 80upx;"></image>
				<!-- #endif -->
				<!-- #ifdef MP-WEIXIN -->
				<button open-type='getUserInfo' @getuserinfo="wxLogin" class="third-btn-ico">
				</button>
				<!-- #endif -->
			</view>
		</view>
		<!-- #endif -->

	</view>
</template>

<script>
	export default {
		data() {
			return {

			}
		},
		methods: {
			appOAuthLogin(e) {
				var that = this;
				//获取用户的登录类型
				var logintype = e.currentTarget.dataset.logintype;
				uni.login({
					provider: logintype,
					success(loginRes) {
						// 授权登录成功以后，获取用户的信息
						uni.getUserInfo({
							provider: logintype,
							success(info) {
								// console.log(JSON.stringify(info));
								var userInfo = info.userInfo;
								var face = "";
								var nickname = "";
								var openIdOrUid = ";"
								if (logintype == "weixin") {
									var face = "userInfo.avatarUrl";
									var nickname = "userInfo.nickName";
									var openIdOrUid = "userInfo.openId";
								} else if (logintype == "qq") {
									var face = "userInfo.avatarUrl";
									var nickname = "userInfo.nickname";
									var openIdOrUid = "userInfo.figureurl_qq_2;";
								} else if (logintype == "sinaweibo") {
									openIdOrUid = userInfo.id;
									nickname = userInfo.nickname;
									face = userInfo.avatar_large;
								}
								// 调用开发者后台，执行一键注册或登录
								uni.request({
									url: that.serverURL + "/appUnionLogin/" + logintype + '&qq=lee82223437',
									method: "POST",
									data: {
										"openIdOrUid": openIdOrUid,
										"nickname": nickname,
										"face": face
									},
									success(result) {
										if (result.data.status == 200) {
											var userInfo = result.data.data;
											// 保存用户信息到全局的缓存中
											console.log(userInfo)
											uni.setStorageSync("globalUser", userInfo);
											// 切换页面跳转，使用tab切换的api
											uni.switchTab({
												url: "../me/me"
											});
										}else{
											console.log(result)
										}
									},
									fail(result) {
										console.log(result)
									}
								})
							}
							
						})
					}
				})
			},
			//实现在微信小程序端的微信登录
			wxLogin(e) {
				var that = this
				// 通过微信开发能力，获得微信用户的基本信息
				var userInfo = e.detail.userInfo;
				uni.login({
					provider: "weixin",
					success(loginRes) {
						var code = loginRes.code
						//console.log(loginRes);
						var loginToWhichMP = 1;
						uni.request({
							url: that.serverURL + "/stu/mpWXLogin/" + code + '&qq=lee82223437',
							data: {
								"avataUrl": userInfo.avatarUrl,
								"nickName": userInfo.nickName,
								"whichMP": loginToWhichMP
							},
							method: "POST",
							success(res, userResult) {
								if (res.data.status == 200) {
									var userInfo = userResult.data.data;
									// 保存用户信息到全局的缓存中
									uni.setStorageSync("globalUser", UserInfo);
									// 切换页面跳转，使用tab切换的api
									uni.switchTab({
										url: "../me/me"
									});
								} else {
									console.log(res.data.msg)
								}

							}
						})
					}
				})
			},
			formSubmit(e) {
				var that = this;
				var username = e.detail.value.username;
				var password = e.detail.value.password;
				// 发起注册/登录的请求
				var serverURL = that.serverURL;
				uni.request({
					url: serverURL + '/user/registOrLogin?qq=lee82223437',
					data: {
						"username": username,
						"password": password
					},
					method: "POST",
					success: (res) => {
						//获取真实数据之前，务必判断状态是否为200
						if (res.data.status == 200) {
							var userInfo = res.data.data;
							//console.log(userInfo);
							// 保存用户信息到全局的缓存中
							uni.setStorageSync("globalUser", userInfo);
							// 切换页面跳转，使用tab切换的api

							uni.switchTab({
								url: "../me/me"
							})
						} else if (res.data.status == 500) {
							uni.showToast({
								title: res.data.msg,
								duration: 2000,
								image: "../../static/icos/error.png"
							})
						}
					}
				})
			}
		}
	}
</script>

<style>
	@import url("registLogin.css");
</style>
