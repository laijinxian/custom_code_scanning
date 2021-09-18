<template>
	<view class="wx-scan-content">
		<view class="scan-cameraCon">
			<camera class="scan-camera" device-position="back" flash="off" @error="error" :mode="scanMode"
				@scancode="cameraScan" :style="{ width: scanCameraWid, height: scanCameraHei }">
				<view class="cover-view-box">
					<!-- 左上角 -->
					<cover-view class="scan-border scan-left-top scan-verLine"></cover-view>
					<cover-view class="scan-border scan-left-top scan-horLine"></cover-view>
					<!-- 左下角 -->
					<cover-view class="scan-border scan-left-bottom scan-verLine"></cover-view>
					<cover-view class="scan-border scan-left-bottom scan-horLine"></cover-view>
					<!-- 右上角 -->
					<cover-view class="scan-border scan-right-top scan-verLine"></cover-view>
					<cover-view class="scan-border scan-right-top scan-horLine"></cover-view>
					<!-- 右下角 -->
					<cover-view class="scan-border scan-right-bottom scan-verLine"></cover-view>
					<cover-view class="scan-border scan-right-bottom scan-horLine"></cover-view>
					<!-- 上下移动的动画线 -->
					<cover-view class="scan-animation" :animation="scanAnimation"></cover-view>
				</view>
			</camera>
		</view>
	</view>

</template>

<script>
	let animation = uni.createAnimation({});
	// 提示音（如果有可以导入，这个我用手机震动来提示扫码完成）
	let innerAudioContext = uni.createInnerAudioContext();
	innerAudioContext.src = '/static/music/beep.mp3';
	export default {
		data() {
			return {
				// 照相模式下图片临时路径
				src: '',
				timer: '',
				goodsCode: '',
				// 扫码模式下的多次扫码列表
				scanGoods: [],
				// 动画
				scanAnimation: null
			};
		},
		props: {
			// 照相模式还是扫码模式（normal：代表照相模式，scanCode：代表扫码模式，注意微信暂不支持扫描小程序码，不要问为什么，微信功能是这样）
			scanMode: {
				type: String,
				default: 'scanCode'
			},
			// 扫码成功用是否用震动来提示（1：表示震动模式，2：表示mp3模式）
			scanTip: {
				type: Number,
				default: 2
			},
			// 相机的宽度
			scanCameraWid: {
				type: String,
				default: '250px'
			},
			// 相机的高度
			scanCameraHei: {
				type: String,
				default: '250px'
			}
		},
		created() {
			if (this.scanMode == 'scanCode') {
				// 初始化扫码动画
				this.initAnimation();
			}
		},
		methods: {
			cameraScan(e) {
				if (!e) return this.$ShowToast('扫描失败，请稍后再试')
				if (this.timer) return
				this.timer = setTimeout(() => {
					clearTimeout(this.timer)
					this.timer = null
					this.onScanSuccess(e.detail.result)
				}, 2000)
			},
			async onScanSuccess (barNumber) {
				// 扫码成功后让手机震动并播放声音
				uni.vibrateShort({
					success: function() {
						innerAudioContext.play();
					}
				});
				this.$emit('getGoodsInfo')
				// 通过扫码返回的结果，调用后台接口获取该规格商品信息添加到购物车
				// ......
			},
			initAnimation() {
				let that = this;
				// 控制向上还是向下移动
				let m = true;
				setInterval(() => {
					if (m) {
						animation.translateY(140).step({
							duration: 3000
						});
						m = !m;
					} else {
						animation.translateY(10).step({
							duration: 3000
						});
						m = !m;
					}
					that.scanAnimation = animation.export();
				}, 3000);
			},
			onAnimation () {
				animation.translateY(140).step({
					duration: 3000
				});
				this.scanAnimation = animation.export();
			},
			error(e) {
				if (e) {
					let that = this;
					// 如果用户不允许使用摄像头，需要提示引导用户授权
					uni.getSetting({
						success(res) {
							if (!res.authSetting['scope.camera']) {
								//获取摄像头权限
								uni.authorize({
									scope: 'scope.camera',
									success() {
										console.log('授权成功');
									},
									fail() {
										uni.showModal({
											title: '提示',
											content: '尚未进行摄像头权限授权，摄像头功能将无法使用',
											showCancel: false,
											success(res) {
												that.openAuthor();
											}
										});
									}
								});
							}
						},
						fail(res) {}
					});
				}
			},
			// 打开设置进行授权
			openAuthor() {
				let that = this;
				uni.openSetting({
					//这里的方法是调到一个添加权限的页面，可以自己尝试
					success: res => {
						if (!res.authSetting['scope.camera']) {
							uni.showModal({
								title: '提示',
								content: '尚未进行摄像头权限授权，摄像头功能将无法使用',
								showCancel: false,
								success(res) {
									if (res.confirm) {
										console.log('用户点击确定');
										that.openAuthor();

									} else if (res.cancel) {
										console.log('用户点击取消');
									}
								}
							});
						} else {
							uni.showModal({
								title: '提示',
								content: '授权成功！',
								showCancel: false,
								success: resData => {
									// 授权成功后，需要重新进入页面能调起相机组件
									uni.reLaunch({
										url: '/pagesService/GoodsPage/index'
									})
								}
							});
						}
					},
					fail: function() {
						console.log('授权摄像头权限失败');
					}
				});
			}
		}
	};
</script>

<style lang="scss" scoped>
	.wx-scan-content {
		height: 100%;
		position: relative;
		.cover-view-box {
			width: 650rpx;
			height: 150px;
			position: relative;
			z-index: 5;
			margin: 100rpx auto 60rpx;
			// 四个角边框
			.scan-border {
				background-color: #28E153;
				position: absolute;
			}
			
			.scan-verLine {
				width: 3px;
				height: 13px;
			}
			
			.scan-horLine {
				width: 13px;
				height: 3px;
			}
			
			// 左上角
			.scan-left-top {
				left: 0;
				top: 0;
			}
			
			// 左下角
			.scan-left-bottom {
				left: 0;
				bottom: 0;
			}
			
			// 右上角
			.scan-right-top {
				right: 0;
				top: 0;
			}
			
			// 右下角
			.scan-right-bottom {
				right: 0;
				bottom: 0;
			}
			
			// 相机中上下移动的线条
			.scan-animation {
				// position: absolute;
				// top: 5px;
				// left: 5px;
				width: 240px;
				height: 2px;
				margin: 0 auto;
				background-color: #28E153;
				border-radius: 50%;
			}
		}
		.scan-cameraCon {
			position: absolute;
			top: 0;
			left: 0;
			z-index: 3;
			width: 100%;
			height: 100%;

			.scan-camera {
				width: 100% !important;
				height: 100% !important;
				position: relative;
			}
		}
	}
</style>
