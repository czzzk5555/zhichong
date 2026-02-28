<template>
	<view class="page">
		<!-- 自定义导航栏 -->
		<view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
			<view class="navbar-content">
				<view class="navbar-left" @tap="handleBack">
					<text class="fa fa-angle-left navbar-icon"></text>
				</view>
				<view class="navbar-title">遛宠地详情</view>
				<view class="navbar-right">
					<text class="fa fa-share-alt share-icon" @tap="handleShare"></text>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<scroll-view class="scroll-content" scroll-y>
			<!-- 图片轮播 -->
			<swiper class="image-swiper" :indicator-dots="true" :autoplay="false" :circular="true">
				<swiper-item v-for="(img, index) in parkInfo.images" :key="index">
					<image :src="img" class="swiper-image" mode="aspectFill"></image>
				</swiper-item>
			</swiper>

			<!-- 基本信息 -->
			<view class="info-section">
				<view class="info-header">
					<view>
						<view class="park-name">{{ parkInfo.name }}</view>
						<view class="park-meta">
							<view class="meta-item">
								<text class="fa fa-compass meta-icon"></text>
								<text class="meta-text">距离你:{{ parkInfo.distance }}</text>
							</view>
							<view class="meta-item">
								<text class="meta-label">推荐指数</text>
								<view class="meta-rating">
									<text
										v-for="(star, idx) in 5"
										:key="idx"
										class="fa fa-star meta-star"
										:class="{ 'meta-star-filled': idx < parkInfo.rating, 'meta-star-empty': idx >= parkInfo.rating }"
									></text>
									<text class="meta-rating-number">{{ parkInfo.rating }}.0</text>
								</view>
							</view>
						</view>
					</view>
					<view 
						class="group-btn" 
						:class="{ 'group-btn-joined': hasJoinedGroup }"
						@tap="toggleGroupJoin"
					>
						<text class="group-btn-text">
							{{ hasJoinedGroup ? '已加入群聊' : '加入群聊' }}
						</text>
					</view>
				</view>
			</view>

			<!-- 标签 -->
			<view class="tags-section">
				<scroll-view class="tags-scroll" scroll-x :show-scrollbar="false">
					<view class="tags-container">
						<view 
							class="tag-item" 
							v-for="(tag, index) in parkInfo.tags" 
							:key="index"
						>
							<text class="tag-text">#{{ tag }}</text>
						</view>
					</view>
				</scroll-view>
			</view>

			<!-- 地址信息 -->
			<view class="address-section">
				<view class="section-title">
					<text class="fa fa-location-dot title-icon"></text>
					<text class="title-text">地址</text>
				</view>
				<view class="address-content">
					<text class="address-text">{{ parkInfo.address }}</text>
					<view class="address-actions">
						<view class="action-btn" @tap="openMap">
							<text class="fa fa-map action-icon"></text>
							<text class="action-text">导航</text>
						</view>
						<view class="action-btn" @tap="copyAddress">
							<text class="fa fa-copy action-icon"></text>
							<text class="action-text">复制</text>
						</view>
					</view>
				</view>
			</view>

			<!-- 详细介绍 -->
			<view class="description-section">
				<view class="section-title">
					<text class="fa fa-file-text title-icon"></text>
					<text class="title-text">详细介绍</text>
				</view>
				<view class="description-content">
					<text class="description-text">{{ parkInfo.description }}</text>
				</view>
			</view>

			<!-- 设施服务 -->
			<view class="facilities-section">
				<view class="section-title">
					<text class="fa fa-list title-icon"></text>
					<text class="title-text">设施服务</text>
				</view>
				<view class="facilities-grid">
					<view 
						class="facility-item" 
						v-for="(facility, index) in parkInfo.facilities" 
						:key="index"
					>
						<text class="fa facility-icon" :class="facility.icon"></text>
						<text class="facility-text">{{ facility.name }}</text>
					</view>
				</view>
			</view>

			<!-- 用户评价 -->
			<view class="reviews-section">
				<view class="section-title">
					<text class="fa fa-comments title-icon"></text>
					<text class="title-text">用户评价</text>
					<text class="reviews-count">({{ parkInfo.reviews.length }})</text>
				</view>
				<view class="reviews-list">
					<view 
						class="review-item" 
						v-for="(review, index) in parkInfo.reviews" 
						:key="index"
					>
						<view class="review-header">
							<image :src="review.avatar" class="review-avatar" mode="aspectFill"></image>
							<view class="review-user">
								<text class="review-name">{{ review.name }}</text>
								<view class="review-rating">
									<text 
										v-for="(star, idx) in 5" 
										:key="idx"
										class="fa fa-star review-star"
										:class="{ 'star-filled': idx < review.rating, 'star-empty': idx >= review.rating }"
									></text>
								</view>
							</view>
							<text class="review-time">{{ review.time }}</text>
						</view>
						<text class="review-content">{{ review.content }}</text>
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
			hasJoinedGroup: false,
			parkInfo: {
				name: '甘棠村公园',
				distance: '1.3KM',
				rating: 3,
				images: [
					'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=800&q=80',
					'https://images.unsplash.com/photo-1551717743-49959800b1f6?auto=format&fit=crop&w=800&q=80',
					'https://images.unsplash.com/photo-1517849845537-4d257902454a?auto=format&fit=crop&w=800&q=80'
				],
				tags: ['小狗多', '狗友聊得来', '没有保安', '车辆少'],
				address: '广东省广州市番禺区东环街道甘棠村',
				description: '甘棠村公园是一个非常适合遛狗的地方，环境优美，设施完善。公园内有宽敞的草坪，适合狗狗自由奔跑。同时，这里聚集了很多爱狗人士，大家可以一起交流养狗心得。',
				facilities: [
					{ name: '停车场', icon: 'fa-car' },
					{ name: '饮水区', icon: 'fa-faucet' },
					{ name: '休息区', icon: 'fa-chair' },
					{ name: '垃圾桶', icon: 'fa-trash' },
					{ name: '照明设施', icon: 'fa-lightbulb' },
					{ name: '监控设备', icon: 'fa-video' }
				],
				reviews: [
					{
						name: '小王',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 5,
						time: '2天前',
						content: '非常棒的遛狗地点！我家狗狗很喜欢这里，环境干净，人也友善。'
					},
					{
						name: '小李',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 4,
						time: '5天前',
						content: '地方不错，就是周末人有点多，建议工作日来。'
					},
					{
						name: '小张',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 5,
						time: '1周前',
						content: '停车方便，环境好，狗狗玩得很开心！'
					}
				]
			}
		}
	},
	onLoad() {
		// 获取状态栏高度
		const systemInfo = uni.getSystemInfoSync()
		this.statusBarHeight = systemInfo.statusBarHeight || 0

		// 从缓存加载地点信息
		try {
			const cached = uni.getStorageSync('location-detail-temp')
			if (cached) {
				// 合并缓存的数据
				this.parkInfo = {
					...this.parkInfo,
					...cached,
					// 确保有默认值
					images: cached.images || this.parkInfo.images,
					facilities: cached.facilities || this.parkInfo.facilities,
					reviews: cached.reviews || this.parkInfo.reviews,
					description: cached.description || this.parkInfo.description
				}
			}
		} catch (e) {
			console.error('加载地点信息失败', e)
		}
	},
	methods: {
		handleBack() {
			uni.navigateBack()
		},
		handleShare() {
			uni.showToast({
				title: '分享功能待开发',
				icon: 'none'
			})
		},
		openMap() {
			uni.openLocation({
				latitude: 23.1291,
				longitude: 113.2644,
				name: this.parkInfo.name,
				address: this.parkInfo.address,
				success: () => {
					console.log('打开地图成功')
				},
				fail: (err) => {
					console.error('打开地图失败', err)
					uni.showToast({
						title: '打开地图失败',
						icon: 'none'
					})
				}
			})
		},
		copyAddress() {
			uni.setClipboardData({
				data: this.parkInfo.address,
				success: () => {
					uni.showToast({
						title: '地址已复制',
						icon: 'success'
					})
				}
			})
		},
		toggleGroupJoin() {
			// 模拟加入群聊
			if (!this.hasJoinedGroup) {
				this.hasJoinedGroup = true
				uni.showToast({
					title: '已加入群聊',
					icon: 'success'
				})
			}
		}
	}
}
</script>

