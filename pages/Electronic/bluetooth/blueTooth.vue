<template>
	<view>
		<view v-for="(item,index) in devices" :key="index" class="boxs" @click="BlueIds(item)">
			<view>设备名称：{{item.name}}|设备deviceId：{{item.deviceId}}|设备charId：{{item.characteristicId}}|</view>
		</view>
	</view>
</template>

<script>
	var main, receiver, filter;
	var _codeQueryTag = false;
	export default {
		data() {
			return {
				devices: [],//蓝牙列表
				weightTotal:'',//重量
			}
		},
		methods: {
			// 初始化蓝牙设备
			seach_searchBle() {
				var that = this
				uni.openBluetoothAdapter({
					success(res) {
						uni.setStorage({key:"openBlue",data:1})
						that.seach_onDevice()
						uni.getBluetoothAdapterState({
							success: function(res) {
								if (res.available) {
									if (res.discovering) {
										that.stopFindBule()
									}
									//搜索蓝牙
									uni.startBluetoothDevicesDiscovery({
										success(res) {
											uni.showToast({title:'搜索到以下设备',icon:'none'})
										}
									})
									that.devices = []
								} else {
									uni.showToast({title: '本机蓝牙不可用',icon:'none'})
								}
							},
						})
					}
				})
			},
			// 停止搜索设备
			stopFindBule(){
				wx.stopBluetoothDevicesDiscovery({
					success(res) {
						uni.showToast({title: '停止搜索',icon:'none'})
					}
				})
			},
			// 获取蓝牙设备列表
			seach_onDevice() {
				var that = this
				//监听寻找到新设备的事件
				uni.onBluetoothDeviceFound(function(devices) {
					var re = JSON.parse(JSON.stringify(devices))
					let name = re.devices[0].name;
					//如果没有名字的设备就给个未知设备
					if(name ==''){
						name = "未知设备"
					}
					let deviceId = re.devices[0].deviceId
					that.devices.push({
						name: name,
						deviceId: deviceId,
						services: [],
						RSSI: re.devices[0].RSSI
					})
				})
			},
			// 点击蓝牙列表的某个蓝牙去连接蓝牙
			BlueIds(item) {
				var that = this;
				let deviceId = item.deviceId;
				wx.createBLEConnection({
					deviceId: deviceId,
					success(res) {
						if (res.errMsg == "createBLEConnection:ok") {
							uni.showLoading({title: '连接蓝牙中'})
							setTimeout(() => {that.lianjielanya(deviceId)}, 2000)
						} else {uni.showToast({title: '22连接失败，请重试',icon:'none'})}
					},
					fail(res) {uni.showToast({title: '33连接失败，请重试',icon:'none'})}
				})
			},
			// 连接蓝牙秤获取所有服务
			lianjielanya(deviceId){
				var that = this;
				wx.getBLEDeviceServices({
					deviceId: deviceId,
					success(res) {
						let serviceId = "";
						for (var s = 0; s < res.services.length; s++) {
							let serviceId = res.services[s].uuid
							wx.getBLEDeviceCharacteristics({
								deviceId: deviceId,
								serviceId: serviceId,
								success(res) {
									var re = JSON.parse(JSON.stringify(res));
									for (var c = 0; c < re.characteristics.length; c++) {
										if (re.characteristics[c].properties.write == true) {
											var uuid = re.characteristics[c].uuid
											for (var index in that.devices) {
												if (that.devices[index].deviceId == deviceId) {
													that.devices[index].services.push({
														serviceId: serviceId,
														characteristicId: uuid
													})
													break
												}
											}
											setTimeout(() => {wx.hideLoading()}, 2000)
											uni.showToast({title: '连接成功',icon:'none'})
										}
									}
									that.notifyBLECharacteristicValueChange(deviceId)
								}
							})
						}
					},
					fail(res) {uni.showToast({title: '11连接失败，请重试',icon:'none'})},
				})
			},
			// 获取蓝牙秤的重量
			notifyBLECharacteristicValueChange(deviceId) {
				console.log("进入了蓝牙秤",deviceId)
				var that = this
				wx.notifyBLECharacteristicValueChange({
					state: true,
					deviceId: deviceId,
					serviceId: "0000FFE0-0000-1000-8000-00805F9B34FB",
					characteristicId: "0000FFE1-0000-1000-8000-00805F9B34FB",
					success: (res) => {
						wx.onBLECharacteristicValueChange((res) => {
							//获取到的蓝牙秤返回的值不是十进制的数字，需要进行转换
							let resValue = res.value
							var view = new Uint8Array(resValue);
							let length = view.byteLength
							var binary = '';
							for(let i= 0; i < length; i++){
								binary += String.fromCharCode(view[i])
							}
							var splitA = binary.split(",");
							var weight = splitA[2];
							let b = weight.slice(1,8);
						// 接收蓝牙秤的值
						console.log("进入了蓝牙秤11",b)
						uni.$emit('blueTooth',Number(b))
						this.weightTotal = Number(b)
						console.log("重量",b);
						})
					},
					fail: (res)=> {
						console.log("蓝牙",res);
						uni.showToast({title:'获取失败，请重试',icon:'none'})
					}
				})
			},
		},
		mounted() {
			//打开页面自动获取蓝牙列表
			this.seach_searchBle()
		},
	}
</script>

<style>
.boxs{
	width: 100%;
	border: 1px solid #ccc;
	background: #ddd;
	font-size: 18px;
	padding: 20px 0;
	margin: 5px 0;
}
</style>