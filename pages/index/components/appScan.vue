<template>
	<view class="app-scan-container">
	</view>
</template>

<script>
	export default {
		data() {
			return {
				value: '',
				timer: null,
				barcode: null,
				scanGoods: [],
				viewContent: null,
				codeType: [
					plus.barcode.CODE128,
					plus.barcode.QR,
					plus.barcode.EAN13,
					plus.barcode.EAN8,
					plus.barcode.UPCA,
					plus.barcode.UPCE,
					plus.barcode.CODABAR,
					plus.barcode.CODE39,
					plus.barcode.CODE93,
					plus.barcode.ITF,
				]
			}
		},
		mounted () {
			const pages = getCurrentPages();
			const page = pages[pages.length - 1];
			// plus.navigator.setFullscreen(true); //全屏
			const currentWebview = page.$getAppWebview();
			this.initBarcode(currentWebview)
			this.createView(currentWebview)
		},
		methods: {
			initBarcode (currentWebview) {
				this.barcode = plus.barcode.create('barcode', this.codeType, {
					top: '200px',
					left: '0px',
					width: '100%',
					height: '300', //设置扫码框的高度
					position: 'static',
					scanbarColor: '#28E153',
					frameColor: '#28E153'
				});
				this.barcode.onmarked = this.onmarked;
				currentWebview.append(this.barcode);
				const res = uni.getSystemInfoSync();
				if(res.platform == 'android'){//安卓机
					this.barcode.start();
				}
			},
			async onmarked (type, result) {
				// 播放音频
				console.log('Success: type='+type+', result='+result);
				if (!result) return this.barcode && this.barcode.start();
				// 通过扫码返回的结果，调用后台接口获取该规格商品信息添加到购物车
				// ......
				this.timer = setTimeout(() => {
					this.barcode && this.barcode.start();
				}, 1000)
				this.$emit('getGoodsInfo')
			},
			closeBarcode () {
				this.barcode && this.barcode.close();
				this.viewContent && this.viewContent.close()
				this.barcode = null
				this.viewContent = null
			},
			// 创建展示类内容组件
			createView(currentWebview) {
				this.viewContent = new plus.nativeObj.View(
					'content', {
						top: '430px',
						left: '0px',
						height: '130px',
						width: '100%'
					},
					[{
							tag: 'font',
							id: 'scanTips',
							text: '将商品条码放入框中进行扫描',
							textStyles: {
								size: '16px',
								color: '#ffffff',
								whiteSpace: 'normal'
							},
							position: {
								top: '90px',
								left: '10%',
								width: '80%',
								height: 'wrap_content'
							}
						}
					]
				);
				currentWebview.append(this.viewContent);
			}
		},
		destroyed () {
			this.closeBarcode()
		}
	}
</script>

<style scoped>
.app-scan-container{
	height: 100%;
	overflow: hidden;
}
</style>
