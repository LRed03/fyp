<template>
	<view class="result-container">
		<view class="result-card">
			<view class="result-header">
				<text class="result-title">Diagnostic results</text>
				<text class="result-time">{{currentTime}}</text>
			</view>

			<view class="image-section">
				<image :src="imageUrl" mode="aspectFit" class="medical-image"></image>
			</view>

			<view class="divider"></view>

			<view class="analysis-section">
				<view class="section-title">
					<text class="icon">📋</text>
					<text>Analysis report</text>
				</view>

				<view class="analysis-content">
					<view v-if="isLoading" class="loading">
						<text>Under analysis...</text>
					</view>
					<view v-else>
						<view class="result-item" v-for="(item, index) in analysisResults" :key="index">
							<text class="item-label">{{item.label}}:</text>
							<text class="item-value">{{item.value}}</text>
						</view>
					</view>
				</view>
			</view>

			<view class="action-buttons">
				<button class="btn primary" @click="saveResult">Saving report</button>
				<button class="btn secondary" @click="goBack">Back</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imageUrl: '',
				isLoading: true,
				currentTime: '',
				analysisResults: []
			}
		},
		onLoad(options) {
			// 获取上传的图片路径
			// if (options.imageUrl) {
			// 	this.imageUrl = options.imageUrl;
			// }

			// 设置当前时间
			this.setCurrentTime();
			this.imageUrl = uni.getStorageSync('imagePath')
			console.log(this.imageUrl)
			uni.request({
				url: 'http://127.0.0.1:11434/api/generate',
				method: 'POST',
				data: {
					"model": "llava",
					"prompt": "请用英文描述这张医疗图像的细节",
					"images": [
						uni.getStorageSync("base64Img")
					]
				},
				success: (res) => {
					const lines = res.data.split('\n'); // 分割成多行
					let result = "";
					try {
						lines.map(item => {
							result += JSON.parse(item).response
						})
					} catch (error) {

					}
					this.isLoading = false;
					console.log(result)
					this.analysisResults = [{
						label: 'Result',
						value: result
					}];
				}
			})

			// 模拟分析过程
			// setTimeout(() => {
			// 	this.isLoading = false;
			// 	this.analysisResults = [
			// 		{ label: 'Type', value: '' },
			// 		{ label: 'Findings', value: '' },
			// 		{ label: 'Possible lesion', value: '' },
			// 		{ label: 'Suggestion', value: '' },
			// 		{ label: 'Confidence level', value: '' }
			// 	];
			// }, 2000);
		},
		methods: {
			setCurrentTime() {
				const now = new Date();
				const year = now.getFullYear();
				const month = String(now.getMonth() + 1).padStart(2, '0');
				const day = String(now.getDate()).padStart(2, '0');
				const hours = String(now.getHours()).padStart(2, '0');
				const minutes = String(now.getMinutes()).padStart(2, '0');

				this.currentTime = `${year}-${month}-${day} ${hours}:${minutes}`;
			},
			saveResult() {
				uni.showToast({
					title: '报告已保存',
					icon: 'success'
				});
			},
			goBack() {
				uni.navigateBack();
			}
		}
	}
</script>

<style>
	.result-container {
		padding: 30rpx;
		background-color: #F5F7FA;
		min-height: 100vh;
	}

	.result-card {
		background-color: #FFFFFF;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
		padding: 40rpx;
		margin-bottom: 30rpx;
	}

	.result-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.result-title {
		font-size: 40rpx;
		font-weight: bold;
		color: #333333;
	}

	.result-time {
		font-size: 28rpx;
		color: #999999;
	}

	.image-section {
		margin-bottom: 30rpx;
		display: flex;
		justify-content: center;
	}

	.medical-image {
		width: 100%;
		height: 400rpx;
		border-radius: 10rpx;
		background-color: #f0f0f0;
	}

	.divider {
		height: 2rpx;
		background-color: #EEEEEE;
		margin: 30rpx 0;
	}

	.section-title {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
		font-size: 34rpx;
		font-weight: bold;
		color: #333333;
	}

	.icon {
		margin-right: 10rpx;
		font-size: 36rpx;
	}

	.analysis-content {
		background-color: #F9FAFC;
		border-radius: 10rpx;
		padding: 30rpx;
	}

	.loading {
		display: flex;
		justify-content: center;
		padding: 40rpx 0;
	}

	.result-item {
		display: flex;
		margin-bottom: 20rpx;
		line-height: 1.6;
	}

	.item-label {
		width: 180rpx;
		color: #666666;
		font-weight: bold;
	}

	.item-value {
		flex: 1;
		color: #333333;
	}

	.action-buttons {
		display: flex;
		justify-content: space-between;
		margin-top: 40rpx;
	}

	.btn {
		width: 45%;
		height: 80rpx;
		line-height: 80rpx;
		border-radius: 40rpx;
		font-size: 30rpx;
		font-weight: bold;
	}

	.primary {
		background-color: #4A7AFF;
		color: #FFFFFF;
	}

	.secondary {
		background-color: #F0F2F5;
		color: #666666;
	}
</style>