<style scoped>
.page {
	width: 100%;
	min-height: 100vh;
	background: #ffffff;
}

/* 自定义导航栏 */
.custom-navbar {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background: rgba(255, 255, 255, 0.95);
	backdrop-filter: blur(10rpx);
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
	width: 60rpx;
	height: 60rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.share-icon {
	font-size: 36rpx;
	color: #1b1b1b;
}

/* 内容区域 */
.scroll-content {
	width: 100%;
	height: calc(100vh - var(--status-bar-height) - 88rpx);
	margin-top: calc(var(--status-bar-height) + 88rpx);
	padding-bottom: 40rpx;
	box-sizing: border-box;
}

/* 图片轮播 */
.image-swiper {
	width: 100%;
	height: 500rpx;
}

.swiper-image {
	width: 100%;
	height: 100%;
}

/* 基本信息 */
.info-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.info-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 24rpx;
}

.park-name {
	font-size: 44rpx;
	font-weight: 700;
	color: #1b1b1b;
	margin-bottom: 12rpx;
}

.park-meta {
	display: flex;
	flex-direction: column;
	gap: 12rpx;
}

.meta-item {
	display: flex;
	align-items: center;
	gap: 8rpx;
}

.meta-icon {
	font-size: 24rpx;
	color: #999999;
}

