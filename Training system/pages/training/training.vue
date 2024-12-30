<template>
	<view class="backgroundTraining">
		<view class="training">
			<button class="trainingDaily" @click="dailyLraining()"><span class="tra_day"></span><view class="day">
				每日答题<p>太阳每天都是新的</p>
			</view></button>
			<view class="sequenceAndAmend">
				<button  @click="practise()"><span class="tra_pra"></span>专项答题 <p>术业有专攻，道义有经论</p></button>
				<button  @click="amend()"><span class="tra_ame"></span>错题集<p>查漏补缺，温故知新</p></button>
			</view>
			
		</view>
		<button class="upTest" @click="up">上传题目</button>
		<view class="">
			
		</view>
			
			
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userId:1
			}
		},
		onReady(){
			const token = uni.getStorageSync('token');
			    if (token) {
			        // 如果 token 存在，进行自动登录操作
			        this.autoLogin(token);
			    } else {
			        this.redirectToLogin();
			    }
		},
		methods: {
			dailyLraining(){
				uni.navigateTo({
					url:'../training/dailyTraining',
				})
			},
			practise(){
				uni.navigateTo({
					url:'/pages/training/sequenceTraining',
				})
			},
			amend(){
				uni.navigateTo({
					url:'../training/amend'
				})
			},
			up(){
				uni.navigateTo({
					url:'/pages/training/upTraining'
				})
			},
			autoLogin(token) {
			       // 调用你的登录接口
			       uni.request({
			           url: 'http://192.168.1.200:8080/user/login', // 替换为你的 API 地址
			           method: 'POST',
			           header: {
			           	'Content-Type': 'application/json' // 设置请求头
			           },
			           data: {
			           	Token: token, // 确保字段名与后端一致
			           
			           },
			
			           success: (res) => {
			               if (res.statusCode === 200) {
			                   // 登录成功，处理后续逻辑
			                   console.log('自动登录成功', res.data);
			                   // 可以跳转到首页或其他页面
			                   uni.switchTab({ url: '/pages/index/index' });
			               } else {
			                   // 登录失败，清除 token 并引导用户登录
			                   uni.clearStorageSync();
			                   this.redirectToLogin();
			               }
			           },
			           fail: (err) => {
			               console.error('请求失败', err);
			               // 请求失败，清除 token 并引导用户登录
			               uni.clearStorageSync();
			               this.redirectToLogin();
			           }
			       });
			   },
			   
			   redirectToLogin() {
			       // 跳转到登录页面
			       uni.navigateTo({
			           url: '/pages/login/login'
			       });
			   },
		}
	}
</script>

<style>

.training{
	box-sizing: border-box;
	display: flex;
	width: 100%;
}
.trainingDaily{
	width: 28%;
	height: 210px;
	margin: 10px;
}
.training p{
	font-size: 14px;
	margin-top: 10px;
	color: #7f7f7f;
}
.sequenceAndAmend{
	width: 68%;
	
}
.sequenceAndAmend button{
	/* flex: 1; */
	height: 100px;
	margin: 10px 10px 10px 0px;
	
}
.upTest{
	margin: 0 10px 10px 10px;
}
.backgroundTraining::before {
    content: "";
                position: absolute;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
                background-image: url('/static/backgroundImg/f4a.jpg'); /* 替换为您的图片路径 */
                background-size: cover; /* 使背景图片覆盖整个页面 */
                background-position: center; /* 使背景图片居中 */
                background-repeat: no-repeat; /* 不重复背景图片 */
                opacity: 0.9; /* 设置透明度为 0.5 */
                z-index: -1; /* 确保伪元素在内容下面 */
}
.day{
	margin: 65px 0 0 0;
}
.tra_day {
		height: 45px;
		width: 45px;
		position: absolute;
		background-position: -55px -36px;
		top: 14px;
		left: 30px;
	}
	.tra_pra{
		height: 45px;
		width: 45px;
		position: absolute;
		background-position: -179px -36px;
		top: 8px;
		left: 30px;
	}
	.tra_ame{
		height: 45px;
		width: 45px;
		position: absolute;
		background-position: -300px -36px;
		top: 8px;
		left: 30px;
	}
</style>
