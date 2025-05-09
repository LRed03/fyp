<template>
	<view class="container">
		<view class="header">
			<image class="header-bg" src="/static/back2.jpg" mode="aspectFill"></image>
			<view class="header-content">
				<!-- <image class="logo" src="/static/logo.png"></image> -->
				<text class="app-name">Medical Image Diagnosis</text>
				<text class="app-desc">Ai-assisted Medical Image Analysis System</text>
			</view>
		</view>

		<view class="main-content">
			<view class="card">
				<view class="card-title">
					<text class="icon">🔍</text>
					<text>Start</text>
				</view>
				<view class="card-content">
					<text class="description">Upload medical images to obtain AI-assisted diagnosis results</text>
					<button class="analyze-btn" @click="openImageSelector">Analysis</button>
				</view>
			</view>

			<view class="features">
				<view class="feature-item">
					<view class="feature-icon">🏥</view>
					<view class="feature-text">
						<text class="feature-title">Professional diagnosis</text>
						<text class="feature-desc">Based on advanced AI models</text>
					</view>
				</view>
				<view class="feature-item">
					<view class="feature-icon">⚡</view>
					<view class="feature-text">
						<text class="feature-title">Rapid analysis</text>
						<text class="feature-desc">Second level analysis report</text>
					</view>
				</view>
				<view class="feature-item">
					<view class="feature-icon">🔒</view>
					<view class="feature-text">
						<text class="feature-title">Data security</text>
						<text class="feature-desc">Strictly protect private data</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Medical image diagnosis',
				avatarBase64: ''
			}
		},
		onLoad() {
			// 页面加载时的逻辑
		},
		methods: {
			openImageSelector() {
				uni.showActionSheet({
					itemList: ['Take a picture', 'Select from album'],
					success: (res) => {
						if (res.tapIndex === 0) {
							// 拍照
							this.takePhoto();
						} else if (res.tapIndex === 1) {
							// 从相册选择
							this.chooseFromAlbum();
						}
					}
				});
			},
			// 安卓专用图片转Base64方法
			androidImageToBase64(filePath) {
				return new Promise((resolve, reject) => {
					// 方案1：先尝试使用plus.io转换（适用于5+ App）
					if (plus.io) {
						plus.io.resolveLocalFileSystemURL(filePath, entry => {
							entry.file(file => {
								const fileReader = new plus.io.FileReader();
								fileReader.onloadend = event => {
									resolve(event.target.result);
								};
								fileReader.readAsDataURL(file);
							}, reject);
						}, reject);
						return;
					}

					// 方案2：使用canvas转换（通用方案）
					uni.getImageInfo({
						src: filePath,
						success: imageInfo => {
							const ctx = uni.createCanvasContext('avatarCanvas', this);
							ctx.drawImage(imageInfo.path, 0, 0, 150, 150);
							ctx.draw(false, () => {
								setTimeout(() => {
									uni.canvasToTempFilePath({
										canvasId: 'avatarCanvas',
										fileType: 'jpg',
										quality: 0.8,
										success: canvasRes => {
											// 在安卓上直接使用临时路径
											resolve(canvasRes
												.tempFilePath);
										},
										fail: reject
									}, this);
								}, 300);
							});
						},
						fail: reject
					});
				});
			},
			imageToBase64(filePath) {
				uni.getFileSystemManager().readFile({
					filePath: filePath,
					encoding: 'base64',
					success: res => {
						this.avatarBase64 = 'data:image/jpeg;base64,' + res.data
						console.log('Base64转换成功', this.avatarBase64)
						uni.setStorageSync("base64Img",res.data)
						this.processImage(this.avatarBase64)
					},
					fail: err => { //
						console.log('Base64转换失败', err)
						// 方案1失败时尝试方案2
						this.imageToBase64Fallback(filePath)
					}
				})
			},
			imageToBase64Fallback(filePath) {
				uni.getImageInfo({
					src: filePath,
					success: (imageInfo) => {
						const ctx = uni.createCanvasContext('avatarCanvas')
						ctx.drawImage(imageInfo.path, 0, 0, 150, 150)
						ctx.draw(false, () => {
							uni.canvasToTempFilePath({
								canvasId: 'avatarCanvas',
								fileType: 'jpg',
								quality: 0.8,
								success: (res) => {
									const fs = uni.getFileSystemManager()
									fs.readFile({
										filePath: res.tempFilePath,
										encoding: 'base64',
										success: res => {
											this.avatarBase64 =
												'data:image/jpeg;base64,' + res
												.data
											console.log('备选方案Base64转换成功')
											console.log(this.avatarBase64)
										}
									})
								}
							})
						})
					}
				})
			},
			takePhoto() {
				uni.chooseImage({
					count: 1,
					sourceType: ['camera'],
					success: async (res) => {
						// this.processImage(res.tempFilePaths[0]);
						this.androidImageToBase64(res.tempFilePaths[0])
						try {
							// 安卓专用处理方式
							this.avatarBase64 = await this.androidImageToBase64(res.tempFilePaths[0]);
							this.processImage(this.avatarBase64)
						} catch (error) {
							console.error('转换失败:', error);
							uni.showToast({
								title: '图片处理失败',
								icon: 'none'
							});
						}
					}
				});
			},
			chooseFromAlbum() {
				uni.chooseImage({
					count: 1,
					sourceType: ['album'],
					success: async (res) => {
						// this.processImage(res.tempFilePaths[0]);
						// this.imageToBase64Fallback(res.tempFilePaths[0])
						this.imageToBase64(res.tempFilePaths[0])
						// try {
						// 	// 安卓专用处理方式
						// 	this.avatarBase64 = await this.androidImageToBase64(res.tempFilePaths[0]);
						// 	this.processImage(this.avatarBase64)
						// } catch (error) {
						// 	console.error('转换失败:', error);
						// 	uni.showToast({
						// 		title: '图片处理失败',
						// 		icon: 'none'
						// 	});
						// }
					}
				});
			},
			processImage(imagePath) {
				uni.setStorageSync('imagePath', imagePath)
				uni.showLoading({
					title: 'Loading...'
				});

				// 这里可以添加图像上传到服务器的逻辑
				// 模拟上传和处理过程
				setTimeout(() => {
					uni.hideLoading();

					// 跳转到结果页面
					uni.navigateTo({
						url: '/pages/analysis/result'
					});
				}, 1500);
			}
		}
	}
