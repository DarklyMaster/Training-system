<template>
	<view class="">
		<hr class="hr" style=' height:1px; border:none;'/>
		<view class="input">
			<form class="" action="#">
				<span class="icon_search"></span>
				<input type="search" placeholder="搜索">
			</form>
		</view>
		<view class="home">
			<!-- 轮播图 -->
			<swiper class="swiper" indicator-dots autoplay circular="true" interval="3000"
				indicator-active-color="#fecccb" touchmove="passive">
				<block v-for="(item, index) in banners" :key="index">
					<swiper-item>
						<image :src="item.img" />
					</swiper-item>
				</block>
			</swiper>
			
			<!-- <view class="neirong">
				<view>
					asd
				</view>
				<view class="neirong2">
					asd
				</view>
			</view> -->
			
			</view>
			<span class="text1">每日新闻</span>
			<view class="uni-list">
				<view class="uni-list-cell" hover-class="uni-list-cell-hover" v-for="(news,index) in list" :key="index" @click="openinfo( { newsid:news.post_id } )">
					<view  class="uni-media-list">					
						<image class="uni-media-list-logo" :src="news.cover"></image>
						<view class="uni-media-list-body">
							<view class="uni-media-list-text-top">{{news.title}}</view>
							<view class="uni-media-list-text-bottom uni-ellipsis">{{news.created_at}}</view>
						</view>
					</view>
				</view>
			</view>
			<!-- <text class="text1">复习内容分类</text>
			<view class="fenlei">
				
				<view>
					asd
				</view>
				<view>
					asd
				</view>
				<view>
					asd
				</view>
				<view>
					asd
				</view>
			</view> -->
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				banners: [] ,// 轮播图数据将被存储在这里
				list:[]	
			};
		},
		onLoad() {
			this.fetchBanners();
			this.newList();
		},
		// onReady(){
		// 	const token = uni.getStorageSync('token');
		// 	    if (token) {
		// 	        // 如果 token 存在，进行自动登录操作
		// 	        this.autoLogin(token);
		// 	    } else {
		// 	        this.redirectToLogin();
		// 	    }
		// },
		methods: {
			async fetchBanners() {
				try {
					// 调用后端API获取轮播图数据
					var res = await uni.request({
						url: 'http://127.0.0.1:8080/swiper', // 替换API地址
						method: 'GET',
					});
					// console.log(res);
					if (res.statusCode == 200) {
						this.banners = res.data.data; // 后端返回的数据格式正确
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},
			async newList() {
				try {
					// 调用后端API获取轮播图数据
					await uni.request({
						url: 'https://unidemo.dcloud.net.cn/api/news', // 替换API地址
						method: 'GET',
						data:{},
						success: res =>{
							// console.log(res);
							if(res.statusCode == 200) {
								this.list = res.data; // 后端返回的数据格式正确
							} else {
								console.error('无法获取数据');
							}	
						
						},
						
					});
					
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},
			 autoLogin(token) {
			        // 调用你的登录接口
			        uni.request({
			            url: 'http://127.0.0.1:8080/user/login', // 替换为你的 API 地址
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
			openinfo(e){
				// console.log(e);
				var newsid=e.newsid;
				// console.log(newsid);
				uni.navigateTo({
					url:'./news?newsid='+newsid,
				});
			}
		}
	};
</script>

<style>
	.hr{
		background-color: rgba(254, 156, 156, 0.5);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
	}
	.home{
		margin: 5px;
	}
	

	.input {
		background-color: rgba(254, 156, 156, 0.5);
		height: 30px;
		padding: 7px 3px;
		/* border-top: 1px solid #000000; */
		box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
	}

	.input input {
		width: 100%;
		height: 30px;
		border-radius: 15px;
		padding-left: 30px;
		background-color: rgba(255, 255, 255, 0.8);
		box-sizing:border-box;
	}

	.icon_search {
		height: 20px;
		width: 20px;
		position: absolute;
		background-position: -60px -109px;
		top: 13px;
		left: 10px;
	}
	.swiper {
		width: 100%;
		height: 30vh;
		overflow: hidden;
		border-radius: 15px;
		transform: translateZ(0);
		
	}
	
	.swiper image {
		width: 100%;
		height: 100%;
		
	}
	/* .neirong {
		width: 100%;
		height: 120px;
		margin: 5px 0px;
		box-sizing: border-box;
		display: flex;
		flex-direction: row;
	}
	.neirong view{
		width: 50%;
		height: 120px;
		background-color: rgba(150, 21, 133, 0.5);
		border-radius: 15px;
		box-sizing:border-box;
	}
	.neirong2 {
		margin: 0 0 0 5px;
	} */
	.text1{
		text-decoration: underline;
		font-size: 20px;
		/* display: contents; */
		margin-left: 10px;
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
	.uni-media-list-body{
		height: auto;
	}
	.uni-media-list-text-top{
	line-height: 1.6em;	
	}
	.uni-media-list image{
		overflow: hidden;
		border-radius: 5px;
		transform: translateZ(0);
		
	}
	
</style>