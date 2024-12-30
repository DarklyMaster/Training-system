<template>
	<view class="backgroundDaily">
		<view v-if="currentQuestionType === 'single'">
			一、单选题
			<view class="timer">
				已用时：
			    {{ti}}
			</view>
			<view class="xuan">
				{{currentQuestion.tm}} <!-- 显示题目 -->
			</view>
			<view v-for="(option, optIndex) in [currentQuestion.tmda1, currentQuestion.tmda2, currentQuestion.tmda3, currentQuestion.tmda4]" :key="optIndex" 
			      :class="{
			          'selected': selectedSingleOption === option,
			          'correct': isAnswered && isCorrectOption(option),
			          'incorrect': isAnswered && selectedSingleOption === option && !isCorrectOption(selectedSingleOption)
			      }"
			      @click="selectSingleOption(option)"
				  class="xuan">
			    <view>{{option}}</view> <!-- 显示选项 -->
			</view>
			<!-- 确定按钮 -->
			<button v-if="!isAnswered" @click="confirmSingleAnswer" :disabled="!selectedSingleOption">确定</button>
			<!-- 下一页按钮 -->
			<button v-else @click="nextQuestion">下一页</button>
			<view v-if="isAnswered">
			    <view>正确答案是：{{ correctAnswer }}</view>
			    <view>你的选择为：{{ selectedSingleOption.charAt(0) }}</view> 
				<view v-if="(currentQuestion.jx!='')">
					{{currentQuestion.jx}}
				</view>
			</view>
		</view>
		<!-- 完成页面 -->
		<view v-if="currentQuestionType === 'submit'">
			<view>你完成了 {{ totalQuestions }} 题，正确 {{ correctCount }} 题。</view>

			<button @click="asd">返回</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				ti: '00:00:00',
				timer: '',
				hour: 0,
				min: 0,
				seconds: 0,
				categoryIds: '', // 题目ID列表
				danData: [], // 单选题目数据
				duoData: [], // 多选题目数据
				currentPage: 0, // 当前显示的页面索引
				selectedSingleOption: '', // 记录单选题选择的选项
				selectedMultipleOptions: [], // 记录多选题选择的选项
				isAnswered: false, // 是否已回答

				correctAnswer: '', // 记录正确答案
				correctCount: 0, // 记录正确答案的数量
				quizStates: {}, // 存储不同测验的状态
				randomNumber: '',
			};
		},

		onLoad(e) {
			const array = e.subjectId;
			this.categoryIds = array.split(',').map(Number);
			this.loadState();
			this.time();
		},
		onReady() {
			this.GetDanData(); // 获取单选题
			this.GetDuoData(); // 获取多选题
		},
		destroyed() {
			clearInterval(this.timer);
		},
		onHide() {
			if (this.timer) {
				clearInterval(this.timer);
			}
		},
		computed: {
		    currentQuestionType() {
		        if (this.currentPage < this.danData.length) {
		            return 'single';
		        } else if (this.currentPage < this.danData.length + this.duoData.length) {
		            return 'multiple';
		        } else {
		            return 'submit'; 
		        }
		    },
		    currentQuestion() {
		        if (this.currentQuestionType === 'single') {
		            return this.danData[this.currentPage];
		        } else if (this.currentQuestionType === 'multiple') {
		            return this.duoData[this.currentPage - this.danData.length];
		        }
		        return null;
		    },
		    totalQuestions() {
		        return this.danData.length + this.duoData.length; 
		    }
		},

		methods: {
			async GetDanData() {
				try {
					const promises = this.categoryIds.slice(0, 19).map(id =>
						uni.request({
							url: `http://192.168.1.200:8080/Getdans/${id}`,
							method: 'GET',
						})
					);
					const results = await Promise.all(promises);
					const shuffled = results.flatMap(result => result.data.data);
					const randomData=shuffled.sort(() => 0.5 - Math.random());
					this.danData = randomData.slice(0, 20);
				} catch (error) {
					console.error('获取单选题时出错:', error);
				}
			},

			async GetDuoData() {
				try {
					const promises = this.categoryIds.slice(0, 19).map(id =>
						uni.request({
							url: `http://192.168.83.51:8080/Getduos/${id}`,
							method: 'GET',
						})
					);
					
					const results = await Promise.all(promises);
					// console.log(results);
					// const shuffled = results.flatMap(result => result.data.data);
					// const randomData=shuffled.sort(() => 0.5 - Math.random());
					// this.duoData = randomData.slice(0, 20);
						
				} catch (error) {
					console.error('获取多选题时出错:', error);
				}
			},

			loadState() {
				const savedState = JSON.parse(localStorage.getItem('quizStates')) || {};
				const currentState = savedState[this.randomNumber] || {};
				this.currentPage = currentState.currentPage || 0;
				this.selectedSingleOption = currentState.selectedSingleOption || '';
				this.selectedMultipleOptions = currentState.selectedMultipleOptions || [];
				this.isAnswered = currentState.isAnswered || false;
				this.correctCount = currentState.correctCount || 0;
			},

			saveState() {
				const savedState = JSON.parse(localStorage.getItem('quizStates')) || {};
				savedState[this.randomNumber] = {
					currentPage: this.currentPage,
					selectedSingleOption: this.selectedSingleOption,
					selectedMultipleOptions: this.selectedMultipleOptions,
					isAnswered: this.isAnswered,
					correctCount: this.correctCount
				};
				localStorage.setItem('quizStates', JSON.stringify(savedState));
			},

			selectSingleOption(option) {
				if (!this.isAnswered) {
					this.selectedSingleOption = this.selectedSingleOption === option ? '' : option; // 切换选择
				}
			},

			toggleMultipleOption(option) {
				if (!this.isAnswered) {
					const index = this.selectedMultipleOptions.indexOf(option);
					if (index > -1) {
						this.selectedMultipleOptions.splice(index, 1); // 取消选择
					} else {
						this.selectedMultipleOptions.push(option); // 添加选择
					}
				}
			},

			confirmSingleAnswer() {
				this.isAnswered = true; // 标记为已回答
				this.correctAnswer = this.currentQuestion.ckda; // 记录正确答案
				if (this.isCorrectOption(this.selectedSingleOption)) {
					this.correctCount++; // 增加正确答案计数
				} else {
					this.saveMistake(this.currentQuestion); // 保存错题
				}
				this.saveState(); // 保存状态
			},

			confirmMultipleAnswer() {
				this.isAnswered = true; // 标记为已回答
				this.correctAnswer = this.currentQuestion.ckda; // 记录正确答案
				const correctAnswers = this.correctAnswer.split(''); // 获取正确答案
				const selectedAnswers = this.selectedMultipleOptions.map(option => option.charAt(0)); // 获取用户选择的答案
				const isCorrect = selectedAnswers.every(answer => correctAnswers.includes(answer)) &&
					selectedAnswers.length === correctAnswers.length; // 检查是否完全正确
				if (isCorrect) {
					this.correctCount++; // 增加正确答案计数
				} else {
					this.saveMistake(this.currentQuestion); // 保存错题
				}
				this.saveState(); // 保存状态
			},

			saveMistake(question) {
				const savedMistakes = JSON.parse(localStorage.getItem('mistakeQuestions')) || [];
				savedMistakes.push(question); // 添加错题
				localStorage.setItem('mistakeQuestions', JSON.stringify(savedMistakes)); // 保存到 localStorage
			},

			nextQuestion() {
				this.currentPage++;
				this.resetSelection(); // 重置选择状态
				this.saveState(); // 保存状态
			},

			resetSelection() {
				this.isAnswered = false; // 重置回答状态
				this.selectedSingleOption = ''; // 清空单选选择
				this.selectedMultipleOptions = []; // 清空多选选择
			},

			isCorrectOption(option) {
				const correctAnswers = this.currentQuestion.ckda.split(''); // 将正确答案字符串转换为数组
				return correctAnswers.includes(option.charAt(0)); // 检查选项是否在正确答案中
			},

			isOptionCorrect(option) {
				const correctAnswers = this.currentQuestion.ckda.split(''); // 获取正确答案
				return correctAnswers.includes(option.charAt(0)); // 检查选项是否在正确答案中
			},

			formatSelectedMultipleOptions() {
				// 将选项格式化为字母形式
				return this.selectedMultipleOptions.map(option => option.charAt(0)).join(''); // 显示选择的字母
			},

			
			asd() {
				uni.switchTab({
					url: '/pages/test/test'
				})
			},
			time() {
				this.timer = setInterval(this.startTimer, 1000);
			},
			startTimer() {
				this.seconds += 1;
				if (this.seconds >= 60) {
					this.seconds = 0;
					this.min = this.min + 1;
				}
				if (this.min >= 60) {
					this.min = 0;
					this.hour = this.hour + 1;
				}
				this.ti = (this.hour < 10 ? '0' + this.hour : this.hour) + ':' + (this.min < 10 ? '0' + this.min : this
					.min) + ':' + (this.seconds < 10 ? '0' + this.seconds : this.seconds);
			},
		}
	}
</script>

<style>
	.backgroundDaily::before {
		content: "";
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-image: url('/static/backgroundImg/16.png');
		/* 替换为您的图片路径 */
		background-size: cover;
		/* 使背景图片覆盖整个页面 */
		background-position: center;
		/* 使背景图片居中 */
		background-repeat: no-repeat;
		/* 不重复背景图片 */
		opacity: 0.8;
		/* 设置透明度为 0.5 */
		z-index: -1;
		/* 确保伪元素在内容下面 */
	}

	.selected {
		background-color: #d0e0f0;
		/* 选中状态的背景色 */
	}

	.correct {
		background-color: #c8e6c9;
		/* 正确选项的背景色 */
	}

	.incorrect {
		background-color: #ffcdd2;
		/* 错误选项的背景色 */
	}

	.xuan {
		padding: 6px 12px 6px 12px;
	}

	.timer {
		display: flex;
		padding: 0px 5px;
		float: right;
	}
</style>