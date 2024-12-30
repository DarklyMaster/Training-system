<template>
	<view class="backgroundMy">
		
	
	<view class="container">
		<view class="header">
			<view class="avatar-container">
				<image :src="avatar"  class="avatar" mode="aspectFill" @click="chooseImage" />
			</view>
			<view class="information">

				<span class="nameCss" v-if="userName==''" @click="chooseImage">未登录</span>
				<span v-else>
					<p class="nameCss">{{userName}}</p>
					<p>段位：{{nb}}</p>
				</span>
			</view>
			<view class="myButton" @click="urlMyInformation">
				<span class="icon_inf"></span>
			</view>
		</view>
		<view class="kong">
			
		</view>
		<button v-if="hasLoggedIn" @click="login">登录</button>
		<button v-else @click="remove">注销</button>
		<uni-popup ref="popup" type="center" background-color="#fff">
			<view class="popup-content">
				<view class="loadImg">
					<image src="/static/loadImg/ABC.jpg" mode=""
						@click="loadImage('/static/loadImg/ABC.jpg')"></image>
					<image src="/static/loadImg/AEA.jpg" mode=""
						@click="loadImage('/static/loadImg/AEA.jpg')"></image>
					<image src="/static/loadImg/AIC.jpg" mode=""
						@click="loadImage('/static/loadImg/AIC.jpg')"></image>
					<image src="/static/loadImg/AHL.jpg" mode=""
						@click="loadImage('/static/loadImg/AHL.jpg')"></image>
					<image src="/static/loadImg/ACT.jpg" mode=""
						@click="loadImage('/static/loadImg/ACT.jpg')"></image>
					<image src="/static/loadImg/AIC.jpg" mode=""
						@click="loadImage('/static/loadImg/AIC.jpg')"></image>

				</view>

			</view>
		</uni-popup>
	</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				avatar: '/static/loadImg/xuesheng.jpg',
				userId: '',
				hasLoggedIn: true,
				userName: '',
				nb:'青铜'
			};
		},
		onShow() {
			this.userId = uni.getStorageSync('id')
			if (this.userId && this.hasLoggedIn) {
				this.getloadImage();
				this.hasLoggedIn = false
			}
		},
		methods: {
			chooseImage() {
				if (this.hasLoggedIn) {
					this.login()
					return
				}
				this.$refs.popup.open()
			},
			loadImage(i) {
				this.avatar = i
				this.uploadImage()
				this.$refs.popup.close()
			},
			uploadImage() {
				uni.request({
					url: 'http://192.168.1.200:8080/user/' + this.userId, // 替换为你的获取头像接口
					method: 'PUT',
					header: {
						'Content-Type': 'application/json' // 设置请求头
					},
					data: {
						Avatar: this.avatar, // 确保字段名与后端一致
					}
				});
			},
			getloadImage() {
				uni.request({
					url: 'http://192.168.1.200:8080/user/' + this.userId, // 替换为你的获取头像接口
					method: 'GET',
					success: (res) => {
						// console.log(res);
						if (res.data.data.avatar != '') {
							this.avatar=res.data.data.avatar
						}
						this.userName = res.data.data.username
					}
				});
			},
			urlMyInformation(){
				if(this.userId){
					uni.navigateTo({
						url:'./myInformation'
					})
					return
				}
				uni.navigateTo({
					url:'/pages/login/login'
				})
			},
			login() {
				const pages = getCurrentPages();
				const currentPage = pages[pages.length - 1]; // 获取当前页面实例
				const route = currentPage.route; // 获取当前页面的路径
				uni.navigateTo({
					url: '/pages/login/login?from=' + this.route
				});
			},
			remove() {
				uni.clearStorageSync();
				uni.reLaunch({
					url: '/pages/training/training'
				})

			},
		}
	};
</script>


<style>
	.backgroundMy::before {
		content: "";
		            position: absolute;
		            top: 0;
		            left: 0;
		            right: 0;
		            bottom: 0;
		            background-image: url('/static/backgroundImg/bg1.png'); /* 替换为您的图片路径 */
		            background-size: cover; /* 使背景图片覆盖整个页面 */
		            background-position: center; /* 使背景图片居中 */
		            background-repeat: no-repeat; /* 不重复背景图片 */
		            opacity: 0.8; /* 设置透明度为 0.5 */
		            z-index: -1; /* 确保伪元素在内容下面 */
	}
	.container {
		/* display: flex; */
		flex-direction: column;
		align-items: center;
		padding: 20px;
	}

	.avatar-container {
		width: 30%;
		display: flex;
		justify-content: center;
		margin-bottom: 20px;
	}

	.avatar {
		width: 100px;
		/* 头像宽度 */
		height: 100px;
		/* 头像高度 */
		border-radius: 50%;
		/* 圆形头像 */
	}

	.header {
		display: flex;
		width: 100%;
		box-sizing: border-box;
		
	}


	.information {
		width: 70%;
		
	}
	.information p{
		margin: 10px
	}
	.nameCss{
		font-size: 24px;
	}
	.myButton{
		height: 100px;
		margin: 10px;
		
	}
	.icon_inf {
		height: 20px;
		width: 20px;
		position: absolute;
		background-position: -20px 0px;
		float: right;
		top: 50px;
		transform: rotate(180deg);
	}
	.popup-content {
		@include flex;
		align-items: center;
		justify-content: center;
		padding: 5px;
		height: 65vh;
		width: 340px;
	}

	.loadImg {
		flex-direction: row;
		box-sizing: border-box;
		flex-wrap: wrap;
		display: flex;
	}

	.loadImg image {
		flex-basis: calc((100% - 18px)/3);
		height: 80px;
		margin: 3px;
	}
	.kong{
		width: 100%;
		height: 55vh;
	}
	/* .fenlei{
		background-color: #637a8c;
		flex-wrap: wrap;
		display: flex;
		padding: 2.5px;
		
		
	}
	
	.fenlei view{
		flex-basis: calc((100% - 15px)/3);
		height: 120px;
		border-radius: 15px;
		margin: 2.5px;
		background-color: rgba(150, 21, 133, 0.5);
		
	} */
</style>