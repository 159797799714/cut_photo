<template>
	<canvas :canvas-id="`myCanvas-${index}`" :id="`myCanvas-${index}`" class="my-canvas"  :style="{height: canvasData.imgSpanHeight+'px',width: canvasData.imgSpanWidth+'px'}"  @error="canvasIdErrorCallback"></canvas>

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
				this.$nextTick(() => {
					imgUrl = imgUrl || this.imgUrl
					if (!imgUrl) return
					const { dWidth, dHeight, imgSpanWidth, imgSpanHeight, dx, dy} = this.canvasData
					const ctx = this.ctx || uni.createCanvasContext(`myCanvas-${this.index}`, this)
					ctx.drawImage(imgUrl, dx, dy, dWidth, dHeight)
					ctx.draw()	
				})
				
			},
			downLoad(cb) {
				const _this = this
					const { dWidth, dHeight} = this.canvasData
				uni.canvasToTempFilePath({
				  x: 0,
				  y: 0,
				  destWidth: dWidth,
				  destHeight: dHeight,
				  canvasId: `myCanvas-${this.index}`,
				  success: function(res) {
				    // 在H5平台下，tempFilePath 为 base64
				    console.log(res.tempFilePath)
						
						// 非h5
						// #ifndef H5
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
						// #endif
						
						// H5
						// #ifdef H5
						_this.savePicture(res.tempFilePath, cb)
						// #endif
						
						
				  } 
				}, this)
			},
			
			// H5
			// #ifdef H5
			// base64转blob文件再下载
			savePicture(base64, cb) {
				var arr = base64.split(',');
				var bytes = atob(arr[1]);
				let ab = new ArrayBuffer(bytes.length);
				let ia = new Uint8Array(ab);
				for (let i = 0; i < bytes.length; i++) {
					ia[i] = bytes.charCodeAt(i);
				}
				var blob = new Blob([ab], { type: 'application/octet-stream' });
				var url = URL.createObjectURL(blob);
				var a = document.createElement('a');
				a.href = url;
				a.download = new Date().valueOf() + ".png";
				var e = document.createEvent('MouseEvents');
				e.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
				a.dispatchEvent(e);
				URL.revokeObjectURL(url);
				
				cb && cb(true)
			},
			// #endif
			
			

			canvasIdErrorCallback: function (e) {
				console.error(e.detail.errMsg)
			}
		}
	}
</script>

<style lang="scss">

.my-canvas{
	float: left;
	display: block!important;
	z-index: 1;
}
</style>