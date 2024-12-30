<template>
	<view class="backgroundMistakeBook">
		<view v-if="mistakeQuestions.length === 0">
			<view>并没有错题</view>
			<button @click="goBack">返回</button>
		</view>
		<view v-else>
			<view class="question-title">{{ currentQuestion.tm }}</view>
			<view
				v-for="(option, optIndex) in [currentQuestion.tmda1, currentQuestion.tmda2, currentQuestion.tmda3, currentQuestion.tmda4]"
				:key="optIndex" :class="{
					  'correctAm': isAnswered && isCorrectOption(option, currentQuestion.ckda),
					  'incorrectAm': isAnswered && selectedMistakeOption === option && !isCorrectOption(option, currentQuestion.ckda),
					  'selectedAm': !isAnswered&&selectedMistakeOption === option // 添加选中样式
				  }" @click="selectMistakeOption(option)" class="option">
				<view>{{ option }}</view>
			</view>
			<view v-if="isAnswered">
				<view>正确答案是：{{ currentQuestion.ckda }}</view>
				<view>你的选择为：{{ selectedMistakeOption.charAt(0) }}</view>
			</view>
			<button v-if="selectedMistakeOption" @click="confirmMistakeAnswer">确认</button>
			<button v-if="!selectedMistakeOption" disabled>确认</button>
			<button @click="nextMistakeQuestion" v-if="isAnswered">下一题</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				mistakeQuestions: [], // 存储错题
				selectedMistakeOption: '', // 记录用户选择的选项
				currentMistakeIndex: 0, // 当前错题索引
				isAnswered: false // 是否已回答
			};
		},
		mounted() {
			this.loadMistakeQuestions(); // 组件挂载后加载错题
		},
		computed: {
			currentQuestion() {
				return this.mistakeQuestions[this.currentMistakeIndex]; // 获取当前错题
			}
		},
		methods: {
			loadMistakeQuestions() {
				try {
					const savedMistakes = JSON.parse(uni.getStorageSync('mistakeQuestions') || '[]');
					this.mistakeQuestions = savedMistakes;
				} catch (error) {
					console.error('加载错题时出错:', error);
					this.mistakeQuestions = []; // 如果出错，确保错题数组为空
				}
			},
			selectMistakeOption(option) {
				this.selectedMistakeOption = this.selectedMistakeOption === option ? '' : option; // 切换选择
			},
			isCorrectOption(option, correctAnswer) {
				return correctAnswer.split('').includes(option.charAt(0)); // 检查选项是否正确
			},
			confirmMistakeAnswer() {
				this.isAnswered = true; // 标记为已回答
			},
			nextMistakeQuestion() {
				if (this.isAnswered) {
					// 清除当前题目数据

					this.mistakeQuestions.splice(this.currentMistakeIndex, 1); // 从错题列表中移除当前题目
					this.selectedMistakeOption = ''; // 清空选择
					this.isAnswered = false; // 重置回答状态

					// 如果还有错题，更新当前索引
					if (this.mistakeQuestions.length > 0) {
						this.currentMistakeIndex = this.currentMistakeIndex >= this.mistakeQuestions.length ? 0 : this
							.currentMistakeIndex;
					}
				}
			},
			goBack() {
				uni.removeStorageSync('mistakeQuestions');
				uni.switchTab({
					url: '/pages/training/training' // 返回到训练页面
				});
			}
		}
	}
</script>

<style>
	.backgroundMistakeBook {
		position: relative;
		padding: 20px;
	}

	.question-title {
		font-weight: bold;
		margin-bottom: 10px;
	}

	.option {
		padding: 6px 12px;
		cursor: pointer;
	}

	.correctAm {
		background-color: #c8e6c9;
		/* 正确选项的背景色 */
	}

	.incorrectAm {
		background-color: #ffcdd2;
		/* 错误选项的背景色 */
	}

	.selectedAm {
		background-color: #bbdefb;
		/* 选中选项的背景色 */
	}
</style>