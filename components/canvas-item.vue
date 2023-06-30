<template>
	<view class="canvas-box">
		<canvas :canvas-id="`myCanvas-${index}`" :id="`myCanvas-${index}`" class="my-canvas"  :style="{height: canvasData.imgSpanHeight+'px',width: canvasData.imgSpanWidth+'px'}"  @error="canvasIdErrorCallback"></canvas>
	</view>
		
</template>

<script>
	export default {
		name:"canvasItem",
		data() {
			return {
				ctx: null,
			};
		},
		props: {
			canvasData: {
				type: Object,
				default: {
					imgSpanWidth: 120,
					imgSpanHeight: 120,
					windowWidth: 375
				}
			},
			index: {
				type: [String, Number],
				default: ''
			},
			imgUrl: {
				type: String,
				default: ''
			}
		},
		watch: {
			imgUrl: {
				handler(val) {
					console.log('来了老弟')
					this.drawImg && this.drawImg(val)
				},
				immediate: true
			},
			canvasData(val) {
					console.log('来了老弟canvasData')
					this.drawImg && this.drawImg()
			},
		},
		mounted() {
			const ctx = uni.createCanvasContext(`myCanvas-${this.index}`, this)
			this.ctx = ctx
		},
		methods: {
			drawImg(imgUrl) {
				imgUrl = imgUrl || this.imgUrl
				if (!imgUrl) return
				const { dWidth, dHeight, imgSpanWidth, imgSpanHeight, dx, dy} = this.canvasData
				const ctx = this.ctx || uni.createCanvasContext(`myCanvas-${this.index}`, this)
				ctx.drawImage(imgUrl, dx, dy, dWidth, dHeight)
				ctx.draw()
			},
			downLoad(cb) {
				uni.canvasToTempFilePath({
				  x: 0,
				  y: 0,
				  destWidth: 700,
				  destHeight: 700,
				  canvasId: `myCanvas-${this.index}`,
				  success: function(res) {
				    // 在H5平台下，tempFilePath 为 base64
				    console.log(res.tempFilePath)
						wx.saveImageToPhotosAlbum({
							filePath: res.tempFilePath,
							success(result) {
								cb && cb(true)
								console.log('保存成功', JSON.stringify(result))
							},
							fail(err) {
								cb && cb(false)
								console.log(`保存失败${JSON.stringify(err)}`)
							}
						})
				  } 
				}, this)
			},
			canvasIdErrorCallback: function (e) {
				console.error(e.detail.errMsg)
			}
		}
	}
</script>

<style lang="scss">

.my-canvas{
	display: block!important;
	z-index: 1;
}
</style>