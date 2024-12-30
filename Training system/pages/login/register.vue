<template>
	<view class="registerCss">
		<view class="inputText">
			账号<input v-model="username" placeholder="请输入用户名" />
		</view>
		<view class="inputText">
			密码<input v-model="password" type="password" placeholder="请输入密码" />
		</view>
		<view class="inputText">
			再次输入密码<input v-model="password2" type="password" placeholder="请输入密码" />
		</view>
		<button @click="register">注册</button>
		<view class="text" @click="login">
			已有账号？点击登录
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username: '',
				password: '',
				password2: '',
			};
		},
		methods: {
			async register() {
				// 验证用户名和密码
				if (this.username.length < 1) {
					uni.showToast({
						title: '用户名不能为空',
						icon: 'none'
					});
					return;
				}
				if (this.password.length < 6) {
					uni.showToast({
						title: '密码长度不能少于6位',
						icon: 'none'
					});
					return;
				}
				if (this.password2 != this.password) {
					uni.showToast({
						title: '密码不一致',
						icon: 'none'
					});
					return;

				}
				// 调用 API 注册
				const response = await this.apiRegister(this.username, this.password);
				if (response.success) {
					uni.showToast({
						title: '注册成功',
						icon: 'success'
					});
					// 跳转到登录页面
					uni.navigateTo({
						url: '/pages/login/login'
					});
				} else {
					uni.showToast({
						title: '注册失败',
						icon: 'none'
					});
				}

			},

			async apiRegister(username, password) {
				return new Promise((resolve) => {
					uni.request({
						url: 'http://192.168.1.200:8080/user', // 确保此 URL 与后端一致
						method: 'POST',
						header: {
							'Content-Type': 'application/json' // 设置请求头
						},
						data: {
							Username: username,
							Password: password,
						},
						success: (res) => {
							console.log(res);
							// 根据后端返回的结果决定成功或失败
							if (res.statusCode === 201) {
								resolve({
									success: true
								});
							} else {
								resolve({
									success: false
								});
							}
						},
						fail: () => {
							resolve({
								success: false
							});
						}
					});
				});
			},
			login() {
				uni.navigateTo({
					url: './login',
				});
			}
		}
	};
</script>

<style scoped>
	.registerCss {
		position: fixed;
		margin: 0px;
		background-color: #eeeff0;
		 flex-direction: column;
		width:100% ;
		height: 100vh;
	}

	.registerCss input {
		height: 30px;
		margin: 0 0 0 20px;
		flex: 1;
		box-sizing: border-box;
	}

	.registerCss button {
		/* font-weight: bold; */
		color:#F8F8F8 ;
		background-color: #ff6a9a;
		border-radius: 50px;
		margin-top:10px;
	}

	.inputText {
		box-sizing: border-box;
		display: flex;
		width: 100%;
		flex-direction: row;
		/* margin-bottom: 10px; */
		border-bottom: 1px solid #eeeff0;
		background-color: #fff;
		line-height: 30px;
		padding: 10px;
	}
	.text{
		float: right;
		margin: 0 10px 0 0;
	}
</style>