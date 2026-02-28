<template>
	<view class="page">
		<!-- 自定义导航栏 -->
		<view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
			<view class="navbar-content">
				<view class="navbar-left" @tap="handleBack">
					<text class="fa fa-angle-left navbar-icon"></text>
				</view>
				<view class="navbar-title">添加爱宠</view>
				<view class="navbar-right" @tap="handleSave">
					<text class="save-text">保存</text>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<scroll-view class="scroll-content" scroll-y>
			<!-- 宠物头像区域 -->
			<view class="avatar-section">
				<view class="avatar-wrapper" @tap="handleChangeAvatar">
					<image v-if="petInfo.avatar" class="avatar-image" :src="petInfo.avatar" mode="aspectFill"></image>
					<view v-else class="avatar-placeholder">
						<text class="fa fa-camera placeholder-icon"></text>
						<text class="placeholder-text">添加头像</text>
					</view>
					<view class="avatar-overlay">
						<text class="fa fa-camera avatar-icon"></text>
						<text class="avatar-text">{{ petInfo.avatar ? '更换头像' : '添加头像' }}</text>
					</view>
				</view>
			</view>

			<!-- 表单区域 -->
			<view class="form-section">
				<!-- 宠物名称 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-heart form-icon"></text>
						<text class="form-label-text">宠物名称</text>
						<text class="form-required">*</text>
					</view>
					<input 
						class="form-input" 
						v-model="petInfo.name" 
						placeholder="请输入宠物名称"
						maxlength="10"
					/>
				</view>

				<!-- 性别和年龄 -->
				<view class="form-row">
					<view class="form-item form-item-half">
						<view class="form-label">
							<text class="fa fa-venus-mars form-icon"></text>
							<text class="form-label-text">性别</text>
						</view>
						<picker 
							mode="selector" 
							:range="genderOptions" 
							:value="genderIndex"
							@change="onGenderChange"
						>
							<view class="form-picker">
								<text class="picker-text">{{ petInfo.gender || '请选择' }}</text>
								<text class="fa fa-chevron-down picker-arrow"></text>
							</view>
						</picker>
					</view>
					<view class="form-item form-item-half">
						<view class="form-label">
							<text class="fa fa-calendar form-icon"></text>
							<text class="form-label-text">年龄</text>
						</view>
						<input 
							class="form-input" 
							v-model="petInfo.age" 
							placeholder="岁"
							type="number"
							maxlength="2"
						/>
					</view>
				</view>

				<!-- 品种 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-paw form-icon"></text>
						<text class="form-label-text">品种</text>
					</view>
					<input 
						class="form-input" 
						v-model="petInfo.breed" 
						placeholder="请输入品种"
						maxlength="20"
					/>
				</view>

				<!-- 体重 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-weight form-icon"></text>
						<text class="form-label-text">体重</text>
					</view>
					<view class="weight-input-wrapper">
						<input 
							class="form-input weight-input" 
							v-model="petInfo.weight" 
							placeholder="0"
							type="digit"
							maxlength="5"
						/>
						<text class="weight-unit">KG</text>
					</view>
				</view>

				<!-- 生日 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-birthday-cake form-icon"></text>
						<text class="form-label-text">生日</text>
					</view>
					<picker 
						mode="date" 
						:value="petInfo.birthday" 
						:start="startDate"
						:end="endDate"
						@change="onBirthdayChange"
					>
						<view class="form-picker">
							<text class="picker-text">{{ petInfo.birthday || '请选择生日' }}</text>
							<text class="fa fa-chevron-down picker-arrow"></text>
						</view>
					</picker>
				</view>

				<!-- 个性标签 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-tags form-icon"></text>
						<text class="form-label-text">个性标签</text>
						<text class="form-tip">(最多5个)</text>
					</view>
					<view class="tags-container">
						<view 
							class="tag-item" 
							v-for="(tag, index) in allTags" 
							:key="index"
							:class="{ 'tag-active': petInfo.tags.includes(tag) }"
							@tap="toggleTag(tag)"
						>
							<text class="tag-text">{{ tag }}</text>
						</view>
					</view>
				</view>

				<!-- 简介 -->
				<view class="form-item">
					<view class="form-label">
						<text class="fa fa-file-text form-icon"></text>
						<text class="form-label-text">简介</text>
					</view>
					<textarea 
						class="form-textarea" 
						v-model="petInfo.description" 
						placeholder="介绍一下你的爱宠吧~"
						maxlength="200"
						:auto-height="true"
					></textarea>
					<view class="char-count">{{ petInfo.description.length }}/200</view>
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
			genderOptions: ['公', '母'],
			genderIndex: 0,
			startDate: '2010-01-01',
			endDate: '',
			allTags: ['活泼', '温顺', '聪明', '粘人', '独立', '爱玩', '安静', '护主'],
			petInfo: {
				avatar: '',
				name: '',
				gender: '公',
				age: '',
				breed: '',
				weight: '',
				birthday: '',
				tags: [],
				description: ''
			}
		}
	},
	onLoad() {
		// 获取状态栏高度
		const systemInfo = uni.getSystemInfoSync()
		this.statusBarHeight = systemInfo.statusBarHeight || 0

		// 设置结束日期为今天
		this.endDate = this.getToday()
	},
	methods: {
		getToday() {
			const date = new Date()
			const year = date.getFullYear()
			const month = String(date.getMonth() + 1).padStart(2, '0')
			const day = String(date.getDate()).padStart(2, '0')
			return `${year}-${month}-${day}`
		},
		handleBack() {
			uni.navigateBack()
		},
		handleSave() {
			// 验证必填项
			if (!this.petInfo.name || !this.petInfo.name.trim()) {
				uni.showToast({
					title: '请输入宠物名称',
					icon: 'none'
				})
				return
			}

			// 保存宠物信息
			uni.showToast({
				title: '添加成功',
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
					this.petInfo.avatar = res.tempFilePaths[0]
					uni.showToast({
						title: '头像已添加',
						icon: 'success'
					})
				},
				fail: (err) => {
					console.error('选择图片失败', err)
				}
			})
		},
		onGenderChange(e) {
			this.genderIndex = e.detail.value
			this.petInfo.gender = this.genderOptions[e.detail.value]
		},
		onBirthdayChange(e) {
			this.petInfo.birthday = e.detail.value
		},
		toggleTag(tag) {
			const index = this.petInfo.tags.indexOf(tag)
			if (index > -1) {
				this.petInfo.tags.splice(index, 1)
			} else {
				if (this.petInfo.tags.length < 5) {
					this.petInfo.tags.push(tag)
				} else {
					uni.showToast({
						title: '最多选择5个标签',
						icon: 'none'
					})
				}
			}
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
	background: #f8f8f8;
}

