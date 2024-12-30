//开屏广告
<template>
	<view class="startCss">
		
		
		<image  src="/static/backgroundImg/R-C.gif" class="img"></image>
		<view class="span">
			
			<view class="but" @click="tiao()"><span>{{count}}</span>s|跳过</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				count:"",
				adImage:''
			}
		},
		
	// onLoad:function(){
	// 	uni.request({
	// 		url:'',
			
	// 		success:(res)=>{
	// 			console.log(res)
	// 			this.adImage=res
	// 		}
	// 	})
	// },
	created(){
		this.dao()
	},
	
	methods: {
		
		dao(){
			const TIME_COUNT=5
			if(!this.timer){
				this.count=TIME_COUNT
				this.show=false
				this.timer=setInterval(()=>{
					if(this.count >0 && this.count <=TIME_COUNT){
						this.count--
					}else{
						this.show=true
						clearInterval(this.timer)
						this.timer=null
						uni.switchTab({
							url:'/pages/training/training',
						});
					}
				},1000)
			}
			
		},
		tiao (){
			this.count=0
		}
	},
	}
</script>

<style>
	.startCss{
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: flex-start;

	}
	.img{
		width: 100%;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.span{
		position: absolute;
		
		right: 0;
		white-space: nowrap;
		z-index: 10;
	}
	.but{
		background-color: rgba(255, 255, 255, 80%);
		width: 60px;
		height: 30px;
		line-height: 30px;
		margin: 10px;
		padding-left: 5px;
		border-radius: 50px;
		border: 1px solid #646464;
	}
</style>
