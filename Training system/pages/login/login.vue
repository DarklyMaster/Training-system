<template>
	<view class="backgroundLogin">
		<view class="container">
			<input v-model="username" placeholder="用户名" />
			<input v-model="password" type="password" placeholder="密码" />
			<button @click="login">登录</button>
			<view class="" @click="register">
				没有账号？点击注册
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username: '',
				password: '',
				fromPage: ''
			};
		},
		onLoad(options) {
			this.fromPage = options.from; // 获取来源页面
		},
		methods: {
			async login() {
				try {
					// 发送登录请求到后端
					const response = await uni.request({
						url: 'http://192.168.1.200:8080/user/login', // 确保此 URL 与后端一致
						method: 'POST',
						header: {
							'Content-Type': 'application/json' // 设置请求头
						},
						data: {
							Username: this.username, 
							Password: this.password
						}
					});

					// console.log(response);
					// 处理响应
					if (response.statusCode === 200) { // 注意：uni.request 返回的是一个数组
						const data = response.data; // 获取响应数据
						if (data.msg === '登录成功') { // 根据后端返回的消息判断成功
							// 保存 token
							uni.setStorageSync('token', data.data); 
							uni.setStorageSync('id',data.id);
							uni.showToast({
								title: '登录成功',
								icon: 'success',
							});
							if (this.fromPage) {
								// 如果有来源页面，返回到该页面
								uni.switchTab({
									url: '/'+this.fromPage
								});
							} else {
								// 如果没有来源页面，跳转到首页
								uni.switchTab({
									url: '/pages/training/training'
								});
							}
						} else {
							uni.showToast({
								title: '登录失败: ' + data.msg,
								icon: 'none'
							});
						}
					} else {
						uni.showToast({
							title: '登录请求失败',
							icon: 'none'
						});
					}
				} catch (error) {
					console.error('登录时出错:', error);
					uni.showToast({
						title: '登录时出错',
						icon: 'none'
					});
				}
			},

			register() {
				uni.navigateTo({
					url: './register',
				});
			}
		}
	};
</script>

<style scoped>
	.backgroundLogin {
		background-image: url('/static/backgroundImg/6b.jpg');
		background-size: cover;
		/* 使背景图片覆盖整个区域 */
		background-position: center;
		/* 背景图片居中 */
		height: 100vh;
		/* 设置高度为视口高度 */
		width: 100%;
		/* 设置宽度为100% */
		display: flex;
		/* 使用 flexbox 布局 */
		justify-content: center;
		/* 水平居中 */
		align-items: center;
		/* 垂直居中 */
		color: #ff6a9a;
		/* 设置文字颜色 */
		position: fixed;
	}

	.container {
		padding: 20px;
		margin-top: 0px;
	}

	.container input {
		height: 30px;
		font-size: 18px;
		margin-bottom: 10px;

	}

	.container button {
		/* font-weight: bold; */
		color: #ff6a9a;
		background-color: rgba(0, 0, 0, 0);
	}
</style>