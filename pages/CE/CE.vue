<template>
	<view>
			<button @click="add" type="default" class="cleck">书写展示</button>
			<button type="default" class="cleck1" @click="search">以图搜题</button>
		<view class="text">
			<div v-if="img == ''"><批改或搜题结果></div>
			<image v-else :src="img"></image>
		</view>
	</view>
</template>

<script>
	export default{
		props:['path'],
		data(){
			return{
				srcPath:this.path,
				img: '',
				token: '',
			}
		},
		onLoad() {
			this.token = uni.getStorageSync("token")
			this.photo('languageChecks')
		},
		methods:{
			search() {
				this.photo('searchQuestions')
			},
			add(){
				/* plus.runtime.launchApplication(  
				        {  
				            pname: 'com.taobao.taobao'  
				        },  
				        function(e) {  
				            console.log('Open system default browser failed: ' + e.message);  
				    }  
				    ); */
				// console.log(this.srcPath)
				plus.runtime.openURL("http://www.baidu.com")
			},
			photo(path) {
				uni.chooseImage({
					count: 1, // 图片数量
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'], //从相册选择或者拍照
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						uni.uploadFile({
							url: 'http://10.3.41.200:8888/homework/' + path, 
							filePath: tempFilePaths[0],
							name: 'homeworkImg',
							header: {
								"Authorization": this.token
							},
							success: (uploadFileRes) => {
								console.log(uploadFileRes.data);
								this.img =  JSON.parse(uploadFileRes.data).data.url
							}
						});
					}
				});
			
			}
		}
		
	}
	
</script>

<style>
	page {
		background-color: #9DB6BA;
	}
	.cleck {
		margin-top: 0%;
		display: block;
		float: right;
		width: 30%;
	}
	.cleck1 {
		margin-left: 80%;
		display: block;
		float: right;
		width: 30%;
	}
	.text {
		color: aliceblue;
		margin: 0 auto;
		font-size: 45rpx;
		position: absolute;
		left: 25%;
		margin-top: 550rpx;
	}
</style>