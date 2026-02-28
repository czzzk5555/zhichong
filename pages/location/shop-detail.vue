<template>
	<view class="page">
		<!-- 自定义导航栏 -->
		<view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
			<view class="navbar-content">
				<view class="navbar-left" @tap="handleBack">
					<text class="fa fa-angle-left navbar-icon"></text>
				</view>
				<view class="navbar-title">宠物店详情</view>
				<view class="navbar-right">
					<text class="fa fa-share-alt share-icon" @tap="handleShare"></text>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<scroll-view class="scroll-content" scroll-y>
			<!-- 图片轮播 -->
			<swiper class="image-swiper" :indicator-dots="true" :autoplay="false" :circular="true">
				<swiper-item v-for="(img, index) in shopInfo.images" :key="index">
					<image :src="img" class="swiper-image" mode="aspectFill"></image>
				</swiper-item>
			</swiper>

			<!-- 基本信息 -->
			<view class="info-section">
				<view class="shop-name">{{ shopInfo.name }}</view>
				<view class="shop-meta">
					<view class="meta-item">
						<text class="fa fa-compass meta-icon"></text>
						<text class="meta-text">距离你:{{ shopInfo.distance }}</text>
					</view>
					<view class="meta-item">
						<text class="meta-label">推荐指数</text>
						<view class="meta-rating">
							<text
								v-for="(star, idx) in 5"
								:key="idx"
								class="fa fa-star meta-star"
								:class="{ 'meta-star-filled': idx < shopInfo.rating, 'meta-star-empty': idx >= shopInfo.rating }"
							></text>
							<text class="meta-rating-number">{{ shopInfo.rating }}.0</text>
						</view>
					</view>
				</view>
			</view>

			<!-- 标签 -->
			<view class="tags-section">
				<scroll-view class="tags-scroll" scroll-x :show-scrollbar="false">
					<view class="tags-container">
						<view 
							class="tag-item" 
							v-for="(tag, index) in shopInfo.tags" 
							:key="index"
						>
							<text class="tag-text">#{{ tag }}</text>
						</view>
					</view>
				</scroll-view>
			</view>

			<!-- 联系方式 -->
			<view class="contact-section">
				<view class="section-title">
					<text class="fa fa-phone title-icon"></text>
					<text class="title-text">联系方式</text>
				</view>
				<view class="contact-content">
					<view class="contact-item" @tap="makeCall">
						<text class="fa fa-phone contact-icon"></text>
						<text class="contact-text">{{ shopInfo.phone }}</text>
						<text class="fa fa-chevron-right contact-arrow"></text>
					</view>
				</view>
			</view>

			<!-- 营业时间 -->
			<view class="hours-section">
				<view class="section-title">
					<text class="fa fa-clock title-icon"></text>
					<text class="title-text">营业时间</text>
				</view>
				<view class="hours-content">
					<text class="hours-text">{{ shopInfo.businessHours }}</text>
				</view>
			</view>

			<!-- 地址信息 -->
			<view class="address-section">
				<view class="section-title">
					<text class="fa fa-location-dot title-icon"></text>
					<text class="title-text">地址</text>
				</view>
				<view class="address-content">
					<text class="address-text">{{ shopInfo.address }}</text>
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
					<text class="description-text">{{ shopInfo.description }}</text>
				</view>
			</view>

			<!-- 服务项目 -->
			<view class="services-section">
				<view class="section-title">
					<text class="fa fa-list title-icon"></text>
					<text class="title-text">服务项目</text>
				</view>
				<view class="services-grid">
					<view 
						class="service-item" 
						v-for="(service, index) in shopInfo.services" 
						:key="index"
					>
						<text class="fa service-icon" :class="service.icon"></text>
						<text class="service-text">{{ service.name }}</text>
					</view>
				</view>
			</view>

			<!-- 用户评价 -->
			<view class="reviews-section">
				<view class="section-title">
					<text class="fa fa-comments title-icon"></text>
					<text class="title-text">用户评价</text>
					<text class="reviews-count">({{ shopInfo.reviews.length }})</text>
				</view>
				<view class="reviews-list">
					<view 
						class="review-item" 
						v-for="(review, index) in shopInfo.reviews" 
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
			shopInfo: {
				name: '宠物用品专营店',
				distance: '0.8KM',
				rating: 4,
				images: [
					'https://images.unsplash.com/photo-1554118811-1e0d58224f24?auto=format&fit=crop&w=800&q=80',
					'https://images.unsplash.com/photo-1555396273-367ea4eb4db5?auto=format&fit=crop&w=800&q=80',
					'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?auto=format&fit=crop&w=800&q=80'
				],
				tags: ['商品齐全', '价格实惠', '品质保证', '服务周到'],
				phone: '020-12345678',
				businessHours: '周一至周日 09:00-21:00',
				address: '广东省广州市天河区天河北路123号',
				description: '本店是一家专业的宠物用品专营店，经营各类宠物食品、玩具、用品等。商品种类齐全，价格实惠，品质有保障。店内环境整洁，服务周到，是您为爱宠购物的理想选择。',
				services: [
					{ name: '宠物用品', icon: 'fa-shopping-bag' },
					{ name: '宠物食品', icon: 'fa-bowl-food' },
					{ name: '宠物玩具', icon: 'fa-baseball' },
					{ name: '宠物美容', icon: 'fa-scissors' },
					{ name: '宠物寄养', icon: 'fa-house' },
					{ name: '咨询服务', icon: 'fa-headset' }
				],
				reviews: [
					{
						name: '小王',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 5,
						time: '2天前',
						content: '商品质量很好，价格也合理，店员服务态度很好，推荐！'
					},
					{
						name: '小李',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 4,
						time: '5天前',
						content: '东西很全，就是周末人有点多，建议工作日来。'
					},
					{
						name: '小张',
						avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
						rating: 5,
						time: '1周前',
						content: '经常来这里买东西，老板人很好，商品质量有保障！'
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
				this.shopInfo = {
					...this.shopInfo,
					...cached,
					// 确保有默认值
					images: cached.images || this.shopInfo.images,
					services: cached.services || this.shopInfo.services,
					reviews: cached.reviews || this.shopInfo.reviews,
					description: cached.description || this.shopInfo.description,
					phone: cached.phone || this.shopInfo.phone,
					businessHours: cached.businessHours || this.shopInfo.businessHours
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
		makeCall() {
			uni.makePhoneCall({
				phoneNumber: this.shopInfo.phone,
				success: () => {
					console.log('拨打电话成功')
				},
				fail: (err) => {
					console.error('拨打电话失败', err)
					uni.showToast({
						title: '拨打电话失败',
						icon: 'none'
					})
				}
			})
		},
		openMap() {
			uni.openLocation({
				latitude: 23.1291,
				longitude: 113.2644,
				name: this.shopInfo.name,
				address: this.shopInfo.address,
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
				data: this.shopInfo.address,
				success: () => {
					uni.showToast({
						title: '地址已复制',
						icon: 'success'
					})
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

.shop-name {
	font-size: 44rpx;
	font-weight: 700;
	color: #1b1b1b;
	margin-bottom: 12rpx;
}

.shop-meta {
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

/* 联系方式 */
.contact-section {
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

.contact-content {
	margin-top: 0;
}

.contact-item {
	display: flex;
	align-items: center;
	gap: 16rpx;
	padding: 20rpx;
	background: #f5f5f5;
	border-radius: 16rpx;
}

.contact-icon {
	font-size: 32rpx;
	color: #1b1b1b;
}

.contact-text {
	flex: 1;
	font-size: 28rpx;
	color: #1b1b1b;
}

.contact-arrow {
	font-size: 24rpx;
	color: #999999;
}

/* 营业时间 */
.hours-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.hours-content {
	margin-top: 0;
}

.hours-text {
	font-size: 28rpx;
	color: #666666;
	line-height: 1.6;
}

/* 地址信息 */
.address-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.address-content {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
	margin-top: 0;
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

/* 服务项目 */
.services-section {
	padding: 24rpx 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
}

.services-grid {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 24rpx;
	margin-top: 0;
}

.service-item {
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 12rpx;
	padding: 24rpx;
	background: #f5f5f5;
	border-radius: 16rpx;
}

.service-icon {
	font-size: 48rpx;
	color: #666666;
}

.service-text {
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
