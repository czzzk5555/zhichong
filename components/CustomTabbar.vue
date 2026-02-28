<template>
	<view class="tabbar">
		<view class="tabbar-item" :class="{ active: activeIndex === 0 }" @tap="handleTabClick(0)">
			<text class="fa fa-home"></text>
		</view>
		<view class="tabbar-item" :class="{ active: activeIndex === 1 }" @tap="handleTabClick(1)">
			<text class="fa fa-location-dot"></text>
		</view>
		<view class="tabbar-item publish" @tap="handlePublishClick">
			<view class="plus-base">
				<view class="plus-inner" :class="{ active: isPlusActive }">
					<text class="fa fa-plus" :class="{ 'rotate-active': isPlusActive }"></text>
				</view>
			</view>
		</view>
		<view class="tabbar-item" :class="{ active: activeIndex === 3 }" @tap="handleTabClick(3)">
			<text class="fa fa-envelope"></text>
		</view>
		<view class="tabbar-item" :class="{ active: activeIndex === 4 }" @tap="handleTabClick(4)">
			<text class="fa fa-user"></text>
		</view>
	</view>
</template>

<script>
export default {
	name: 'CustomTabbar',
	props: {
		activeIndex: {
			type: Number,
			default: 0
		},
		// 未登录时，是否拦截“我的”(index=4)并跳转登录页
		authGuardMyTab: {
			type: Boolean,
			default: true
		}
	},
	data() {
		return {
			isPlusActive: false
		}
	},
	methods: {
		isLoggedIn() {
			// 你后续接 uni-id 时，可以改成读 uni-id token（例如：uni_id_token）
			return !!(uni.getStorageSync('token') || uni.getStorageSync('uni_id_token'))
		},
		handleTabClick(index) {
			// 第5个按钮（我的）：未登录则跳登录页
			if (this.authGuardMyTab && index === 4 && !this.isLoggedIn()) {
				this.$emit('need-login', index)
				return uni.navigateTo({ url: '/pages/auth/login' })
			}
			this.$emit('change', index)
		},
		handlePublishClick() {
			this.isPlusActive = !this.isPlusActive
			this.$emit('publish', this.isPlusActive)
		}
	}
}
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');

.tabbar {
	position: fixed;
	left: 24rpx;
	right: 24rpx;
	bottom: 22rpx;
	height: 118rpx;
	background: #383838;
	border-radius: 999rpx;
	display: flex;
	align-items: center;
	justify-content: space-around;
	padding: 0 32rpx;
	box-sizing: border-box;
	z-index: 1000;
	box-shadow: 0 12rpx 28rpx rgba(0, 0, 0, 0.28);
	padding-bottom: constant(safe-area-inset-bottom);
	padding-bottom: env(safe-area-inset-bottom);
}

.tabbar-item {
	flex: 1;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 38rpx;
	color: #b7b7b7;
}

.tabbar-item.active .fa {
	color: #E7FF60;
}

.tabbar-item .fa {
	transition: color 0.4s ease-in-out;
}

.publish {
	flex: 0;
	padding: 0 20rpx;
	position: relative;
}

.plus-base {
	width: 134rpx;
	height: 134rpx;
	background: #383838;
	border-radius: 50%;
	margin-top: -50rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 25;
}

.plus-inner {
	width: 110rpx;
	height: 110rpx;
	background: #E7FF60;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	box-shadow: 0 10rpx 20rpx rgba(231, 255, 96, 0.5);
	transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.plus-inner.active {
	background: #ff4f4f;
	box-shadow: 0 10rpx 20rpx rgba(255, 79, 79, 0.5);
}

.plus-inner .fa {
	font-size: 52rpx;
	color: #111;
	transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
	transform-origin: center center;
	display: block;
	line-height: 1;
}

.rotate-active {
	transform: rotate(45deg);
}
</style>
