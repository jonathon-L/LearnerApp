<template>
	<view class="prounciation">
		<view class="newWord" @click="addWord">
			<!-- <button type="primary" size="mini" plain="true" @click="addWord">点击生成随机数</button> -->
			<h2 v-text="word"></h2>
		</view>
		<view class="audioResult">
			<view class="exportResult" v-html="html">
			</view>
			<!-- <select name="grade" id="" v-model="num" @change="gradeChange()">
				<option value="1">一年级</option>
				<option value="2">二年级</option>
			</select><br> -->
			<view class="select" name="grade" >
				<!-- <uni-section   title="本地数据" type="line" v-model="value" >
				  <uni-data-select
				    v-model="num"
				    :localdata="range"
				    @change="gradeChange()"
					:clear="false"
				  ></uni-data-select>
				</uni-section> -->
				<view class="uni-list-cell-db">
					<picker @change="bindPickerChange" :value="index" :range="array">
						<view class="uni-input"  >{{array[index]}}</view>
						<view class="icon">
							<uni-icons type="bottom" size="20" color="white"></uni-icons>
						</view>

					</picker>
				</view>
			</view><br>
			<button type="primary" size="mini" @click="read">跟读</button><br>
			<button type="primary" size="mini" @longpress="longpressBtn()" @touchend="touchendBtn()">对照</button>
		</view>
		<!-- <view class="tabbar">
			<custom-tab-bar direction="horizontal" :show-icon="false" :selected="selectedIndex" @onTabItemTap="tabbarTaped" />
		</view> -->
		
		
		
	</view>

</template>

<script>

	// import server from './components/server.vue'
	export default {
		// components: {
		// 	server
		// },
		data() {
			return {
				selectedIndex: 0,
				word: '',
				wordsId: '',
				num: '1',
				token: '',
				duration: 60000, //录音最大值ms 60000/1分钟
				tempFilePath: '', //音频路径
				recorderManager: null,
				html: '',
				//111111111111
				// value:'1',
				// range: [ 
				//   {  text: "一年级", value: '1' },
				//   {  text: "二年级", value: '2' },
				// ],
				array: ['一年级', '二年级'],
				index: 0,
			}
		},
		onLoad() {
			this.token = uni.getStorageSync("token")
			console.log(this.token);
			uni.request({
				url: 'http://10.3.41.200:8888/pronunciation/generateWords/1',
				method: 'GET',
				header: {
					"Authorization": this.token
				},
				data: {
					text: 'uni.request'
				},
				success: (res) => {
					// console.log(res.data)
					this.word = res.data.data.info.wordsInfo
					this.wordsId = res.data.data.info.wordsId
				}
			})

		},

		methods: {
			
			// tabbarTaped (e) {
			// 	let path = e.pagePath
			// 	console.log(path);
			// 	uni.switchTab({
			// 		url:'path'
			// 	})
			// }
			bindPickerChange: function(e) {
			    console.log('picker发送选择改变，携带值为', e.detail.value)
			    this.index = e.detail.value
				this.num = this.index+1
				console.log("n",this.num)
				uni.request({
					url: `http://10.3.41.200:8888/pronunciation/generateWords/${this.num}`,
					method: 'GET',
					header: {
						"Authorization": this.token
					},
					success: (res) => {
						// console.log(res.data.data.token)
						console.log(res.data)
						// console.log(this.token)
						console.log(res.data.data.info)
						this.word = res.data.data.info.wordsInfo
						this.wordsId = res.data.data.info.wordsId
					}
				})
			},

			// 年级选择
			gradeChange(e) {
				// console.log(e)
				// this.value = e;
				// console.log(this.token)
				console.log('n',this.num)
				
				uni.request({
					url: `http://10.3.41.200:8888/pronunciation/generateWords/${this.num}`,
					method: 'GET',
					header: {
						"Authorization": this.token
					},
					success: (res) => {
						// console.log(res.data.data.token)
						console.log(res.data)
						// console.log(this.token)
						console.log(res.data.data.info)
						this.word = res.data.data.info.wordsInfo
						this.wordsId = res.data.data.info.wordsId
					}
				})


			},

	


			// 长按录音事件
			longpressBtn() {
				if(this.recorderManager == null) {
					this.recorderManager = uni.getRecorderManager()
				}
				const options = {
					duration: this.duration, // 指定录音的时长，单位 ms
					sampleRate: 16000, // 采样率
					numberOfChannels: 1, // 录音通道数
					encodeBitRate: 96000, // 编码码率
					format: 'mp3', // 音频格式，有效值 aac/mp3
					frameSize: 10, // 指定帧大小，单位 KB
				}
				this.recorderManager.start(options);
				// 监听音频开始事件
				this.recorderManager.onStart((res) => {
					console.log("测试",res)
				})
			},
			// 长按松开录音事件
			touchendBtn() {
				this.recorderManager.onStop((res) => {
					// this.tempFilePath = res.tempFilePath
					uni.uploadFile({
						url: 'http://10.3.41.200:8888/pronunciation/control',
						filePath: res.tempFilePath,
						name: 'voiceFile',
						header: {
							"Authorization": this.token
						},
						formData: {
							wordsId: this.wordsId,
							text: this.word
						},
						success: (uploadFileRes) => {
							console.log(uploadFileRes.data);
							this.html = JSON.parse(uploadFileRes.data).data.text
						}
					});
				})
				this.recorderManager.stop()


				
			},


			// 跟读
			read() {
				const innerAudioContext = uni.createInnerAudioContext()
				// alert('3242')
				// console.log(this.word)
				uni.request({
					url: `http://10.3.41.200:8888/pronunciation/followAlong`,
					method: 'POST',
					header: {
						"Authorization": this.token
					},
					data: {
						text: this.word
					},
					success: (res) => {
						innerAudioContext.autoplay = true
						innerAudioContext.src = res.data.data.voice

						// alert(res.data)
						console.log(res.data.data.voice)
					}
				})
				// console.log(this.num)

			},

			// 添加词语
			addWord() {
				uni.request({
					url: 'https://api.uixsj.cn/hitokoto/get?type=social',

					data: {
						text: 'uni.request'
					},
					success: (res) => {
						// console.log(res.data)
						this.word = res.data
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	em {
		color: red;
	}
	.newWord {
		position: absolute;
		width: 100%;
		height: 50%;
		border: 1px solid red;
		background-color: pink;
		overflow: auto;
	}

	.audioResult {
		position: absolute;
		width: 100%;
		top: 50%;
		left: 75%;
		margin-top: 10%;

		button {
			margin-top: 10%;
		}

		select {
			height: 70upx;
		}

		.exportResult {
			position: absolute;
			right: 100%;
			width: 75%;
			height: 100%;
			background-color: skyblue;
		}
	}
	.icon{
		// position: absolute;
		// position: relative;
		position: absolute;
		left: 80upx;
		top: 8upx;
		font-size: 16upx
		
		// bottom: 40upx;
		
	}
	.uni-input{
		position: relative;
		top: 10upx;
		left: 10upx;
		font-size: 30%;
		font: 80;
	}
	.select{
		position: absolute;
		width: 16%;
		height: 18%;
		// background-color: aqua;
		border: 1upx solid white;
		
		border-radius: 15%;
		background-color: rgb(0, 122, 255);
		color: white;
		
	}
	

	// .tabbar{
	// 		position: fixed;
	// 			bottom: 0;
	// 			left: 0;
	// 			width: 100%;

	// }
	// .uni-tabbar__icon {
	// 		height: 20rpx;
	// 		width: 20rpx;
	// 	}
	
</style>
