<template>
	<view>
		<!--
		<image class="background-image" src="../../static/bgimg.jpg" mode="aspectFill"></image>-->
		<view class="user-info">
			<view class="avatar">
				<open-data type="userAvatarUrl" default-avatar="../../static/default_avatar.jpg"></open-data>
			</view>
			<view class="nickname">
				<open-data type="userNickName"></open-data>
			</view>
		</view>
		<view class="auth-box" @tap="toAuth">
			<view class="check-box">
				<text class="cuIcon-check auth-check"></text>
			</view>
			<text class="auth-text" :class="authorized ? 'authed' : ''">身份认证</text>
		</view>
		<!--<view class="my-tab">
			<view class="my-tab-item" :class="index==tabCur ? 'selected' : ''" 
			v-for="(item, index) in myTabs" :key="index" @tap="tabSelect(index)">
				{{item}}
			</view>
		</view>-->
		<view class="action-box">
			<view class="my-action" v-for="(icon, title) in actions" :key="title" @tap="toAction(title)">
				<image :src="icon" class="action-icon"></image>
				<text class="action-title">{{title}}</text>
			</view>
		</view>
		<view class="dynamic-title">动态</view>
		<view class="dynamic-list">
			<order-box v-for="(item, index) in dynamicList" :key="index" :item="item" mode="dynamic"
			@send-confirm="onSendConfirm(index)" @receive-confirm="onReceiveConfirm(index)"></order-box>
		</view>
	</view>
</template>

<script>
	import orderBox from './order-box.vue'
	export default {
		components: {orderBox},
		data() {
			return {
				authorized: this.global.isAuthorized,
				actions: {
					'我买到的': '../../static/tabbar/shop_icon.png',
					'我卖出的': '../../static/tabbar/sell_icon.png',
					'我发布的': '../../static/tabbar/cart_icon.png'
				},
				dynamicList: []
			}
		},
		onShow() {
			this.authorized = this.global.isAuthorized
			this.updateDynamic()
		},
		onReady() {
			/*uni.navigateTo({
				url: 'login'
			})*/
		},
		methods: {
			toAuth() {
				uni.navigateTo({url: 'auth?authorized=' + this.global.isAuthorized})
			},
			toAction(title) {
				var index = Object.keys(this.actions).indexOf(title)
				uni.navigateTo({url: 'action?tab=' + index})
			},
			updateDynamic() {
				this.dynamicList = []
				var _this = this
				var token = uni.getStorageSync('token')
				if (token) {
					uni.request({
						url: this.global.serverUrl + "user/dynamics",
						data: {
							token: token
						},
						success: function (res) {
							if (res.statusCode == 200) {
								console.log(res.data.dynamicList);
								_this.dynamicList = res.data.dynamicList
								for (let item of _this.dynamicList) {
									item.imageUrl = _this.global.bucketUrl + item.imageName
								}
							}
							else if (res.statusCode == 204) {
								console.log('empty');
							}
							else if (res.statusCode == 403) {
								uni.showToast({title: 'token过期，请重新进入小程序', duration: 3000, icon: 'none'})
							}
						}
					})
				} else {
					uni.showToast({title: '未登录', duration: 3000, icon: 'none'})
				}
			},
			onSendConfirm(index) {
				uni.navigateTo({
					url: '../order/send?orderId=' + this.dynamicList[index].orderId
				})
			},
			onReceiveConfirm(index) {
				var _this = this
				uni.showModal({
					title: '提示',
					content: '确认已取到书本',
					success: function (res) {
						if (res.confirm) {
							_this.receiveConfirm(_this.dynamicList[index].orderId)
						} else if (res.cancel) {
							console.log('取消已取货');
						}
					}
				})
			},
			receiveConfirm(orderId) {
				var _this = this
				var token = uni.getStorageSync('token')
				uni.request({
					url: this.global.serverUrl + "order",
					method: 'PUT',
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						token: token,
						orderId: orderId,
						action: 'receive'
					},
					success: function (res) {
						if (res.statusCode == 200) {
							_this.updateDynamic()
						} else {
							uni.showToast({title: '请求失败，请稍后再试', duration: 3000, icon: 'none'})
						}
					}
				})
			}
			/*tabSelect(index) {
				this.tabCur = index
			}*/
		}
	}
</script>

<style>
	.background-image {
		width: 100%;
		height: 100%;
		position: absolute;
		top: 0;
		background-size: 100% 100%;
		z-index: -1;
	}
	
	.user-info {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.avatar {
		width: 170rpx;
		height: 170rpx;
		border-radius: 50%;
		overflow: hidden;
		margin-top: 120rpx;
	}
	
	.nickname {
		margin-top: 12rpx;
		font-size: 36rpx;
		color: #727272;
	}
	
	.auth-box {
		margin-top: 10rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.check-box {
		width: 30rpx;
		height: 30rpx;
		background-color: #D8D8D8;
		border: 1rpx solid #FFFFFF;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: flex-end;
	}
	.auth-check {
		color: #FFFFFF;
		font-size: 24rpx;
	}
	.auth-text {
		color: #B1B1B1;
		font-size: 30rpx;
	}
	.auth-text.authed {
		color: #FF8F97;
	}
	
	.action-box {
		width: 700rpx;
		height: 214rpx;
		background-color: #FFFFFF;
		border-radius: 18rpx;
		display: flex;
		justify-content: space-around;
		margin: 20rpx auto;
		padding: 0 30rpx;
	}
	.my-action {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 60rpx;
	}
	.action-icon {
		width: 72rpx;
		height: 72rpx;
		margin-bottom: 20rpx;
	}
	.action-title {
		color: #FF8F97;
		font-size: 32rpx;
	}
	
	.dynamic-title {
		margin-left: 50rpx;
		font-size: 32rpx;
	}
	.dynamic-list {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
</style>
