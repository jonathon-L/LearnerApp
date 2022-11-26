<template>
	<view>
		<view class="uni-title uni-common-pl">请选择需要查看的日期</view>
		<view class="uni-list">
			<view class="uni-list-cell">
				<view class="uni-list-cell-left">
					当前选择
				</view>
				<view class="uni-list-cell-db">
					<picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange">
						<view class="uni-input">{{date}}</view>
					</picker>
				</view>
			</view>
		</view>
		<view class="charts-box" v-if="type == '0' || type == '1'">
			<qiun-data-charts type="area" :opts="opts" :chartData="chartData" />
		</view>
		<view class="charts-box" v-if="type == '2'">
			<qiun-data-charts type="area" :opts="opts2" :chartData="chartData" />
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			const currentDate = this.getDate({
				format: true
			})
			return {
				type: '',
				typeUri: ['ray', 'attitude'],
				token: '',
				title: 'picker',
				date: currentDate,
				chartData: {},
				analyseData: [],
				opts2: {
					color: ["#1890FF", "#91CB74", "#FAC858", "#EE6666", "#73C0DE", "#3CA272", "#FC8452", "#9A60B4",
						"#ea7ccc"
					],
					padding: [15, 15, 0, 5],
					legend: {},
					xAxis: {
						disableGrid: true,
						rotateLabel: true,
						rotateAngle: 20
					},
					yAxis: {
						data: [{
							unit: '%',
							min: 0,
							max: 100
						}],
					},
					extra: {
						column: {
							type: "meter",
							width: 30,
							activeBgColor: "#000000",
							activeBgOpacity: 0.08,
							meterBorder: 3,
							meterFillColor: "#EBFBD6"
						}
					}
				},
				opts: {
					color: ["#1890FF"],
					padding: [15, 15, 0, 15],
					legend: {},
					xAxis: {
						disableGrid: true
					},
					yAxis: {
						gridType: "dash",
						dashLength: 2,
						data: [{
							unit: '%',
							max: 100,
							min: 0
						}],
						
					},
					extra: {
						area: {
							type: "straight",
							opacity: 0.2,
							addLine: true,
							width: 2,
							gradient: false
						}
					}
				}
			}
		},
		computed: {
			startDate() {
				return this.getDate('start');
			},
			endDate() {
				return this.getDate('end');
			}
		},
		onLoad(option) {
			console.log("路由传参", option);
			this.type = option.type
			this.token = uni.getStorageSync("token")
			this.getAnalyseData();
		},
		watch: {
			date() {
				this.getAnalyseData()
			}
		},
		methods: {
			getAnalyseData() {
				let tDate = this.date == null ? new Date() : this.date;
				if (this.type == '0' || this.type == '1') {
					uni.request({
						url: `http://10.3.41.200:8888/analysis/${this.typeUri[this.type * 1]}/data`,
						method: 'GET',
						header: {
							"Authorization": this.token
						},
						data: {
							date: this.formatDate(tDate, 2)
						},
						success: (res) => {
							this.analyseData = res.data.data.items
							this.getServerData()
						}
					})
				} else if (this.type == '2') {
					// 作业完成分析
					uni.request({
						url: `http://10.3.41.200:8888/analysis/completionTime`,
						method: 'GET',
						header: {
							"Authorization": this.token
						},
						data: {
							date: this.formatDate(tDate, 2)
						},
						success: (res) => {
							this.analyseData = res.data.data.items
							this.getServerData2()
						}
					})
				}

			},
			// 时间格式化
			formatDate(time, type) {
				let date = new Date(time);

				let year = date.getFullYear(),
					month = date.getMonth() + 1, //月份是从0开始的
					day = date.getDate(),
					hour = date.getHours(),
					min = date.getMinutes(),
					sec = date.getSeconds();
				let newTime;
				if (type == 1) {
					newTime = hour + ':' + min + ':' + sec;
				} else if (type == 2){
					newTime = year + '-' +
						month + '-' +
						day + ' ' +
						hour + ':' +
						min + ':' +
						sec;
				} else if (type == 3) {
					newTime = month + '-' +
						day + ' ' +
						hour + ':' +
						min;
				}

				return newTime;
			},
			// 计算两时间间隔
			differMinuteTime(startDate, endDate) { // 一分钟等于60000毫秒
				return Math.floor((new Date(endDate).getTime() - new Date(startDate).getTime()) / 60000)
			},
			getServerData2() {
				let times = [];
				let maxJobTime;
				let jobTime;
				let values = []
				
				for(let i = 0; i < this.analyseData.length; i++) {
					times.push(this.formatDate(this.analyseData[i].beginTime,3));
					maxJobTime = this.differMinuteTime(this.analyseData[i].beginTime,this.analyseData[i].endTime);
					if(this.analyseData[i].complete == false) {
						jobTime = this.differMinuteTime(this.analyseData[i].beginTime,this.analyseData[i].endTime);
					} else {
						jobTime = this.differMinuteTime(this.analyseData[i].beginTime,this.analyseData[i].finishTime);
					}
					values.push(parseInt((jobTime / maxJobTime) * 100));
				}
				
				let res = {
					categories: times,
					series: [{
							name: "作业完成占作业总时间",
							data: values
						},
					]
				};
				this.chartData = JSON.parse(JSON.stringify(res));
			},
			getServerData() {
				let time = [];
				let values = [];

				for (let i = 0; i < this.analyseData.length; i++) {
					values.push(this.analyseData[i].value)
					time.push(this.formatDate(this.analyseData[i].time, 1))
				}
					
				let text = '';
				if(this.type == '0') {
					text = '光线数据分析'
				} else if(this.type == '1') {
					text = '阅读及书写姿态分析'
				}

				let res = {
					categories: time,
					series: [{
						name: text,
						data: values
					}, ]
				};
				this.chartData = JSON.parse(JSON.stringify(res));

			},
			bindDateChange: function(e) {
				this.date = e.detail.value
			},
			getDate(type) {
				const date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();

				if (type === 'start') {
					year = year - 60;
				} else if (type === 'end') {
					year = year + 2;
				}
				month = month > 9 ? month : '0' + month;
				day = day > 9 ? day : '0' + day;
				return `${year}-${month}-${day}`;
			}
		}
	}
</script>

<style>
	.charts-box {
		width: 100%;
		height: 300px;
	}

	.uni-title {
		font-size: 30rpx;
		font-weight: 500;
		padding: 20rpx 0;
		line-height: 1.5;
	}

	,
	.uni-common-pl {
		padding-left: 25%;
		background-color: #F5F5F5;
	}

	,
	.uni-list {
		background-color: #D3D3D3;
		position: relative;
		width: 100%;
		display: flex;
		flex-direction: column;
	}

	,
	.uni-list-cell {
		position: relative;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}

	,
	.uni-list-cell-left {
		white-space: nowrap;
		font-size: 28rpx;
		padding: 0 30rpx;
	}

	,
	.uni-list-cell-db,
	.uni-list-cell-right {
		flex: 1;
	}

	,
	.uni-input-group {
		position: relative;
		padding: 0;
		border: 0;
		background-color: #fff;
	}
</style>
