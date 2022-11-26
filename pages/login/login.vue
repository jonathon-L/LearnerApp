<template>
	<view class="content">
		<view class="logo">
			<image src="../../static/logo.png" mode="center"></image>
		</view>
		<view class="user001">
			<view class="userName">
				<input type="text" placeholder="请输入手机号/用户名" v-model="name">
			</view>

			<view class="userPassword">
				<input type="password" placeholder="请输入密码" v-model="password">
			</view>

			<view class="btn" @click="login">立即登录</view>

			<view class="regist" @click="toRegist">点击注册</view>
		</view>
	</view>

</template>

<script>
	export default {
		data() {
			return {
				name: '',
				password: ''

			}
		},
		methods: {
			toRegist() {
				uni.navigateTo({
					url: "/pages/regist/regist"
				})
			},
			login() {
				if (this.name && this.password) {
					uni.request({

						url: 'http://10.3.41.200:8888/user/loginByPhone',
						method: 'POST',
						header: {
							"content-type": "application/json",
						},

						data: {
							'phone': this.name,
							'password': this.password
						},
						success: (res) => {
							console.log(res.data)
							if(res.data.success == true) {
								console.log(res.data)
								uni.setStorageSync('token', res.data.data.token)
								uni.reLaunch({
									url: '/pages/study/study',
								})
							} else {	
								uni.showToast({
									title: res.data.message
								})
							}
							
						},

					})


				}

			}
		},
	}
</script>

<style lang="scss">
	image {
		width: 100%;
		border-radius: 50%;
	}

	page {
		background-color: #3F536E;
	}

	input {
		width: 70%;
		height: 90upx;
		background-color: #fff;
		font-weight: 200;
		border-radius: 45upx;
	}

	.userName {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;

	}

	.userPassword {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-top: 15px;

	}

	.btn {
		position: relative;
		margin-top: 15px;
		display: flex;
		width: 70%;
		height: 90upx;
		justify-content: center;
		align-items: center;
		left: 15%;
		border-radius: 45upx;
		color: #f06c7a;
		background-color: #fff;
		font-size: 40upx;

	}

	.regist {
		position: relative;
		margin-top: 15px;
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 45upx;
		color: #fff;
		font-size: 40upx;
	}
</style>
