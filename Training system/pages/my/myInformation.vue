<template>
	<view class="myCss">
		<view class="myInformation">
			<view class="infButton">
				<view class="informationText">头像</view>
				<view class="myAvatar-container">
					<image :src="avatar" class="myAvatar" mode="aspectFill" />
				</view>
				<view class="icon_up"></view>
			</view>
			<view class="infButton">
				<view class="informationText">昵称</view>
				<view class="informationGet">
					{{userName}}
				</view>
				<view class="icon_up"></view>
				
			</view>
			<view class="infButton">
				<view class="informationText">学号</view>
				<view class="informationGet">
					{{userId}}
				</view>
			</view>
			<view class="infButton">
				<view class="informationText">生日</view>
				<view class="informationGet">
					{{userBirthday}}
				</view>
				<view class="icon_up"></view>
			</view>
		</view>
		<uni-popup ref="popup" background-color="#fff" >
			<view class="popup-content" >
				<text class="text">请选择考试范围</text></view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				avatar: '/static/loadImg/xuesheng.jpg',
				userName:'',
				userId:'',
				userBirthday: '2000年1月1日',
				hasLoggedIn:true
			}
		},
		onShow() {
			this.userId = uni.getStorageSync('id')
			if (this.userId && this.hasLoggedIn) {
				this.getloadImage();
				this.hasLoggedIn = false
			}
		},
		methods: {
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
		}
	}
</script>

<style>
	.myCss {
		position: fixed;
		width: 100%;
		height: 100vh;
		background-color: #eeeff0;
	}

	.myInformation {
		width: 100%;
		margin-top: 5px;
		/* align-items: center; */

		box-sizing: border-box;
	}

	.informationText {
		width: 50%;
		align-items: right;
	}

	.infButton {
		box-sizing: border-box;
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		height: 50px;
		background-color: #fff;
		border-bottom: 1px solid #eeeff0;
		padding: 0 20px;
		line-height: 50px;
		/* position: relative; */
	}

	.myAvatar-container {
		width: 40%;
		display: flex;
		justify-content: flex-end;
	}
	.informationGet{
		width: 40%;
		display: flex;
		justify-content: flex-end;
	}
	.myAvatar {
		width: 50px;
		height: 50px;
		border-radius: 50%;

	}

	.icon_up {
		height: 20px;
		width: 20px;
		background-position: -180px -50px;
	}
</style>