</script>

<style>
	.container {
		min-height: 100vh;
		background-color: #F5F7FA;
	}

	.header {
		position: relative;
		height: 400rpx;
		overflow: hidden;
	}

	.header-bg {
		position: absolute;
		width: 100%;
		height: 100%;
		z-index: 1;
	}

	.header-content {
		position: relative;
		z-index: 2;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100%;
		color: #FFFFFF;
	}

	.logo {
		width: 120rpx;
		height: 120rpx;
		margin-bottom: 20rpx;
	}

	.app-name {
		font-size: 44rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
		text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.2);
	}

	.app-desc {
		font-size: 28rpx;
		opacity: 0.9;
	}

	.main-content {
		padding: 30rpx;
		margin-top: -10rpx;
	}

	.card {
		background-color: #FFFFFF;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
		padding: 40rpx;
		margin-bottom: 30rpx;
	}

	.card-title {
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

	.card-content {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.description {
		font-size: 28rpx;
		color: #666666;
		margin-bottom: 40rpx;
		text-align: center;
	}

	.analyze-btn {
		width: 80%;
		height: 90rpx;
		line-height: 90rpx;
		background: linear-gradient(135deg, #4A7AFF 0%, #3B5EE8 100%);
		color: #FFFFFF;
		font-size: 32rpx;
		font-weight: bold;
		border-radius: 45rpx;
		box-shadow: 0 10rpx 20rpx rgba(74, 122, 255, 0.3);
	}

	.features {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}

	.feature-item {
		width: 30%;
		background-color: #FFFFFF;
		border-radius: 16rpx;
		padding: 30rpx 10rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		box-shadow: 0 4rpx 10rpx rgba(0, 0, 0, 0.05);
	}

	.feature-icon {
		font-size: 48rpx;
		margin-bottom: 20rpx;
	}

	.feature-text {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.feature-title {
		font-size: 28rpx;
		font-weight: bold;
		color: #333333;
		margin-bottom: 6rpx;
	}

	.feature-desc {
		font-size: 22rpx;
		color: #999999;
		text-align: center;
	}
</style>