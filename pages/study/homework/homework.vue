<template>
	<view class="homework">
		<web-view ref="webview" :src="url" @message="getMessage"></web-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userId: '',
				url: '/hybrid/html/test.html?data='
			}
		},
		onLoad() {
			this.getUserInfo()
		},
		methods: {
			// 获取用户信息
			getUserInfo() {
				let token = uni.getStorageSync("token")
				uni.request({
					url: 'http://10.3.41.200:8888/user/getUserInfo',
					method: 'GET',
					header: {
						"Authorization": token
					},
					success: (res) => {
						this.userId = res.data.data.userInfo.userId
						this.url = this.url + this.userId
					}
				})
			}
		}
	}
</script>

<style>

</style>
