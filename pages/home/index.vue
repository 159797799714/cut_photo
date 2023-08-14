<template>
	<view class="container">
		<use-list-title title="个人资料" iconfont=" "></use-list-title>
		<view class="gap"></view>
		
		<use-list-title v-if="aboutData && aboutData._id" title="关于用云" iconfont=" "></use-list-title>
		<view v-if="aboutData && aboutData._id" class="gap"></view>
		
		<!-- #ifdef MP-ALIPAY -->
		<view class="use-item padding-left">
			<button class="no-border wh-full tal" open-type="feedback">意见反馈</button>
		</view>
		<!-- #endif -->
		
		<!-- #ifndef MP-ALIPAY -->
		<view class="use-item">
			<button class="no-border wh-full tal" open-type="feedback">意见反馈</button>
		</view>
		<!-- #endif -->
		<view class="gap"></view>
		
		<use-list-title title="用云电商" iconfont=" " :tip="version"></use-list-title>
		<view class="gap"></view>
		
		<view class="use-item log-out-btn tac">
			<text>切换账号</text>
		</view>
		<view class="gap"></view>
		
		<view class="use-item log-out-btn tac">
			<text>退出登录</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				version: '1.0.0',
				aboutData: {}
			};
		},
		methods: {
		}
	}
</script>

<style lang='scss'>

	.use-item {
		height: 100rpx;
		line-height: 100rpx;
		position: relative;
		background: #fff;
		
		switch {
			transform: translateX(16rpx) scale(.84);
		}
		
		button {
			line-height: 100rpx;
			background: #fff;
			font-size: 15px;
		}
	}
</style>
