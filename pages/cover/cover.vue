<template>
	<view class="black">
		<image 
		:src="cover" 
		mode="widthFix"
		class="cover"
		@longpress="opertor"></image>	
	</view>
</template>

<script>
	export default {
		data() {
			return {
				cover:""
			}	
		},
		onLoad(params){
			var cover = params.cover;
			this.cover = cover;
			//通过api修改导航栏的属性
			uni.setNavigationBarColor({
				frontColor:"#FFFFFF",
				backgroundColor:"#000000"
			})
		},
		methods: {
			opertor(){
				var that = this;
				uni.showActionSheet({
					itemList:["保存图片到本地"],
					success:function(res){
						if(res.tapIndex == 0){
							uni.showLoading({
								title:"图片保存中"
							})
							uni.downloadFile({
								url:that.cover,
								success:function(result){
									var tempFilePath = result.tempFilePath
									//console.log(tempFilePath)
									uni.saveImageToPhotosAlbum({
										filePath:tempFilePath,
										success:function(){
											uni.showToast({
												title:"保存成功",
												duration:2000
											})
										},
										complete:function(){
											uni.hideLoading();
										}
									})
								}
							})
						}
					}
				})
			}
		}
	}
</script>

<style>
.black{
	background-color: #000000;
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	justify-content: center;
	position: fixed;
}
.cover{
	align-self: center;
}
</style>
