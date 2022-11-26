<template>
	<!--头像+用户名部分-->
	<view>
		<view class="box1">
			<!-- <image class="header" src="../../static/header.png"></image> -->
			<image class="header" @click="alterTouxiang" :src="touxiang"></image>
			<view class="headtext">用户昵称：{{inputValue}}</view>
			<input v-if="inputValue == '默认名称'" class="headinput" @confirm="onKeyInput" placeholder="请输入用户名" />
		</view>
		<!--功能部分-->
		<view>
			<navigator open-type="navigate" url="/pages/light/light?type=0">
				<button type="default">光线数据分析</button>
			</navigator>
			<navigator open-type="navigate" url="/pages/light/light?type=1">
				<button type="default">阅读及书写姿势分析</button>
			</navigator>
			<navigator open-type="navigate" url="/pages/light/light?type=2">
				<button type="default">作业完成时间分析</button>
			</navigator>
			<button type="default" @click="logout()">退出账号</button>
			<button type="default" @click="deleteUser()">注销账号</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				touxiang: "../../static/headerLogo.png",
				inputValue: '默认名称',
				token: ''
			}
		},

		onLoad() {
			this.token = uni.getStorageSync("token")
			this.getUserInfo()
		},

		methods: {
			//修改用户信息
			onKeyInput: function(event) {
				this.inputValue = event.target.value
				uni.request({
					url:'http://10.3.41.200:8888/user/updateUser',
					method:'PUT',
					header: {
						"Authorization": this.token
					},
					data:{
						username:event.target.value,
					},
					success: (res) => {
						if(res.data.success == true){
							console.log(res.data)
							// this.inputValue = event.target.value
						}else{
							uni.showToast({
								title:res.data.message
							})
							console.log(res.data)
						}
					}
				})
			},
			
			//修改头像
			alterTouxiang(){
				
				uni.chooseImage({
					count: 1, // 图片数量
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'], //从相册选择或者拍照
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						uni.uploadFile({
							url: 'http://10.3.41.200:8888/oss/fileoss',
							filePath: tempFilePaths[0],
							name: 'file',
							header: {
								"Authorization": this.token
							},
							
							success: (uploadFileRes) => {
								console.log(uploadFileRes.data);
								this.touxiang = JSON.parse(uploadFileRes.data).data.url
								
								uni.request({
									url:'http://10.3.41.200:8888/user/updateUser',
									method:'PUT',
									header: {
										"Authorization": this.token
									},
									data:{
										avatarUrl:this.touxiang,
									},
									success: (res) => {
										if(res.data.success == true){
											console.log(res.data)
											// this.inputValue = event.target.value
										}else{
											uni.showToast({
												title:res.data.message
											})
											console.log(res.data)
										}
									}
								})
								
								
							}
						});
					}
				});
				
				
			},
			
			
			// 退出登录
			logout() {
				uni.request({
					url: 'http://10.3.41.200:8888/user/logout',
					method: 'GET',
					header: {
						"Authorization": this.token
					},
					success: (res) => {
						if (res.data.success == true) {
							uni.showToast({
								title: "成功退出"
							})
							this.token = ""
							uni.removeStorageSync('token')
							uni.reLaunch({
								url: '/pages/login/login',
							})
						} else {
							uni.showToast({
								title: "退出失败"
							})
						}

					}
				})
			},

			// 注销账号
			deleteUser() {
				if (this.token == undefined || this.token == '') {
					this.token = uni.getStorageSync("token")
					console.log(this.token);
				}
				console.log(this.token);
				uni.showModal({
					title: '提示',
					content: '是否确认注销账号',
					success: (res) => {
						if (res.confirm) {
							uni.request({
								url: 'http://10.3.41.200:8888/user/deleteUser',
								method: 'DELETE',
								header: {
									"Authorization": this.token
								},
								success: (res) => {
									if (res.data.success == true) {
										uni.showToast({
											title: "注销成功"
										})
										uni.reLaunch({
											url: '/pages/login/login',
										})
									} else {
										uni.showToast({
											title: "注销失败"
										})
									}
								}
							})
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				});
			},

			// 获取用户信息
			getUserInfo() {
				uni.request({
					url: 'http://10.3.41.200:8888/user/getUserInfo',
					method: 'GET',
					header: {
						"Authorization": this.token
					},
					success: (res) => {
						this.touxiang = res.data.data.userInfo.avatarUrl == null ? "../../static/头像.png" : res
							.data.data.userInfo.avatarUrl
						this.inputValue = res.data.data.userInfo.username == null ? "默认名称" : res.data.data
							.userInfo.username
					}
				})
			}
		
			//修改用户信息
			
		},
	
		
	}
</script>

<style>
	.box1 {
		font-size: 40rpx;
		width: 100%;
		height: 300rpx;
		background-color: antiquewhite;
	}

	.header {
		border-style: ridge;
		border-radius: 50%;
		width: 160rpx;
		height: 160rpx;
		margin: auto;
		display: block;
		justify-content: center;
		align-items: center;
	}

	.headtext {
		margin-top: 20rpx;
		text-align: center;

	}

	.headinput {
		text-align: center;
	}
</style>