.avatar-image {
	width: 100%;
	height: 100%;
}

.avatar-placeholder {
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.placeholder-icon {
	font-size: 64rpx;
	color: #cccccc;
	margin-bottom: 12rpx;
}

.placeholder-text {
	font-size: 24rpx;
	color: #999999;
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

.form-row {
	display: flex;
	gap: 24rpx;
}

.form-item-half {
	flex: 1;
	padding: 40rpx 0;
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

.form-required {
	font-size: 32rpx;
	color: #ff3b30;
	margin-left: 4rpx;
}

.form-tip {
	font-size: 24rpx;
	color: #999999;
	margin-left: 8rpx;
	font-weight: normal;
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

.form-picker {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 20rpx 24rpx;
	background: #f8f8f8;
	border-radius: 16rpx;
}

.picker-text {
	font-size: 30rpx;
	color: #1b1b1b;
}

.picker-text:empty::before {
	content: '请选择';
	color: #999999;
}

.picker-arrow {
	font-size: 24rpx;
	color: #1b1b1b;
}

.weight-input-wrapper {
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.weight-input {
	flex: 1;
}

.weight-unit {
	font-size: 30rpx;
	color: #999999;
	font-weight: 500;
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

/* 标签容器 */
.tags-container {
	display: flex;
	flex-wrap: wrap;
	gap: 16rpx;
}

.tag-item {
	padding: 12rpx 24rpx;
	background: #f8f8f8;
	border-radius: 40rpx;
	border: 2rpx solid transparent;
	transition: all 0.3s;
}

.tag-item.tag-active {
	background: #E7FF60;
	border-color: #E7FF60;
}

.tag-text {
	font-size: 28rpx;
	color: #1b1b1b;
}
</style>
