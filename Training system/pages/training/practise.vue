<template>
	<view class="backgroundPractise">
		<swiper class="swiper" :indicator-dots="true" :autoplay="false" :interval="5000" :duration="1000"
			touchmove="passive" @change="onSwiperChange">
			<swiper-item v-for="(item, index) in danData" :key="index">
				<view>
					<view class="xuan"> {{ item.id }}.{{ item.tm }} </view>
					<view v-for="(option, optIndex) in [item.tmda1, item.tmda2, item.tmda3, item.tmda4]" :key="optIndex"
						:class="getOptionClass(option)" @click="selectSingleOption(option)" class="xuan">
						<view>{{ option }}</view>
					</view>
				</view>
			</swiper-item>
			<swiper-item v-for="(item, index) in duoData" :key="index">
				<view>
					<view class="xuan"> {{ item.id }}.{{ item.tm }} </view>
					<view v-for="(option, optIndex) in [item.tmda1, item.tmda2, item.tmda3, item.tmda4, item.tmda5]"
						:key="optIndex" :class="getMultipleOptionClass(option)" @click="toggleMultipleOption(option)"
						class="xuan">
						<view>{{ option }}</view>
					</view>
				</view>
			</swiper-item>
			<swiper-item>
				<button @click="submitAnswers">提交</button>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				danData: [],
				duoData: [],
				score: 0,
				total: 0,
				isAnswered: false,
				correctAnswers: [],
				selectedOptions: [], // 用于存储每个题目的选择状态
				currentPage: 0,
				trainingId: '',
			};
		},
		onLoad(e) {
			this.trainingId = e;
			console.log(this.trainingId.subjectId);
		},
		onReady() {
			this.fetchBanners();
			this.duoDataa();
		},
		
		methods: {
			
			async fetchBanners() {
				try {
					const res = await uni.request({
						url: 'http://192.168.1.200:8080/Getdans/' + this.trainingId.subjectId,
						method: 'GET',
					});
					console.log(res);
					if (res.statusCode === 200) {
						this.danData = res.data.data;
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},
			async duoDataa() {
				try {
					const res = await uni.request({
						url: 'http://192.168.1.200:8080/Getduos/' + this.trainingId.subjectId,
						method: 'GET',
					});
					console.log(res);
					if (res.statusCode === 200) {
						this.duoData = res.data.data;
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},
			currentQuestion() {
				if (this.currentPage < this.danData.length) {
					return this.danData[this.currentPage];
				} else {
					return this.duoData[this.currentPage - this.danData.length];
				}
			},
			getOptionClass(option) {
				const questionIndex = this.currentPage < this.danData.length ? this.currentPage : this.currentPage - this.danData.length;
				return {
					'correct': this.isAnswered && (this.correctAnswers[questionIndex] === option.charAt(0)),
					'incorrect': this.isAnswered && (this.selectedOptions[questionIndex]?.single === option && this.correctAnswers[questionIndex] !== option.charAt(0)),
					'selected': this.selectedOptions[questionIndex]?.single === option
				};
			},
			getMultipleOptionClass(option) {
				const questionIndex = this.currentPage < this.danData.length ? this.currentPage : this.currentPage - this.danData.length;
				return {
					'correct': this.isAnswered && this.correctAnswers[questionIndex+this.danData.length]?.includes(option.charAt(0)),
					'incorrect': this.isAnswered && !this.correctAnswers[questionIndex+this.danData.length]?.includes(option.charAt(0)) && this.selectedOptions[questionIndex]?.multiple?.includes(option),
					'selected': this.selectedOptions[questionIndex]?.multiple?.includes(option)
				};
			},
			selectSingleOption(option) {
				if (!this.isAnswered) {
					const questionIndex = this.currentPage < this.danData.length ? this.currentPage : this.currentPage - this.danData.length;
					if (!this.selectedOptions[questionIndex]) {
						this.$set(this.selectedOptions, questionIndex, {
							single: null,
							multiple: []
						});
					}
					// 选择或取消选择单选项
					this.selectedOptions[questionIndex].single = this.selectedOptions[questionIndex].single === option ? null : option;
				}
			},
			
			toggleMultipleOption(option) {
				if (!this.isAnswered) {
					const questionIndex = this.currentPage < this.danData.length ? this.currentPage : this.currentPage - this.danData.length;
					if (!this.selectedOptions[questionIndex]) {
						this.$set(this.selectedOptions, questionIndex, {
							single: null,
							multiple: []
						});
					}
					const index = this.selectedOptions[questionIndex].multiple.indexOf(option);
					if (index > -1) {
						this.selectedOptions[questionIndex].multiple.splice(index, 1); // 取消选择
					} else {
						this.selectedOptions[questionIndex].multiple.push(option); // 添加选择
					}
				}
			},

			onSwiperChange(event) {
				this.currentPage = event.detail.current;
			},
			submitAnswers() {
				this.score = 0;
				this.total = this.danData.length + this.duoData.length;
				this.correctAnswers = []; // 清空正确答案

				// 处理单选题
				this.danData.forEach((item, index) => {
				    this.correctAnswers.push(item.ckda);
				    const selectedFirst = this.selectedOptions[index]?.single;
				    const selectedChars = typeof selectedFirst === 'string' ? selectedFirst.charAt(0) : '';
				    if (selectedChars === item.ckda) {
				        this.score++;
				    }
				});
				// 处理多选题
				this.duoData.forEach((item, index) => {
				    const correctAnswerArray = item.ckda.split(''); // 正确答案数组
				    this.correctAnswers.push(correctAnswerArray);
				    const selected = this.selectedOptions[index]?.multiple || [];
				    const selectedFirstChars = selected.map(option => option.charAt(0));
				    const isCorrect = selectedFirstChars.sort().join('') === correctAnswerArray.sort().join('');
				    // console.log(selectedFirstChars.sort().join(''));
					// console.log(correctAnswerArray.sort().join(''));
					if (isCorrect) {
				        this.score++;
				    }
				});
				uni.showToast({
					title: `总题数: ${this.total}, 答对: ${this.score}`,
					icon: 'none'
				});
				this.isAnswered = true;
				console.log();
			},
			
		}
	};
</script>


<style>
	.backgroundPractise::before{
		content: "";
		            position: absolute;
		            top: 0;
		            left: 0;
		            right: 0;
		            bottom: 0;
		            background-image: url('/static/backgroundImg/E-110.jpg'); /* 替换为您的图片路径 */
		            background-size: cover; /* 使背景图片覆盖整个页面 */
		            background-position: center; /* 使背景图片居中 */
		            background-repeat: no-repeat; /* 不重复背景图片 */
		            opacity: 0.5; 
		            z-index: -1; 
	}
	.selected {
		background-color: #d0e0f0;
	}

	.correct {
		background-color: #c8e6c9;
	}

	.incorrect {
		background-color: #ffcdd2;
	}

	.xuan {
		padding: 6px 12px;
	}

	.swiper {
		width: 100%;
		height: 100vh;
	}	
</style>
