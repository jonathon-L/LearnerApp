<template>
	<view class="btn">
		<button type="default" plain="true" @click="pronunciation" >发音矫正</button>
		<button type="default" plain="true" @click="homework" >作业助手</button>
	</view>
</template>

<script>
	export default {
		methods:{
			pronunciation(){
				uni.navigateTo({
					url:"/pages/study/pronunciation/prounce"
				})
			},
			homework(){
				
				uni.navigateTo({
					url:"/pages/study/homework/homework"
				})
			},
		}
	}
</script>

<style>
	.btn{
		margin-top: 50%;
	}
	button{
		margin-top: 5%;
	}
</style>