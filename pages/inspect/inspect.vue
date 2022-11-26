<template>
	<view>
		<view class="size">
			<view class="boxx">
				<navigator open-type="navigate" url="/pages/CE/CE">
					<button @click="photo" type="default">语文检查</button>
				</navigator>
			</view>
			<view class="boxx">
				<navigator open-type="navigate" url="/pages/Math/Math">
					<button :path='mypath' @click="photo" type="default">数学检查</button>
				</navigator>
			</view>
			<view class="boxx">
				<navigator open-type="navigate" url="/pages/EN/EN">
					<button @click="photo" type="default">英语检查</button>
				</navigator>
			</view>
		</view>
	</view>
</template>

<script>
	import path from '../Math/Math.vue'
	export default{
		components:{
			path
		},
		data(){
			return{
				mypath:''
			}
		},
		methods:{
			photo(){
				// uni.chooseImage({
				// 	sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
				// 	sourceType: ['album','camera'], //从相册选择
				// 	success: function (res) {
				// 		console.log(JSON.stringify(res.tempFilePaths));
				// 		this.mypath = res.tempFilePaths
				// 	}
				// })
			}
		}
		
	}
</script>

<style>
	page {
		background-color: #CFD7DF;
	}
	.size {
		margin: auto;
		display: block;
		justify-content: center;
		align-items: center;
		width: 75%;
	},
	.boxx {
		margin-top: 150rpx;
		margin-bottom: 80rpx;
		border-style: ridge;
	}
</style>