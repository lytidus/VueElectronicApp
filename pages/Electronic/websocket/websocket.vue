<template>
	<view>
		<image class="bg-0" src="../../../static/image/bg-0.gif"></image>
		<!-- <page-head title="websocket"></page-head> -->
		<view class="logo">
			<image src="../../../static/image/logo.png" mode="widthFix"></image>
			<view class="websocket-msg">{{showMsg}}</view>
		</view>

		<view class="products">			
			<view class="names">
				<!-- <image src="../../../static/image/chanpin.png" mode="widthFix"></image> -->
				<text>{{showspame}}</text>
			</view>		
			<view class="num">{{showspqty}}</view>
		</view>
	</view>
</template>

<script>
	let platform = uni.getSystemInfoSync().platform
	export default {
		data() {
			return {
				connected: false,
				connecting: false,
				msg: false,
				timer: null,
				heartbeatInterval: null, //检测服务器端是否还活着
				reconnectTimeOut: null, //重连之后多久再次重连
				spname: '',
				spqty: '',
			}
		},
		computed: {
			showMsg() {
				if (this.connected) {
					if (this.msg) {
						return this.msg
					} else {
						return '连接成功'
					}
				} else if (this.msg != '') {
					return this.msg
				} else {
					return '未连接'
				}
			},
			showspame() {
				return this.spname;
			},
			showspqty() {
				return this.spqty;
			}
		},
		onShow() {
			if (!this.connected) {
				//选择适合需求的定时器
				this.timer = setTimeout(() => {
					this.connect()
				}, 1000)
				this.timer = setInterval(() => {
					this.connect()
				}, 1000)
			} else {
				//选择适合需求的定时器 
				this.timer = setTimeout(() => {
					// 这里添加您的逻辑
					uni.onSocketMessage((res) => {
					let message = JSON.parse(res.data)
					this.spname = message.SPNAME;
					this.spqty = message.SPQTY;
					})
				}, 1000)
			}
		},
		onUnload() {
			uni.closeSocket()
			uni.hideLoading()
		},
		methods: {
			connect() {
				if (this.connected || this.connecting) {
					console.log('正在连接或者已经连接，请勿重复连接')
					return
				}
				this.connecting = true
				console.log('连接中')
				uni.connectSocket({
					url: getApp().globalData.connecturl,
					data() {
						return {
							msg: '连接成功'
						}
					},
					// #ifdef MP
					header: {
						'content-type': 'application/json'
					},
					// #endif
					// #ifdef MP-WEIXIN
					method: 'GET',
					// #endif
					success(res) {
						// 这里是接口调用成功的回调，不是连接成功的回调，请注意
					},
					fail(err) {
						// 这里是接口调用失败的回调，不是连接失败的回调，请注意
					}
				})
				uni.onSocketOpen((res) => {
					this.connecting = false
					this.connected = true
					uni.hideLoading()
					uni.showToast({
						icon: 'none',
						title: '连接成功'
					})
					console.log('onOpen', res);
				})
				uni.onSocketError((err) => {
					this.connecting = false
					this.connected = false
					uni.hideLoading()
					this.msg = '连接失败';
					console.log('onError', err);
				})
				uni.onSocketMessage((res) => {
					this.msg = "连接成功"
					let message = JSON.parse(res.data)
					this.spname = message.SPNAME;
					this.spqty = message.SPQTY;
					console.log('onMessage', res)
				})
				uni.onSocketClose((res) => {
					this.connected = false
					this.startRecive = false
					this.msg = "未连接"
					uni.closeSocket()
					console.log('onClose', res)
				})
			},
			send() {
				uni.sendSocketMessage({
					data: 'from ' + platform + ' : ' + parseInt(Math.random() * 10000).toString(),
					success(res) {
						console.log(res);
					},
					fail(err) {
						console.log(err);
					}
				})
			},
			close() {
				uni.closeSocket()
				clearTimeout(this.timer);
				this.timer = null;
			},
			//重新连接
			reconnect() {
				//停止发送心跳
				clearInterval(this.heartbeatInterval)
				//如果不是人为关闭的话，进行重连
				console.log(this.connected)
				if (!this.connected) {
					this.reconnectTimeOut = setInterval(() => {
						this.connect()
					}, 1000)
				}
			}
		}
	}
</script>

<style>
	.bg-0{
		position: fixed;
		width:100%;
		height: 100%;
		opacity: 0.3;
		z-index: 0;
	}
	uni-page-body{
		background-color: #10111a !important;
		/* background:url(../../../static/image/bg-0.gif) no-repeat;
		background-size: 100% 100%; */
	}
	body{
		background-color: #10111a;
	}
	.logo{
		padding:0 15% 0 2%;
		height: 60px;
		line-height: 60px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		width:83%;
		position: relative;
		z-index: 9;
	}
	.logo image{
		width:25%;
	}
	.products{
		width:92%;
		height: 430px;
		margin:10px auto;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 34px;
		padding:0;
		background-color: #f9f9f9;
		border-radius: 5px;
		border:1px solid #eee;
		background-color: rgba(2,14,35,0.8);
		color: #fff;
		text-align: center;
		position: relative;
		z-index: 9;
	}
	.products .num{
		font-size: 240px;
		font-weight: 600;
		line-height: 430px;
		border-left: 1px solid #ddd;
		padding:0 1%;
	}
	.products .names{
		display: flex;
		justify-content: center;
		align-items: center;
		text-align: center;
		font-size: 90px;
		line-height: 120px;
		margin: 0 auto;
	}
	.products .names image{
		width:35px;
		margin-right: 6px;
	}
	.operate{
		width:88%;
		margin:5% auto 2%;
		text-align: center;
	}
	.operate .p{
		font-size: 14px;
		color: #666;
		line-height: 50px;
	}
	.operate button{
		font-size: 22px;
	}
	.uni-btn-v {
		padding: 10rpx 0;
	}

	.uni-btn-v button {
		margin: 20rpx 0;
	}
.websocket-msg{
	color: #f9f9f9;
	font-size: 16px;
}
	/* .websocket-msg {
		padding: 40px 0px;
		text-align: center;
		font-size: 14px;
		line-height: 40px;
		color: #666666;
	} */

    .websocket-tips{
        padding: 40px 0px;
        text-align: center;
        font-size: 14px;
        line-height: 24px;
        color: #666666;
    }
</style>
