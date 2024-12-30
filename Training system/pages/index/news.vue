<template>
	<view class="container">
		<view class="title">{{title}}</view>
		<view class="art-content">
			<rich-text class="richText" :nodes="strings"></rich-text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title:'',
				strings:''
			}
		},
		onLoad:function(e){
			console.log(e);
			var id=e.newsid
			 uni.request({
			 	url: 'https://unidemo.dcloud.net.cn/api/news/36kr/'+id,
			 	method:'GET',
			 	data:{},
			 	success: res => {
			 		console.log(res);
			 		this.title = res.data.title;
					this.strings=res.data.content;
			 	},
				fail: error => {
				    console.error("Request failed", error);
				},
			 });
		},
		methods: {
			
		}
	}
</script>

<style>
.container{
	margin: 10px;
	
}

</style>
