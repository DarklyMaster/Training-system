<template>
	<view class="upTrainingCss">
		<view class="uni-list">
			<view class="uni-list-cell">题目</view>
			<textarea v-model="tm" :placeholder="placeholder" :auto-height="true"
				style="margin-bottom: 5px;"></textarea>
		</view>

		<view class="uni-list">
			<view class="uni-list-cell">试题类型</view>
			<view class="uni-list-cell">
				<radio-group> <!-- 设置为列方向以换行显示 -->
					<radio @click="stlx('dans')">单选题</radio>
					<radio @click="stlx('duos')" style="margin: 0 0 0 10px;">多选题</radio>
				</radio-group>
			</view>
		</view>
		<view class="uni-list">
			<view class="uni-list-cell">选项</view>
			<view v-if="stlxData!=''" class="">
				<view  class="uni-list-cell">
					A <input v-model="tmda1" />
				</view>
				<view class="uni-list-cell">
					B <input v-model="tmda2" />
				</view>
				<view class="uni-list-cell">
					C <input v-model="tmda3" />
				</view>
				<view class="uni-list-cell">
					D <input v-model="tmda4" />
				</view>
				<view v-if="stlxData=='duos'" class="uni-list-cell">
					E <input v-model="tmda5" />
				</view>
			</view>
			
		</view>
		<view class="uni-list">
			<view class="uni-list-cell">
				答案
			</view>
			<view v-if="stlxData!='duos'" class="uni-list-cell">

				<radio-group>
					<radio @click="ckdaDan('A')">A</radio>
					<radio @click="ckdaDan('B')">B</radio>
					<radio @click="ckdaDan('C')">C</radio>
					<radio @click="ckdaDan('D')">D</radio>

				</radio-group>

			</view>
			<view v-else class="uni-list-cell">
				<checkbox-group >
					<checkbox  @click="ckdaDuo('A')">A</checkbox>
					<checkbox @click="ckdaDuo('B')">B</checkbox>
					<checkbox @click="ckdaDuo('C')">C</checkbox>
					<checkbox @click="ckdaDuo('D')">D</checkbox>
					<checkbox @click="ckdaDuo('E')">E</checkbox>
				</checkbox-group>
			</view>
		</view>
		<view class="uni-list">
			<view class="uni-list-cell">
				答案解析
			</view>
			<textarea v-model="jx" :placeholder="placeholder" :auto-height="true"
				style="margin-bottom: 5px;"></textarea>
		</view>
		<button @click="upTm()">上传</button>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				tm:'', // 绑定的文本
				placeholder: '请输入内容...', // 占位符
				stlxData: '',
				ckda:'',
				jx: '',
				tmda1:'',
				tmda2:'',
				tmda3:'',
				tmda4:'',
				tmda5:'',
				 selectedValues: []
			}
		},
		methods: {
			asd() {
				console.log(this.tmda1);
			},
			stlx(e) {
				this.stlxData = e
			},
			ckdaDan(ckda){
				this.ckda=ckda
				console.log(this.ckda);
			},
			ckdaDuo(value){
				const index = this.selectedValues.indexOf(value);
				            if (index > -1) {
				                // 如果存在，则移除该值
				                this.selectedValues.splice(index, 1);
				            } else {
				                // 如果不存在，则添加该值
				                this.selectedValues.push(value);
				            }
				            // 对数组进行排序
				            this.selectedValues.sort();
				            // 将数组转换为字符串
				            this.ckda = this.selectedValues.join('');
				            console.log( this.ckda); // 输出结果字符串
				            return  this.ckda; // 返回结果字符串
			},
			async upTm() {
				return new Promise((resolve) => {
					uni.request({
						url: 'http://192.168.1.200/POST'+this.stlxData, // 确保此 URL 与后端一致
						method: 'POST',
						header: {
							'Content-Type': 'application/json' // 设置请求头
						},
						data: {
							Tm: this.tm,
							Tmda1: this.tmda1,
							Tmda2: this.tmda2,
							Tmda3: this.tmda3,
							Tmda4:this.tmda4,
							Tmda5:this.tmda5,
							Ckda:this.ckda,
							Jx:this.js
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


		}
	}
</script>

<style>
	.upTrainingCss {
		margin: 10px;
	}

	.upTrainingCss textarea {
		box-sizing: border-box;
		width: 100%;
		min-height: 50px;
		border: 1px solid #ccc;
		border-radius: 5px;
		padding: 10px;

	}

	.uni-list-cell {
		margin: 10px 0 5px 0;
	}
	.uni-list-cell input{
		width: 100%;
		height:	30px ;
		border-radius: 5px;
		border: 1px solid #ccc;
		margin: 0 0 0 10px;
	}
	.uni-list-cell radio {
		margin: 0 10px 0 0;
	}


</style>