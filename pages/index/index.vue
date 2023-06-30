<template>
	<view class="content">
		
		<view class="uni-list">
				<view class="list-left">
					生成图片数量：
				</view>
				<view class="list-right">
					<picker @change="bindPickerChange" :value="index" :range="array">
						<view class="uni-input">{{array[index]}}</view>
					</picker>
				</view>
		</view>
		
		<!-- canvas层级太高，所以利用定位canvas不可见 -->
		<view style="position: fixed;top: -100%;left: -100%;">
				<canvasItem v-for="(item, index) in count" :key="index" :imgUrl="imgUrl" :index="index"  :canvasData="item" :ref="`canvas${index}`" class="canvas-item"/>
		</view>
		
		<view class="box" :style="'height:'+windowHeight+'px'">
			<imgView v-for="(item, index) in count" :key="index" :imgUrl="imgUrl" :index="index"  :canvasData="item" :slipeWidth="slipeWidth"></imgView>
		</view>
		
		
		<view class="foot-btn">
			<button @tap="uploadImg" class="btn">{{imgUrl ? '更换': '上传'}}图片</button>
			<button v-if="imgUrl" @tap="downloadImg(0)" class="btn down">保存切图</button>
		</view>
		
		
	</view>
</template>

<script>
	import canvasItem  from "../../components/canvas-item.vue"
	import imgView  from "../../components/img-view.vue"
	export default {
		components: {imgView, canvasItem},
		data() {
			return {
				imageValue: [],
				imgUrl: '',
				count: [],
				windowWidth: 375,
				windowHeight: 375,
				slipeWidth: 4, // 间距
				imgInfo: {},
				array: ['4张', '9张', '16张'],
				index: 0
			}
		},
		computed: {
			boxHeight() {
				return this.imgInfo.height || this.windowWidth
			}
		},
		onLoad() {
			const _this = this
			uni.getSystemInfo({
				success: function (res) {
					_this.windowWidth = res.windowWidth
					_this.windowHeight = res.windowWidth
				}
			});
		},
		methods: {
			uploadImg() {
				let _this = this
				uni.chooseImage({
					count: 1, //默认9
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera '], //从相册选择
					success: function (res) {
						_this.imgUrl = res.tempFilePaths[0]
				
						uni.getImageInfo({
							src: res.tempFilePaths[0],
							success(result) {
								console.log('图片信息', result)
								_this.imgInfo = result
								_this.createCanvasList()
							}
						})
					}
				})

			},
			bindPickerChange(e) {
				this.index = e.detail.value
				this.createCanvasList()
			},
			createCanvasList() {
				
				if (!this.imgUrl) return
				
				const {width, height} = this.imgInfo
				// 图片比例
				const ratio = height / width
				this.windowHeight = this.windowWidth * ratio
				
				const num = Number(this.index) + 2
				
				// 图片一等分宽高
				const imgSpanWidth = this.windowWidth / num
				const imgSpanHeight = imgSpanWidth * ratio
				
				const canvasData = {
					imgSpanWidth,
					imgSpanHeight,
					dWidth: this.windowWidth,
					dHeight: this.windowWidth * ratio
				}
				
				const allCount = num * num + 1
				const list = []
				for(let i= 1;i<allCount;i++) { 
					
					let chuyushu = (i % num) // x轴
					chuyushu = chuyushu > 0 ? chuyushu: num  // 除余等于0则代表为最后一列
					
					let chu = Math.ceil(i / num) - 1 // y轴
					
					list.push({...canvasData,
						dx: imgSpanWidth * (chuyushu - 1) * -1,
						dy: imgSpanHeight * chu * -1,
					})
				}
				this.count = list
				
			},
			
			// #ifndef H5 || APP-PLUS
			// 判断是否请求过保存到相册授权
			checkSettingAuthorize() {
				return new Promise((resolve, reject) => {
					uni.getSetting({
					   success(res) {
					      console.log('获取设置', res.authSetting)
								const authSetting = JSON.stringify(res.authSetting)
							  if (authSetting.indexOf('scope.writePhotosAlbum') === -1) {
									resolve(false) // 未请求过授权，第一次获取权限
								} else {
									resolve(true) // 非第一次请求授权
								}
					   }
					})
				})
			},
			// 检测是否授权保存相册权限
			getAuthorize() {
				return new Promise((resolve, reject) => {
					
					uni.authorize({
						scope: 'scope.writePhotosAlbum',
						success(res) {
							console.log('res', res)
							resolve(true)
						},
						fail(err) {
							console.log('err', err)
							resolve(false)
						}
					})	
				})
			},
			// #endif
			
			
			async downloadImg(index = 0) {
				
				// #ifndef H5 || APP-PLUS
				
				const hadGet = await this.checkSettingAuthorize() // 是否请求过授权
				const auth = await this.getAuthorize() // 是否授权保存相册
				
				// 不是第一次请求授权且拒绝授权
				if (hadGet && !auth) {
					uni.showModal({
						title: '提示',
						content: '请开启添加到相册权限',
						confirmText: '去开启',
						success: function (res) {
							if (res.confirm) {
								console.log('用户点击确定');
								uni.openSetting({
									success(result) {
										console.log('打开设置成功', result)
									},
									fail(err) {
										console.log('打开设置失败', err)
									}
								})
							} else if (res.cancel) {
								console.log('用户点击取消')
							}
						}
					});
					console.log('非第一次授权没权限')
					return
				} 
				// 第一次授权，拒绝了
				else if (!auth) {
					console.log('第一次授权，拒绝了权限')
					return
				}
				// #endif
				
				
				const _this = this
				this.$refs[`canvas${index}`][0].downLoad(isLoad => {
					console.log(`第${index}张，下载：${isLoad}`)
					index += 1
					if (index >= _this.count.length) return uni.showToast({
						title: '下载完成'
					})
					_this.downloadImg(index)
				})
			}
		}
	}
</script>

<style lang="scss">
	page{
		height: 100%;
	}
	.content {
		height: 100%;
		display: flex;
		flex-direction: column;
		.uni-list{
			display: flex;
			justify-content: space-between;
			box-shadow: 8px 0px 0px -10px rgba(0,47,125,0.1);
			padding: 30rpx 20px;
			.list-right{
				color: $uni-color-primary;
			}
		}
		.box{
			background: #f5f5f5;
			display: flex;
			justify-content: space-around;
			flex-wrap: wrap;
			overflow-y: auto;
			// .canvas-item{
			// 	overflow: hidden;
			// 	float: left;
			// 	box-sizing: border-box;
			// }
		}
		.foot-btn{
			padding: 20rpx 20rpx 0;
			background: #FFFFFF;
			box-shadow: 0px -10px 8px 0px rgba(0,47,125,0.1);
			padding-bottom: env(safe-area-inset-bottom);
			padding-bottom: contant(safe-area-inset-bottom);
			.btn{
				margin: 20rpx auto 0;
				width: 95%;
				height: 88rpx;
				background: #1C6BEE;
				color: #ffffff;
				border-radius: 2rpx;
			}
		}
	}
</style>