.meta-text {
	font-size: 28rpx;
	color: #666666;
}

.meta-label {
	font-size: 28rpx;
	color: #666666;
}

.meta-rating {
	display: flex;
	align-items: center;
	gap: 8rpx;
}

.meta-star {
	font-size: 24rpx;
	line-height: 1;
}

.meta-star-filled {
	color: #FFD700;
}

.meta-star-empty {
	color: #ddd;
}

.meta-rating-number {
	font-size: 24rpx;
	color: #999999;
}

.group-btn {
	padding: 14rpx 28rpx;
	border-radius: 999rpx;
	background: linear-gradient(135deg, #E7FF60 0%, #D4F050 50%, #C8E840 100%);
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.group-btn-text {
	font-size: 26rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.group-btn-joined {
	background: #e0e0e0;
}

/* 标签 */
.tags-section {
	padding: 0 32rpx 24rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.tags-scroll {
	width: 100%;
	white-space: nowrap;
}

.tags-container {
	display: inline-flex;
	gap: 16rpx;
	padding-right: 32rpx;
}

.tag-item {
	padding: 12rpx 24rpx;
	background: #f5f5f5;
	border-radius: 40rpx;
	flex-shrink: 0;
}

.tag-text {
	font-size: 28rpx;
	color: #1b1b1b;
}

/* 地址信息 */
.address-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.section-title {
	display: flex;
	align-items: center;
	gap: 12rpx;
	margin-bottom: 24rpx;
}

.title-icon {
	font-size: 32rpx;
	color: #1b1b1b;
}

.title-text {
	font-size: 32rpx;
	font-weight: 600;
	color: #1b1b1b;
}

.address-content {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
}

.address-text {
	font-size: 28rpx;
	color: #666666;
	line-height: 1.6;
}

.address-actions {
	display: flex;
	gap: 16rpx;
}

.action-btn {
	display: flex;
	align-items: center;
	gap: 8rpx;
	padding: 16rpx 32rpx;
	background: #f5f5f5;
	border-radius: 40rpx;
}

.action-icon {
	font-size: 24rpx;
	color: #1b1b1b;
}

.action-text {
	font-size: 28rpx;
	color: #1b1b1b;
}

/* 详细介绍 */
.description-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.description-content {
	margin-top: 0;
}

.description-text {
	font-size: 28rpx;
	color: #666666;
	line-height: 1.8;
}

/* 设施服务 */
.facilities-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.facilities-grid {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 24rpx;
	margin-top: 0;
}

.facility-item {
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 12rpx;
	padding: 24rpx;
	background: #f5f5f5;
	border-radius: 16rpx;
}

.facility-icon {
	font-size: 48rpx;
	color: #666666;
}

.facility-text {
	font-size: 24rpx;
	color: #666666;
}

/* 用户评价 */
.reviews-section {
	padding: 32rpx;
	background: #ffffff;
}

.reviews-count {
	font-size: 28rpx;
	color: #999999;
	font-weight: normal;
}

.reviews-list {
	display: flex;
	flex-direction: column;
	gap: 32rpx;
	margin-top: 0;
}

.review-item {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
	padding-bottom: 32rpx;
	border-bottom: 1rpx solid #f0f0f0;
}

.review-item:last-child {
	border-bottom: none;
	padding-bottom: 0;
}

.review-header {
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.review-avatar {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.review-user {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
}

.review-name {
	font-size: 28rpx;
	font-weight: 600;
	color: #1b1b1b;
}

.review-rating {
	display: flex;
	gap: 4rpx;
}

.review-star {
	font-size: 20rpx;
	line-height: 1;
}

.review-star.star-filled {
	color: #FFD700;
}

.review-star.star-empty {
	color: #ddd;
}

.review-time {
	font-size: 24rpx;
	color: #999999;
	flex-shrink: 0;
}

.review-content {
	font-size: 28rpx;
	color: #666666;
	line-height: 1.6;
}
</style>
