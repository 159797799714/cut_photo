<template>
	<view class="img-view-item" :style="styleObj"></view>

		
</template>

<script>
	export default {
		name:"imgView",
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
			},
			slipeWidth: {
				type: [Number, String],
				default: 5
			}
		},
		computed: {
			styleObj() {
				let { imgSpanWidth, imgSpanHeight, dx, dy, dWidth, dHeight} = this.canvasData || {}
				imgSpanWidth -= Number(this.slipeWidth)
				imgSpanHeight -= Number(this.slipeWidth)
				
				
				// #ifdef MP-WEIXIN 
				// 微信小程序不支持background本地静态资源图片路径,故将本地资源转成base64
				return  `margin-bottom: ${this.slipeWidth}px;
					height: ${imgSpanHeight}px;
					width: ${imgSpanWidth}px;
					background-repeat: no-repeat;
					background-image: url(${this.urlTobase64()});
					background-size: ${dWidth}px ${dHeight}px;
					background-position: ${dx}px ${dy}px;`
				// #endif
				
				// #ifndef MP-WEIXIN 
				return  `
					height: ${imgSpanHeight}px;
					width: ${imgSpanWidth}px;
					margin-bottom: ${this.slipeWidth}px;
					background-repeat: no-repeat;background-image: url(${this.imgUrl});
					background-size: ${dWidth}px ${dHeight}px;
					background-position: ${dx}px ${dy}px;`
				// #endif
			}
		},
		methods: {
			/**
			 * 获取本地图
			 * @param folder // 文件夹名字 如 /static/images/home
			 * @param fileName // 文件名 如 home-bg
			 * @param format // 文件类型 如 png jpg
			 * @returns {*|string}
			 */
			  
			// #ifdef MP-WEIXIN
			urlTobase64 () {
			  let img = this.imgUrl,
			    imgBase64 = wx.getFileSystemManager().readFileSync(img, "base64"),
			    base64Url = `data:image/png;base64,${imgBase64}`;
			  return base64Url;
			},
			// #endif
			drawImg(imgUrl) {
				imgUrl = imgUrl || this.imgUrl
				if (!imgUrl) return
				const { dWidth, dHeight, imgSpanWidth, imgSpanHeight, dx, dy} = this.canvasData
				const ctx = this.ctx || uni.createCanvasContext(`myCanvas-${this.index}`, this)
				ctx.drawImage(imgUrl, dx, dy, dWidth, dHeight)
				ctx.draw()
			},
		}
	}
</script>

<style lang="scss">
</style>