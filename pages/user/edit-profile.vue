<template>
	<view class="page">
		<!-- 自定义导航栏 -->
		<view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
			<view class="navbar-content">
				<view class="navbar-left" @tap="handleBack">
					<text class="fa fa-angle-left navbar-icon"></text>
				</view>
				<view class="navbar-title">编辑资料</view>
				<view class="navbar-right" @tap="handleSave">
					<text class="save-text">保存</text>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<scroll-view class="scroll-content" scroll-y>
			<!-- 头像区域 -->
			<view class="avatar-section">
				<view class="avatar-wrapper" @tap="handleChangeAvatar">
					<image class="avatar-image" :src="userInfo.avatar" mode="aspectFill"></image>
					<view class="avatar-overlay">
						<text class="fa fa-camera avatar-icon"></text>
						<text class="avatar-text">更换头像</text>
					</view>
				</view>
			</view>

			<!-- 表单区域 -->
			<view class="form-section">
				<!-- 用户名 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-user form-icon"></text>
						<text class="form-label-text">用户名</text>
					</view>
					<input 
						class="form-input" 
						v-model="userInfo.username" 
						placeholder="请输入用户名"
						maxlength="20"
					/>
				</view>

				<!-- 性别 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-venus-mars form-icon"></text>
						<text class="form-label-text">性别</text>
					</view>
					<view class="gender-selector">
						<view 
							class="gender-option" 
							:class="{ 'gender-active': userInfo.gender === 'female' }"
							@tap="selectGender('female')"
						>
							<image class="gender-icon" src="/static/images/wode/nvhhai.svg" mode="aspectFit"></image>
							<text class="gender-text">女</text>
						</view>
						<view 
							class="gender-option" 
							:class="{ 'gender-active': userInfo.gender === 'male' }"
							@tap="selectGender('male')"
						>
							<image class="gender-icon" src="/static/images/wode/nanhai.svg" mode="aspectFit"></image>
							<text class="gender-text">男</text>
						</view>
					</view>
				</view>

				<!-- 知宠ID（只读） -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-id-card form-icon"></text>
						<text class="form-label-text">知宠ID</text>
					</view>
					<view class="form-readonly">
						<text class="readonly-text">{{ userInfo.userId }}</text>
						<text class="fa fa-lock readonly-icon"></text>
					</view>
				</view>

				<!-- 个性签名 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-pen form-icon"></text>
						<text class="form-label-text">个性签名</text>
					</view>
					<textarea 
						class="form-textarea" 
						v-model="userInfo.signature" 
						placeholder="介绍一下自己吧~"
						maxlength="100"
						:auto-height="true"
					></textarea>
					<view class="char-count">{{ userInfo.signature.length }}/100</view>
				</view>

				<!-- 背景图 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-image form-icon"></text>
						<text class="form-label-text">背景图</text>
					</view>
					<view class="banner-preview" @tap="handleChangeBanner">
						<image class="banner-image" :src="userInfo.banner" mode="aspectFill"></image>
						<view class="banner-overlay">
							<text class="fa fa-camera banner-icon"></text>
							<text class="banner-text">更换背景</text>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			statusBarHeight: 0,
			userInfo: {
				avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=180&h=180&q=80',
				username: 'EULA',
				gender: 'female', // 'male' or 'female'
				userId: '123456789',
				signature: '我是养萨摩耶的一个博主!!',
				banner: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=180&h=180&q=80'
			}
		}
	},
	onLoad() {
		// 获取状态栏高度
		const systemInfo = uni.getSystemInfoSync()
		this.statusBarHeight = systemInfo.statusBarHeight || 0

		// 从缓存或服务器加载用户信息
		this.loadUserInfo()
	},
	methods: {
		loadUserInfo() {
			// 这里可以从缓存或服务器加载用户信息
			// 暂时使用默认值
		},
		handleBack() {
			uni.navigateBack()
		},
		handleSave() {
			// 保存用户信息
			uni.showToast({
				title: '保存成功',
				icon: 'success',
				duration: 1500
			})
			// 这里可以调用API保存数据
			setTimeout(() => {
				uni.navigateBack()
			}, 1500)
		},
		handleChangeAvatar() {
			uni.chooseImage({
				count: 1,
				sizeType: ['compressed'],
				sourceType: ['album', 'camera'],
				success: (res) => {
					this.userInfo.avatar = res.tempFilePaths[0]
					uni.showToast({
						title: '头像已更换',
						icon: 'success'
					})
				},
				fail: (err) => {
					console.error('选择图片失败', err)
				}
			})
		},
		selectGender(gender) {
			this.userInfo.gender = gender
		},
		handleChangeBanner() {
			uni.chooseImage({
				count: 1,
				sizeType: ['compressed'],
				sourceType: ['album', 'camera'],
				success: (res) => {
					this.userInfo.banner = res.tempFilePaths[0]
					uni.showToast({
						title: '背景已更换',
						icon: 'success'
					})
				},
				fail: (err) => {
					console.error('选择图片失败', err)
				}
			})
		}
	}
}
</script>

