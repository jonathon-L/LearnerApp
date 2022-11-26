<template>
	<view class="homework003">


		<view class="date-start">
			<text>作业开始时间</text>
			<view class="example-body">
				<uni-datetime-picker type="datetime" v-model="datetimeStart" @change="changeLog" />
			</view>
		</view>

		<view class="date-end">
			<text>作业结束时间</text>
			<view class="example-body">
				<uni-datetime-picker type="datetime" v-model="datetimeEnd" @change="changeLog" />
			</view>
		</view>

		<!-- <view class="date">
			<text>作业时间范围</text>
			<view class="example-body">
				<uni-datetime-picker v-model="datetimerange" type="datetimerange" rangeSeparator="至" />
			</view>
		</view> -->
		<view class="container">
			<countdown-timer ref="countdown" :time="time" @finish="onFinish" autoStart>
				<template v-slot="{day, hour, minute, second, remain, time}">
					<!-- 基本样式 -->
					<view class="case">
						<view>{{day}}天{{hour}}时{{minute}}分{{second}}秒</view>
					</view>
				</template>

			</countdown-timer>

		</view>



		<view class="btn">
			<button type="default" plain="true" @click="onset">开始计时</button>
		</view>

		<view class="btn">
			<button type="default" plain="true" @click="submit">作业完成</button>
		</view>


	</view>
</template>

<script>
	export default {
		data() {
			return {
				datetimeStart: '',
				datetimeEnd: '',
				datetimerange: [],
				start: Date.now() - 1000000000,
				end: Date.now() + 1000000000,
				// time: new Date('2020/04/24 01:00:00').getTime() - new Date('2020/04/24 00:20:00').getTime()
				time: '',

				timer: '',
				innerAudioContext: uni.createInnerAudioContext(),
				token: ''
			}
		},


		watch: {

			datetimeStart(newval) {
				console.log('开始:', this.datetimeStart);

			},
			datetimeEnd(newval) {
				console.log('结束:', this.datetimeEnd);
				console.log(this.datetimeStart)
			},



			datetimerange(newval) {
				console.log('时间范围:', this.datetimerange);
			},


		},

		onLoad() {
			this.token = uni.getStorageSync("token")
		},

		methods: {

			changeLog(e) {
				console.log('----change事件:', e);
			},
			
			// 时间格式化
			formatDate(time){
			    var date = new Date(time);
			 
			    var year = date.getFullYear(),
			        month = date.getMonth() + 1,//月份是从0开始的
			        day = date.getDate(),
			        hour = date.getHours(),
			        min = date.getMinutes(),
			        sec = date.getSeconds();
			    var newTime = year + '-' +
			        month + '-' +
			        day + ' ' +
			        hour + ':' +
			        min + ':' +
			        sec;
			    return newTime;
			}, 
			submit() {
				uni.request({
					url: 'http://10.3.41.200:8888/homework/finish2',
					method: 'POST',
					header: {
						"content-type": "application/json",
						"Authorization": this.token
					},
					data: {
						beginTime: this.formatDate(this.datetimeStart),
						endTime: this.formatDate(this.datetimeEnd),
					 finishTime: this.formatDate(new Date())
					},
					success: (res) => {
						if (res.data.success == true) {
							uni.showToast({
								title: "上传成功"
							})
							this.datetimeEnd = ''
							this.datetimeStart = ''

							clearTimeout(this.timer)
							this.time = ''
						} else {
							uni.showToast({
								title: res.data.message
							})

						}

					},
				})


			},



			onset() {
				this.time = new Date(this.datetimeEnd).getTime() - new Date(this.datetimeStart).getTime()

				if (this.time) {


					// const innerAudioContext = uni.createInnerAudioContext();
					this.timer = setTimeout(() => {
						this.innerAudioContext.autoplay = true;
						// this.innerAudioContext.src = 'https://bjetxgzv.cdn.bspapp.com/VKCEYUGU-hello-uniapp/2cc220e0-c27a-11ea-9dfb-6da8e309e0d8.mp3';
						this.innerAudioContext.src = '/static/remind.mp3'
						this.datetimeEnd = ''
						this.datetimeStart = ''

						setInterval(() => {
							this.innerAudioContext.stop()
						}, 3000)
					}, this.time)


					this.innerAudioContext.onPlay(() => {
						console.log('开始播放');
					});
					this.innerAudioContext.onError((res) => {
						console.log(res.errMsg);
						console.log(res.errCode);
					});

				} else {
					uni.showToast({
						icon: "none",
						title: '请输入时间'
					})
					this.time = ''
				}


			},

			onFinish() {
				uni.showToast({
					icon: 'none',
					title: '时间到了'
				})
				this.submit()
			},

		}
	}
</script>

<style lang="scss">
	.homework003 {
		margin-top: 10%;
	}

	.example-body {
		background-color: #fff;
		padding: 10px;
	}

	.btn {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-top: 10%;

		button {
			width: 50%;
		}
	}

	text {
		color: rgba(128, 128, 128, 1);
		// font-size: 20%;
		font-weight: 500;
		margin-left: 5%;
	}

	.container {

		padding: 20upx;
	}

	.case {
		display: flex;
		margin: 20upx;
		justify-content: center;
	}

	.case>.title {
		margin-right: 10upx;
	}

	.custom {
		display: flex;
	}

	.custom :nth-child(odd) {
		background-color: red;
		padding: 2upx 4upx;
		color: white;
		border-radius: 5upx;
		text-align: center;
	}

	.custom :nth-child(even) {
		padding: 0 5upx;
	}

	.reset-btn {
		margin: 20upx 10upx;
		padding: 20upx;
		text-align: center;
		background-color: red;
		border-radius: 10upx;
		color: white;
	}
</style>
