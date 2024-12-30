<template>
	<view class="backgroundDaily">
		<view v-if="currentQuestionType === 'single'">
			一、单选题
			<view class="xuan">
				{{currentQuestion.tm}}
			</view>
			<view
				v-for="(option, optIndex) in [currentQuestion.tmda1, currentQuestion.tmda2, currentQuestion.tmda3, currentQuestion.tmda4]"
				:key="optIndex" :class="{
			          'selected': selectedSingleOption === option,
			          'correct': isAnswered && isCorrectOption(option),
			          'incorrect': isAnswered && selectedSingleOption === option && !isCorrectOption(selectedSingleOption)
			      }" @click="selectSingleOption(option)" class="xuan">
				<view>{{option}}</view>
			</view>
			<!-- 确定按钮 -->
			<button v-if="!isAnswered" @click="confirmSingleAnswer" :disabled="!selectedSingleOption">确定</button>
			<!-- 下一页按钮 -->
			<button v-else @click="nextQuestion">下一页</button>
			<view v-if="isAnswered">
				<view>正确答案是：{{ correctAnswer }}</view>
				<view>你的选择为：{{ selectedSingleOption.charAt(0) }}</view> <!-- 显示用户选择的选项 -->
				<view v-if="(currentQuestion.jx!='')">
					{{currentQuestion.jx}}
				</view>
			</view>
		</view>
		<view v-if="currentQuestionType === 'multiple'">
			二、多选题
			<view class="xuan">
				{{currentQuestion.tm}}
			</view>
			<view
				v-for="(option, optIndex) in [currentQuestion.tmda1, currentQuestion.tmda2, currentQuestion.tmda3, currentQuestion.tmda4, currentQuestion.tmda5]"
				:key="optIndex" :class="{
					  'selected': selectedMultipleOptions.includes(option),
					  'correct': isAnswered && isOptionCorrect(option),
					  'incorrect': isAnswered && !isOptionCorrect(option) && selectedMultipleOptions.includes(option)
				  }" @click="toggleMultipleOption(option)" class="xuan">
				<view>{{option}}</view>
			</view>
			<!-- 确定按钮 -->
			<button v-if="!isAnswered" @click="confirmMultipleAnswer"
				:disabled="selectedMultipleOptions.length === 0">确定</button>
			<!-- 下一页按钮 -->
			<button v-else @click="nextQuestion">下一页</button>
			<view v-if="isAnswered">
				<view>正确答案是：{{ correctAnswer }}</view>
				<view>你选择的选项为：{{ formatSelectedMultipleOptions() }}</view> <!-- 显示用户选择的选项 -->
				<view v-if="(currentQuestion.jx!='')">
					{{currentQuestion.jx}}
				</view>
			</view>
		</view>
		<!-- 完成页面 -->
		<view v-if="currentQuestionType === 'submit'">
			<view>你完成了 {{ totalQuestions }} 题，正确 {{ correctCount }} 题。</view>
			<button @click="restartQuiz">重新开始</button>
			<button @click="asd">太简单了！再来几题</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
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

		onLoad() {
			this.loadRandomNumber(); // 组件挂载后生成随机数
			this.loadState();
		},
		onReady() {
			this.GetDanData();
			this.GetDuoData();
		},
		computed: {
			currentQuestionType() {
				if (this.currentPage < this.danData.length) {
					return 'single';
				} else if (this.currentPage < this.danData.length + this.duoData.length) {
					return 'multiple';
				} else {
					return 'submit'; // 提交页面
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
				return this.danData.length + this.duoData.length; // 总题数
			}
		},



		methods: {
			getRandomItems(data, count) {
				const shuffled = data.sort(() => 0.5 - Math.random()); // 随机打乱数组
				return shuffled.slice(0, count); // 返回前 count 项
			},
			loadRandomNumber() {
				// 从本地存储加载随机数
				const storedNumber = uni.getStorageSync('randomNumber');
				if (storedNumber) {
					this.randomNumber = storedNumber; // 使用存储的随机数
				} else {
					this.generateRandomNumber(); // 生成新的随机数并保存
				}
			},
			generateRandomNumber() {
				// 生成一个 0 到 19 之间的随机数
				this.randomNumber = (Math.floor(Math.random() * 19) + 1).toString(); // 0 到 19
				uni.setStorageSync('randomNumber', this.randomNumber);
			},
			async GetDanData() {
				try {
					const res = await uni.request({
						url: 'http://192.168.1.200:8080/Getdans/' + this.randomNumber,
						method: 'GET',
					});
					if (res.statusCode == 200) {
						const randomSingleChoices = res.data.data || []; // 确保是数组 // 调试信息
						this.danData = this.getRandomItems(randomSingleChoices, 5);
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},

			async GetDuoData() {
				try {
					const res = await uni.request({
						url: 'http://192.168.1.200:8080/Getduos/' + this.randomNumber,
						method: 'GET',
					});
					if (res.statusCode == 200) {
						const randomMultipleChoices = res.data.data || []; // 确保是数组
						this.duoData = this.getRandomItems(randomMultipleChoices, 5);
					} else {
						console.error('无法获取数据');
					}
				} catch (error) {
					console.error('获取数据时出错:', error);
				}
			},


			loadState() {
				try {
					const savedState = JSON.parse(uni.getStorageSync('quizStates') || '{}'); // 使用 uni.getStorageSync
					const currentState = savedState[this.randomNumber] || {};
					this.currentPage = currentState.currentPage || 0;
					this.selectedSingleOption = currentState.selectedSingleOption || '';
					this.selectedMultipleOptions = currentState.selectedMultipleOptions || [];
					this.isAnswered = currentState.isAnswered || false;
					this.correctCount = currentState.correctCount || 0;
				} catch (error) {
					console.error('加载状态时出错:', error);
				}
			},

			saveState() {
				try {
					const savedState = JSON.parse(uni.getStorageSync('quizStates') || '{}'); // 使用 uni.getStorageSync
					savedState[this.randomNumber] = {
						currentPage: this.currentPage,
						selectedSingleOption: this.selectedSingleOption,
						selectedMultipleOptions: this.selectedMultipleOptions,
						isAnswered: this.isAnswered,
						correctCount: this.correctCount
					};
					uni.setStorageSync('quizStates', JSON.stringify(savedState)); // 使用 uni.setStorageSync
				} catch (error) {
					console.error('保存状态时出错:', error);
				}
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
				try {
					const savedMistakes = JSON.parse(uni.getStorageSync('mistakeQuestions') || '[]');
					savedMistakes.push(question); // 添加错题
					uni.setStorageSync('mistakeQuestions', JSON.stringify(savedMistakes)); // 保存到 uni 存储
				} catch (error) {
					console.error('保存错题时出错:', error);
				}
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

			restartQuiz() {
				this.currentPage = 0; // 重置页面索引
				this.correctCount = 0; // 重置正确计数
				this.resetSelection(); // 重置选择状态
				const savedState = JSON.parse(localStorage.getItem('quizStates')) || {};
				delete savedState[this.randomNumber]; // 清除当前测验的状态
				localStorage.setItem('quizStates', JSON.stringify(savedState)); // 更新存储
			},
			asd() {
				uni.removeStorageSync('randomNumber');
				uni.switchTab({
					url: '/pages/training/training'
				})
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
</style>