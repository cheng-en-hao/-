<template>
	<view>
		<view class="radio-list">
			<view style="color: #DD524D;"> 请选择类别：</view>
			<radio-group class="radioG" @change="radioChange">
				<label class="radioR" v-for="(item, index) in radioItems" :key="item.value">
					<view>
						<radio :value="item.value" :checked="item.checked" />
					</view>
					<view> {{item.name}} </view>
				</label>
			</radio-group>
		</view>
		<view class="line"></view>

		<view class="content">
			<!-- 选择图片区域 -->
			<view class="images_box">
				<image class='img' :src='imgTempUrl' mode='aspectFill'></image>
			</view>
			<!-- 按钮区域 -->
			<button class="btn" type="primary" @click="imgAdd">选择图片</button>
			<!-- 识别结果 -->
			<view class="txt" v-for="(item, index) in resItems" :key="item.value" >
				<text> {{ (item.score * 100).toFixed(1) }}% ---> {{item.name}} </text>				
			</view>
		</view>
	</view>
</template>

<script>
	//定义本页用到的变量
	var imgB64 = ''
	var id = '25396884'
	var secret = '1cq72g8bzrBVLjT1nuCnW3mb'
	var typeUrl = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/animal?access_token=' //默认是动物识别

	export default {
		data() {
			return {				
				type: 0 ,
				imgTempUrl: '',
				resItems:[],
				radioItems: [
					{
						value: 0,
						name: '动物识别',
						checked: true
					},
					{
						value: 1,
						name: '植物识别',
						checked: false
					},
					{
						value: 2,
						name: 'Logo识别',
						checked: false
					}
				],

			}
		},

		// 分享给朋友
		onShareAppMessage: function() {},
		//分享到朋友圈
		onShareTimeline: function() {},

		methods: {
			radioChange(e) {
				//console.log(e.detail.value)
				this.type = e.detail.value
				if(this.type == 0) {  //动物识别
					typeUrl = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/animal?access_token='
				}
				if(this.type == 1) {  //植物识别
					typeUrl = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/plant?access_token='
				}
				if(this.type == 2) {  //商标识别
					typeUrl = 'https://aip.baidubce.com/rest/2.0/image-classify/v2/logo?access_token='
				}
				
			},

			//选择图片
			imgAdd() {
				wx.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'],
					sourceType: ['album', 'camera'],
					success: (res) => {
						this.imgTempUrl = res.tempFilePaths[0]
						this.getB64ByUrl(this.imgTempUrl)  //调用方法并传入参数
					}
				})
			},
			
			//转换成base64格式
			getB64ByUrl: function(url) {				
				const FileSystemManager = wx.getFileSystemManager();
				FileSystemManager.readFile({
					filePath: url,
					encoding: 'base64',
					success:(res) => {
						//console.log(res.data)
						imgB64 = res.data
						this.plantTap()  //调用方法
					},
					fail:(err) => {
						console.log(err)
					}
				})
			},
			
			plantTap() {
				let that = this
				this.getToken(function(token) {  //先调用getToken方法得到token,再传入getResult方法中运行
					that.getResult(token)
				});
			},
			
			getToken: function(callback) {  //回调函数，要有返回值
				wx.request({
					url: 'https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=' + id + '&client_secret=' + secret,
					data: {},
					header: {
						'content-type': 'application/x-www-form-urlencoded' // 默认值
					},
					success(res) {						
						//console.log(res)
						var token = res.data.access_token
						return callback(token) //返回值
					},
					fail(err) {
						console.log('错误',err)
					}
				});
			},
			getResult: function(token) { 
				let that = this
				wx.request({
					url: typeUrl + token, 
					method: "post",
					data: {
						image: imgB64
					},
					header: {
						'content-type': 'application/x-www-form-urlencoded' // 默认值
					},
					success(res) {
						console.log(res.data)
						that.resItems = res.data.result
					}
				})
			}

		}
	}
</script>

<style>
	.radio-list {
		margin-left: 3.3%;
	}

	.radioG {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: flex-start;
	}

	.radioR {
		display: flex;
		flex-direction: row;
		margin: 5rpx;
	}

	.line {
		border-top: 1rpx;
		border-top-style: solid;
		border-color: #007AFF;
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.btn {
		margin: 10px;
		width: 80%;
	}


	.images_box {
		border: 1rpx;
		border-style: solid;
		border-color: rgba(0, 0, 0, 0.452);
		width: 98%;
		height: 496rpx;
		margin: 10rpx;
		position: relative;
	}

	.img {
		width: 100%;
		height: 100%;
	}
	
	.txt{
		margin: 10px;
		font-size: 20px;
		color: #007AFF;
	}

</style>