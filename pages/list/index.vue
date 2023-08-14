<template>
	<view class="container">
		列表
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
