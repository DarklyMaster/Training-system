<template>
	<view class="contain">
		<view class="category-title">
			请选择分类
		</view>
		<uni-data-checkbox mode="list" 
			v-model="categoryIds" 
			multiple :map="{
                text:'sjname',
                value:'id',
			}" 
			:localdata="categoryList"
		></uni-data-checkbox>
		<view class="select-all">
			<uni-data-checkbox 
				v-model="selectAll" 
				@change="selectAllHandler" 
				multiple
				:localdata="[{value:1,text:'全选'}]"
			></uni-data-checkbox>
		</view>
		<view class="btn-panel">
			<button type="primary" @click="createExam()" >开始考试</button>
		</view>
		<view class="tips">选择考试范围</view>
		<view>
			<!-- 普通弹窗 -->
			<uni-popup ref="popup" background-color="#fff" >
				<view class="popup-content" >
					<text class="text">请选择考试范围</text></view>
			</uni-popup>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				categoryList: [], // 数据将被存储在这里
				categoryIds: [],
				selectAll: "",
			};
		},
		onLoad() {
			this.fetchBanners();
		},
		methods: {
			async fetchBanners() {
				try {
					// 调用后端API获取数据
					const res = await uni.request({
						url: 'http://192.168.1.200:8080/Zbdans',
						method: 'GET'
					});
					// console.log(res);
					if (res.statusCode == 200) {
						this.categoryList = res.data.data; // 后端返回的数据格式正确
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}

			},
			selectAllHandler(e) {
				if (e.detail.value.length===0) {
					this.categoryIds = [];
				} else {
					this.categoryIds = this.categoryList.map((item) => {
						return item.id
					});
				}
			},
			createExam() {
				// 这里可以添加开始考试的逻辑，例如根据选择的categoryIds向后端发送请求等
				
				if(this.categoryIds.length==0){
					this.$refs.popup.open('center')
					
					return
				}
				uni.navigateTo({
					url:'../test/testIndex?subjectId='+this.categoryIds,
				})
			},
			
		}
	};
</script>

<style lang="scss">
	@mixin flex {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
	}
	
	@mixin height {
		/* #ifndef APP-NVUE */
		height: 100%;
		/* #endif */
		/* #ifdef APP-NVUE */
		flex: 1;
		/* #endif */
	}
	.contain{
		margin: 10px
	}
	.box {
		@include flex;
	}
	
	.button {
		@include flex;
		align-items: center;
		justify-content: center;
		flex: 1;
		height: 35px;
		margin: 0 5px;
		border-radius: 5px;
	}
	
	.example-body {
		background-color: #fff;
		padding: 10px 0;
	}
	
	.button-text {
		color: #fff;
		font-size: 12px;
	}
	.category-title {
		font-weight: bolder;
		padding: 10px 5px;
	}

	.select-all {
		margin-top: 20px;
	}

	.btn-panel button {
		margin-top: 10px;
		background-color: #e278d4;
		
	}
	.tips {
		font-size: 14px;
		margin-top: 10px;
		color: #7f7f7f;
	}
	.popup-content {
		@include flex;
		align-items: center;
		justify-content: center;
		padding: 15px;
		height: 50px;
		background-color: #fff;
	}
</style>