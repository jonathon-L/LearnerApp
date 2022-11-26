<template>
	
	<view>
		<view class="user003">
			<view class="userName">
				<input type="text" placeholder="请输入手机号" @input="messagesearch" :value="phoneNumber" v-model="phoneNumber">
			</view>
			
			<view class="userCode">
				<view class="getCode" @click="getCode">{{getCodeText}}</view>
				<input type="text" placeholder="请输入验证码"  v-model="code" :value="code">
			</view>
			
			<view class="userPassword">
				<input type="password" placeholder="请输入密码" v-model="password">
			</view>
			
			<view class="btn" @click="regist">立即注册</view>
			
		</view> 
 
 
	</view>

	




</template>

 <script>
	 export default {
		 data(){
			 return{
				 getCodeText:'获取验证码',
				 phoneNumber:'',
				 code:'',
				 password:'',

			 }
		 },
		 
		 methods: {
			messagesearch:function(event){
			                // 这里动态获取VALUE 值
			                // console.log(event.target.value)
			                this.phoneNumber = event.target.value;
			        },
		 	getCode() {
				if (!(/^1(3|4|5|6|7|8|9)\d{9}$/.test(this.phoneNumber))) { //校验手机号码是否有误
					uni.showToast({
						title: '请填写正确手机号码',
						icon: "none"
					});
					return false;
				}
				this.getCodeText = "发送中..." //发送验证码
				this.getCodeBtnColor = "rgba(255,255,255,0.5)" //追加样式，修改颜色
				//示例用定时器模拟请求效果
				//setTimeout(()用于在指定的毫秒数后调用函数或计算表达式
				setTimeout(() => {
					this.setTimer(); //调用定时器方法
				}, 1000)
				// console.log('test')
				uni.request({
					url:`http://10.3.41.200:8888/user/getCode/${this.phoneNumber}`,
					method:'GET',
					success:(res)=>{
						if(res.data.success == true){
							console.log(res.data)
							uni.showToast({
								title:res.data.message
							})
						}else{
							uni.showToast({
								title:res.data.message
							})
						}
					}
				})
				
				
		 	},
			setTimer() {
				let holdTime = 60; //定义变量并赋值
				this.getCodeText = "重新获取(60)"
				this.Timer = setInterval(() => {
					if (holdTime <= 0) {
						this.getCodeisWaiting = false;
						this.getCodeBtnColor = "#ffffff";
						this.getCodeText = "获取验证码"
						clearInterval(this.Timer); //清除该函数
						return; //返回前面
					}
					this.getCodeText = "重新获取(" + holdTime + ")"
					holdTime--;
				}, 1000)
			},
			
			
			
			regist(){
				
					if(this.phoneNumber && this.password){
						
						uni.request({
							url:'http://10.3.41.200:8888/user/registered',
							method:'POST',
							header:{
								"content-type": "application/json",
							},
							data:{
								
								// username:JSON.stringify(this.phoneNumber),
								// password:JSON.stringify(this.password),
								// code:JSON.stringify(this.code)
								phone:this.phoneNumber,
								password:this.password,
								code:this.code
							},
							success: (res) => {
								if(res.data.success == true){
									uni.setStorageSync('token', res.data.data.token)
									console.log(res.data)
									uni.showToast({
										title:res.data.message,
										//注册成功后进入到登录页面
										
									})
									setTimeout(() => {
										
										uni.navigateTo({
											url:"/pages/login/login",
											
										})
										
									}, 2000);
									
								}else{
									uni.showToast({
										title:res.data.message,
										
									})
									console.log(res.data)
								}
								
							}
							
						})
						
						}
					
				
				}
			}
		 }
 </script>

 -->
 <style lang="scss">
	 page{
		 background-color: #3F536E;
	 }
	 .user003{
		 margin-top: 40%;
	 }
	 input{
		 width: 70%;
		 height: 90upx;
		 background-color: #fff;
		 font-weight: 200;
		 border-radius: 45upx;
	 }
	 .userName{
		 position: relative;
		 display: flex;
		 justify-content: center;
		 align-items: center;
	 }
	 
	 .userCode{
		 position: relative;
		 display: flex;
		 justify-content: center;
		 align-items: center;
		 margin-top: 15px;
		 .getCode{
			 position: absolute;
			 color: #000000;
			 right: 16%;
			line-height: 90upx;
			z-index: 9;
		 }
	 }
	 .userPassword{
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

// </style>