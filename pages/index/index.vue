<template>
	<view class="page-container">
		<view class="input">
			<input
				class="search-input"
				v-model="barNumber"
				placeholder="输入商品条码"
				@confirm="onInputSuccess"
				placeholder-style="color: #fff; font-size: 30rpx; padding-left: 15rpx">
			</input >
		</view>
		
		<!-- App -->
		<!-- #ifdef APP-PLUS -->
		<view>
			<AppScan ref="appScan" @getGoodsInfo="getGoodsInfo"></AppScan>
		</view>
		<!-- #endif -->
		
		<!-- 小程序 -->
		<!-- #ifdef MP-WEIXIN -->
		<view class="wx-scan-container">
			<WxScan ref="scanOrder" @getGoodsInfo="getGoodsInfo"></WxScan>
			<view class="wx-tip">将商品条码放入框中进行扫描</view>
		</view>
		<!-- #endif -->
		<!-- 购物车 -->
		<view class="footer">
			<view>
				<view class="badge">
					<text v-show="totalCount > 0">{{totalCount ? totalCount : ''}}</text>
					<image class="cart" src="../../static/cart.png" mode=""></image>
				</view>
				<view class="amout"> <text>￥</text> {{totalPrice}}</view>
			</view>
			<view>下单</view>
		</view>
		<!-- 购物车弹窗 -->
	</view>
</template>

<script>
	import AppScan from './components/appScan.vue'
	import WxScan from './components/wxScan.vue'
	export default {
		components: { AppScan, WxScan },
		data() {
			return {
				isFocus: false,
				barNumber: '',
				password: '',
				totalPrice: 0,
				totalCount: 0,
			}
		},
		onLoad() {

		},
		methods: {
			getGoodsInfo () {
				this.totalCount++
			},
			async onInputSuccess () {
				// 通过商品条码获取上商品详情
				// ......
				this.getGoodsInfo()
			},
		}
	}
</script>

<style lang="scss" scoped>
	.page-container{
		position: relative;
		height: 100%;
		overflow: hidden;
		background-color: rgba(0, 0, 0, 0.6);
		.input{
			position: relative;
			z-index: 20;
			margin-top: 80rpx;
			height: 70rpx;
			padding: 0 80rpx;
			/deep/ .search-input {
				height: 70rpx;
				color: #fff;
				border-radius: 40rpx;
				border: 1px solid #FFD202;
			}
		}
	}
	.wx-scan-container{
		margin-top: 100rpx;
		height:450rpx;
		.wx-tip{
			font-size: 30rpx;
			color: #FFFFFF;
			text-align: center;
			margin-top: 30rpx;
		}
	}
	.footer{
		position: fixed;
		bottom: 0;
		z-index: 10;
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		height: 108rpx;
		background: #FFFFFF;
		border-top: 1px solid #E8E8E8;
		> view {
			width: 50%;
			height: 100%;
			font-size: 34rpx;
			font-weight: 600;
			display: flex;
			justify-content: center;
			align-items: center;
			&:active{
				opacity: 0.8;
			}
			.cart{
				width: 67rpx;
				height: 67rpx;
				margin-top: 10rpx;
				margin-right: 25rpx;
			}
			.amout {
				font-size: 34rpx;
				> text {
					font-size: 20rpx;
					padding-right: 5rpx;
				}
			}
			.badge{
				position: relative;
				> text {
					position: absolute;
					top: 2rpx;
					right: 10rpx;
					z-index: 10;
					font-size: 22rpx;
					background: #FF3D00;
					border-radius: 50%;
					color: #fff;
					padding: 0 10rpx 2rpx;
				}
			}
		}
		> view:last-child{
			background: #FFD202;
		}
	}
</style>