<style scoped>
.page {
	width: 100%;
	min-height: 100vh;
	background: #f5f5f5;
}

/* 自定义导航栏 */
.custom-navbar {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background: #ffffff;
	z-index: 1000;
	border-bottom: 1rpx solid #f0f0f0;
}

.navbar-content {
	height: 88rpx;
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 0 32rpx;
}

.navbar-left {
	width: 60rpx;
	height: 60rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.navbar-icon {
	font-size: 40rpx;
	color: #1b1b1b;
}

.navbar-title {
	font-size: 36rpx;
	font-weight: 600;
	color: #1b1b1b;
}

.navbar-right {
	padding: 12rpx 24rpx;
}

.save-text {
	font-size: 32rpx;
	color: #1b1b1b;
	font-weight: 600;
}

/* 内容区域 */
.scroll-content {
	width: 100%;
	height: calc(100vh - var(--status-bar-height) - 88rpx);
	margin-top: calc(var(--status-bar-height) + 88rpx);
	padding-bottom: 40rpx;
	box-sizing: border-box;
}

/* 头像区域 */
.avatar-section {
	width: 100%;
	padding: 60rpx 0 40rpx;
	display: flex;
	justify-content: center;
	background: #ffffff;
	margin-bottom: 20rpx;
}

.avatar-wrapper {
	position: relative;
	width: 200rpx;
	height: 200rpx;
	border-radius: 50%;
	overflow: hidden;
	border: 6rpx solid #f0f0f0;
}

.avatar-image {
	width: 100%;
	height: 100%;
}

.avatar-overlay {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.5);
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	opacity: 0;
	transition: opacity 0.3s;
}

.avatar-wrapper:active .avatar-overlay {
	opacity: 1;
}

.avatar-icon {
	font-size: 48rpx;
	color: #ffffff;
	margin-bottom: 8rpx;
}

.avatar-text {
	font-size: 24rpx;
	color: #ffffff;
}

/* 表单区域 */
.form-section {
	background: #ffffff;
	padding: 0 32rpx;
}

.form-item {
	padding: 40rpx 0;
	border-bottom: 1rpx solid #f0f0f0;
}

.form-item:last-child {
	border-bottom: none;
}

.form-label {
	display: flex;
	align-items: center;
	margin-bottom: 24rpx;
}

.form-icon {
	font-size: 32rpx;
	color: #1b1b1b;
	margin-right: 16rpx;
	width: 32rpx;
	text-align: center;
}

.form-label-text {
	font-size: 32rpx;
	font-weight: 500;
	color: #1b1b1b;
}

.form-input {
	width: 100%;
	height: 80rpx;
	padding: 0 24rpx;
	background: #f8f8f8;
	border-radius: 16rpx;
	font-size: 30rpx;
	color: #1b1b1b;
	box-sizing: border-box;
}

.form-readonly {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 20rpx 24rpx;
	background: #f8f8f8;
	border-radius: 16rpx;
}

.readonly-text {
	font-size: 30rpx;
	color: #999999;
}

.readonly-icon {
	font-size: 24rpx;
	color: #cccccc;
}

.form-textarea {
	width: 100%;
	min-height: 160rpx;
	padding: 24rpx;
	background: #f8f8f8;
	border-radius: 16rpx;
	font-size: 30rpx;
	color: #1b1b1b;
	box-sizing: border-box;
	line-height: 1.6;
}

.char-count {
	text-align: right;
	margin-top: 12rpx;
	font-size: 24rpx;
	color: #999999;
}

/* 性别选择器 */
.gender-selector {
	display: flex;
	gap: 24rpx;
}

.gender-option {
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 32rpx 0;
	background: #f8f8f8;
	border-radius: 16rpx;
	border: 2rpx solid transparent;
	transition: all 0.3s;
}

.gender-option.gender-active {
	background: #E7FF60;
	border-color: #E7FF60;
}

.gender-icon {
	width: 64rpx;
	height: 64rpx;
	margin-bottom: 12rpx;
}

.gender-text {
	font-size: 28rpx;
	color: #1b1b1b;
	font-weight: 500;
}

/* 背景图预览 */
.banner-preview {
	position: relative;
	width: 100%;
	height: 300rpx;
	border-radius: 16rpx;
	overflow: hidden;
}

.banner-image {
	width: 100%;
	height: 100%;
}

.banner-overlay {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.4);
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	opacity: 0;
	transition: opacity 0.3s;
}

.banner-preview:active .banner-overlay {
	opacity: 1;
}

.banner-icon {
	font-size: 48rpx;
	color: #ffffff;
	margin-bottom: 8rpx;
}

.banner-text {
	font-size: 28rpx;
	color: #ffffff;
}
</style>
