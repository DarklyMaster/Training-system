<template>
    <view class="">
		<button class="sequenceCss" v-for="(item,index) in subject" :key="index" @click="training({subjectId:item.id})">{{item.id}}.{{item.sjname}}</button>
    </view>
</template>
<script>
    export default {
    	data() {
    		return {
    			subject: [] ,// 题目数据将被存储在这里
    		};
    	},
    	onLoad() {
    		this.fetchBanners();
    	},
    	methods: {
    		async fetchBanners() {
    			try {
    				// 调用后端API获取题目数据
    				const res = await uni.request({
    					url: 'http://192.168.1.200:8080/Zbdans', // 替换API地址
    					method: 'GET',
    				});
    				// console.log(res);
    				if (res.statusCode == 200) {
    					this.subject = res.data.data; // 后端返回的数据格式正确
    				} else {
    					console.error('无法获取数据');
    				}
    			} catch (error) {
    				console.error('获取数据时出错:', error);
    			}
    		},
			training(e){
				var subjectId=e.subjectId
				// console.log(subjectId);
				uni.navigateTo({
					url:'./practise?subjectId='+subjectId,
				})
			}
    	}
    };
</script>　
<style  >
.sequenceCss{
	font-size: 16px
	/* height: 80px; */
}
</style>