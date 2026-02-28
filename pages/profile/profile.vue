<template>
	<view class="page">
		<!-- 顶部横幅 -->
		<view class="banner-section">
			<image class="banner-image" :src="bannerImage" mode="aspectFill" @tap="showBannerPreview"></image>
			<!-- 横幅上的操作按钮 -->
			<view class="banner-actions">
				<view class="banner-action-left">
					<view class="icon-wrapper" @tap="handleScan">
						<text class="fa fa-qrcode banner-icon"></text>
					</view>
				</view>
				<view class="banner-action-right">
					<view class="icon-wrapper" @tap="handleShare">
						<text class="fa fa-share-alt banner-icon"></text>
					</view>
					<view class="icon-wrapper" @tap="handleSetting">
						<text class="fa fa-gear banner-icon"></text>
					</view>
				</view>
			</view>
		</view>

		<!-- 用户头像区域 -->
		<view class="profile-avatar-section">
			<view class="profile-content-wrapper">
				<view class="profile-avatar-column">
					<view class="profile-avatar-wrapper">
						<image class="profile-avatar" src="https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=180&h=180&q=80" mode="aspectFill"></image>
					</view>
				</view>
				
				<!-- 用户信息区域 -->
				<view class="user-info-section">
					<view class="user-info-left">
						<!-- 用户名和性别 -->
						<view class="username-section">
							<text class="username">EULA</text>
							<view class="gender-icon-wrapper">
								<image class="gender-icon" src="/static/images/wode/nvhhai.svg" mode="aspectFit"></image>
							</view>
						</view>
						<!-- 知宠ID -->
						<text class="user-id">知宠ID : 123456789</text>
					</view>
					<!-- 编辑资料按钮 -->
					<view class="edit-profile-btn" @tap="handleEditProfile">
						<text class="edit-profile-text">编辑资料</text>
					</view>
				</view>
			</view>

			<!-- 个性签名 -->
			<view class="signature-section">
				<text id="user-signature" class="user-signature">我是养萨摩耶的一个博主!!</text>
			</view>
			
			<!-- 统计数据 -->
			<view class="stats-section">
				<view class="stat-item">
					<text class="stat-number">209</text>
					<text class="stat-label">关注</text>
				</view>
				<view class="stat-item">
					<text class="stat-number">10</text>
					<text class="stat-label">粉丝</text>
				</view>
				<view class="stat-item">
					<text class="stat-number">59</text>
					<text class="stat-label">获得点赞</text>
				</view>
			</view>

			<!-- 宠物卡片：左滑切换到“添加爱宠卡片” -->
			<swiper class="pet-card-swiper" :circular="false" :indicator-dots="false" :autoplay="false">
				<!-- 第一个：当前宠物卡片 -->
				<swiper-item>
					<view class="pet-card">
						<view class="pet-card-content">
							<!-- 宠物头像 -->
							<view class="pet-avatar-wrapper">
								<image class="pet-avatar" src="https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=180&h=180&q=80" mode="aspectFill"></image>
							</view>
							<!-- 宠物信息 -->
							<view class="pet-info">
								<view class="pet-name-row">
									<text class="pet-name">小小肉酱</text>
									<view class="pet-gender-age">
										<view class="pet-gender-icon">♂</view>
										<text class="pet-age">2岁</text>
									</view>
								</view>
								<view class="pet-details">
									<view class="pet-detail-item">
										<text class="pet-detail-label">品种</text>
										<text class="pet-detail-value">萨摩耶</text>
									</view>
									<view class="pet-detail-item">
										<text class="pet-detail-label">体重</text>
										<text class="pet-detail-value">7KG</text>
									</view>
									<view class="pet-detail-item">
										<text class="pet-detail-label">生日</text>
										<text class="pet-detail-value">2024.8.06</text>
									</view>
								</view>
							</view>
						</view>
						<!-- 右上角卡通狗狗 -->
						<image class="pet-card-dog" src="/static/images/wode/chongwukapian.png" mode="widthFix"></image>
						<!-- 右下方编辑按钮 -->
						<view class="pet-edit-btn" @tap="handleEditPet">
							<text class="pet-edit-text">编辑</text>
						</view>
					</view>
				</swiper-item>

				<!-- 第二个：添加爱宠卡片 -->
				<swiper-item>
					<view class="pet-add-card" @tap="handleAddPet">
						<text class="pet-add-icon">+</text>
						<text class="pet-add-text">添加爱宠卡片</text>
					</view>
				</swiper-item>
			</swiper>
		</view>

		<!-- 标签栏 -->
		<view class="tabs-section">
			<view class="tabs-container">
				<view 
					class="tab-item" 
					:class="{ 'tab-active': activeContentTab === 'notes' }"
					@tap="switchContentTab('notes')"
				>
					<text class="tab-text">笔记</text>
					<view class="tab-indicator" v-if="activeContentTab === 'notes'"></view>
				</view>
				<view 
					class="tab-item" 
					:class="{ 'tab-active': activeContentTab === 'liked' }"
					@tap="switchContentTab('liked')"
				>
					<text class="tab-text">赞过</text>
					<view class="tab-indicator" v-if="activeContentTab === 'liked'"></view>
				</view>
				<view 
					class="tab-item" 
					:class="{ 'tab-active': activeContentTab === 'favorites' }"
					@tap="switchContentTab('favorites')"
				>
					<text class="tab-text">收藏</text>
					<view class="tab-indicator" v-if="activeContentTab === 'favorites'"></view>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<view class="content-grid-section">
			<!-- 笔记：使用推荐页样式（瀑布流） -->
			<view v-if="activeContentTab === 'notes'" class="masonry">
				<view
					class="card"
					v-for="(item, index) in currentContentList"
					:key="index"
					:class="[item.size || 'medium', { 'card-visible': true }]"
					@tap="handleContentCardTap(item)"
				>
					<view class="image-wrap">
						<image :src="item.image" class="image" mode="aspectFill"></image>
						<view v-if="item.isVideo" class="play-badge">
							<text class="fa fa-play"></text>
						</view>
						<!-- 观看人数 -->
						<view class="view-count-badge">
							<text class="fa fa-eye view-icon"></text>
							<text class="view-count-number">{{ formatViewCount(item.viewCount || 0) }}</text>
						</view>
					</view>
					<view class="title" :title="item.title">{{ item.title && item.title.length > 10 ? item.title.substring(0, 10) + '...' : (item.title || '') }}</view>
					<view class="meta">
						<view class="user">
							<image :src="item.avatar || 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80'" class="avatar" mode="aspectFill"></image>
							<text class="card-username">{{ item.author && item.author.length > 6 ? item.author.substring(0, 6) + '...' : (item.author || '') }}</text>
						</view>
						<view class="likes" @tap.stop="toggleLike(item)">
							<text 
								class="fa fa-heart like-icon-recommend" 
								:class="[
									item.isLiked ? 'fa-solid' : 'fa-regular'
								]"
								:style="{ color: item.isLiked ? '#f25e5e' : '#1b1b1b' }"
							></text>
							<text class="like-num">{{ formatLikes(item.likes || 0) }}</text>
						</view>
					</view>
				</view>
			</view>
			
			<!-- 赞过和收藏：使用原网格样式 -->
			<view v-else class="content-grid">
				<view 
					class="content-card" 
					v-for="(item, index) in currentContentList" 
					:key="index"
					@tap="handleContentCardTap(item)"
				>
					<image 
						class="card-image" 
						:src="item.image" 
						mode="aspectFill"
					></image>
					<!-- 视频播放按钮 -->
					<view class="play-icon" v-if="item.isVideo">
						<text class="fa fa-play"></text>
					</view>
					<!-- 查看数 -->
					<view class="view-count">
						<text class="fa fa-eye view-icon"></text>
						<text class="view-number">{{ item.viewCount }}</text>
					</view>
				</view>
			</view>
		</view>

		<!-- 底部导航 -->
		<CustomTabbar :activeIndex="activeTab" @change="handleTabChange" @publish="handlePublish" />

		<!-- 图片预览弹窗 -->
		<view class="preview-modal" v-if="showPreview" @tap="closePreview">
			<view class="preview-content" @tap.stop>
				<image class="preview-image" :src="bannerImage" mode="aspectFit"></image>
				<view class="preview-actions">
					<view class="preview-change-btn" @tap="handleChangeBanner">
						<text class="fa fa-camera preview-btn-icon"></text>
						<text class="preview-btn-text">更换背景</text>
					</view>
					<view class="preview-close" @tap="closePreview">
						<text class="fa fa-times"></text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import CustomTabbar from '@/components/CustomTabbar.vue'

export default {
	components: {
		CustomTabbar
	},
	data() {
		return {
			activeTab: 4, // "我的"页面对应第5个tab，索引为4
			bannerImage: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=180&h=180&q=80',
			showPreview: false, // 控制预览弹窗显示
			activeContentTab: 'notes', // 当前激活的内容标签：notes(笔记)、liked(赞过)、favorites(收藏)
			notesList: [
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '周末去哪玩?带狗子去...',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 2100,
					isLiked: false,
					isVideo: false,
					size: 'tall',
					viewCount: 25
				},
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '今天天气真好',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 3580,
					isLiked: false,
					isVideo: true,
					size: 'medium',
					viewCount: 3580
				},
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '我的小可爱',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 12000,
					isLiked: true,
					isVideo: true,
					size: 'wide',
					viewCount: 12000
				},
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '日常分享',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 25,
					isLiked: false,
					isVideo: false,
					size: 'tall',
					viewCount: 25
				},
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '快乐时光',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 3580,
					isLiked: false,
					isVideo: true,
					size: 'medium',
					viewCount: 3580
				},
				{ 
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&h=400&q=80', 
					title: '美好的一天',
					author: 'EULA',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					likes: 12000,
					isLiked: true,
					isVideo: true,
					size: 'wide',
					viewCount: 12000
				}
			],
			likedList: [],
			favoritesList: []
		}
	},
	computed: {
		currentContentList() {
			if (this.activeContentTab === 'notes') {
				return this.notesList
			} else if (this.activeContentTab === 'liked') {
				return this.likedList
			} else {
				return this.favoritesList
			}
		}
	},
	methods: {
		showBannerPreview() {
			// 显示预览弹窗
			this.showPreview = true
		},
		closePreview() {
			// 关闭预览弹窗
			this.showPreview = false
		},
		handleChangeBanner() {
			// 更换背景图片
			uni.chooseImage({
				count: 1,
				sizeType: ['original', 'compressed'],
				sourceType: ['album', 'camera'],
				success: (res) => {
					const tempFilePath = res.tempFilePaths[0]
					// 这里可以上传图片到服务器，然后更新 bannerImage
					// 暂时直接使用本地路径
					this.bannerImage = tempFilePath
					this.showPreview = false // 关闭预览弹窗
					uni.showToast({
						title: '背景已更换',
						icon: 'success'
					})
				},
				fail: (err) => {
					console.error('选择图片失败', err)
					uni.showToast({
						title: '选择图片失败',
						icon: 'none'
					})
				}
			})
		},
		handleScan() {
			// 处理扫码
		},
		handleShare() {
			// 处理分享
		},
		handleSetting() {
			// 跳转到设置页面
			uni.navigateTo({
				url: '/pages/settings/settings'
			})
		},
		handleEditProfile() {
			// 处理编辑资料
		},
		handleEditPet() {
			// 处理编辑宠物信息
			uni.showToast({
				title: '编辑宠物信息',
				icon: 'none'
			})
		},
		handleAddPet() {
			// 跳转到添加爱宠卡片页面（暂时用提示代替）
			uni.showToast({
				title: '添加爱宠卡片',
				icon: 'none'
			})
		},
		handleTabChange(index) {
			if (index === 0) {
				// 跳转到首页
				uni.reLaunch({
					url: '/pages/index/index'
				})
			} else if (index === 1) {
				// 跳转到位置页面
				uni.navigateTo({
					url: '/pages/location/location'
				})
			} else if (index === 3) {
				// 跳转到消息页面
				uni.navigateTo({
					url: '/pages/message/message'
				})
			} else if (index === 4) {
				// "我的"页面，不处理
				this.activeTab = index
			} else {
				this.activeTab = index
			}
		},
		handlePublish(isActive) {
			// 发布按钮点击事件
			if (isActive) {
				uni.showToast({ title: '发布入口，待接入', icon: 'none' })
			}
		},
		switchContentTab(tab) {
			// 切换内容标签
			this.activeContentTab = tab
		},
		handleContentCardTap(item) {
			// 笔记卡片点击：跳转到笔记详情页
			try {
				// 临时把当前笔记数据存到本地，详情页从这里读取
				uni.setStorageSync('note-detail-temp', {
					...item,
					author: 'EULA'
				})
			} catch (e) {
				console.error('缓存笔记详情失败', e)
			}
			uni.navigateTo({
				url: '/pages/note/note-detail'
			})
		},
		toggleLike(item) {
			// 切换点赞状态
			item.isLiked = !item.isLiked
			if (item.isLiked) {
				item.likes = (item.likes || 0) + 1
			} else {
				item.likes = Math.max(0, (item.likes || 0) - 1)
			}
		},
		formatLikes(likes) {
			// 格式化点赞数：0-9999正常显示，超过9999用W
			if (likes >= 10000) {
				return (likes / 10000).toFixed(1) + 'w'
			}
			return likes.toString()
		},
		formatViewCount(count) {
			// 格式化观看数：0-9999正常显示，超过9999用W
			if (count >= 10000) {
				return (count / 10000).toFixed(1) + 'w'
			}
			return count.toString()
		}
	}
}
</script>

<style scoped>
.page {
	width: 100%;
	min-height: 100vh;
	background: #ffffff;
	padding-bottom: 0;
	box-sizing: border-box;
}

/* 顶部横幅区域 */
.banner-section {
	position: relative;
	width: 100%;
	height: 350rpx;
	overflow: hidden;
}

.banner-image {
	width: 100%;
	height: 100%;
}

.banner-actions {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	display: flex;
	justify-content: space-between;
	align-items: flex-start;
	padding: 40rpx 32rpx;
	box-sizing: border-box;
	z-index: 10;
}

.banner-action-left,
.banner-action-right {
	display: flex;
	gap: 24rpx;
}

.icon-wrapper {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
	background: rgba(0, 0, 0, 0.3);
	display: flex;
	align-items: center;
	justify-content: center;
	backdrop-filter: blur(10rpx);
}

.banner-icon {
	font-size: 32rpx;
	color: #fff;
}

/* 图片预览弹窗 */
.preview-modal {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.9);
	z-index: 9999;
	display: flex;
	align-items: center;
	justify-content: center;
}

.preview-content {
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	position: relative;
}

.preview-image {
	width: 100%;
	height: 70%;
	max-width: 100%;
}

.preview-actions {
	width: 100%;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	gap: 24rpx;
	padding: 40rpx 0;
}

.preview-change-btn {
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 12rpx;
	padding: 20rpx 48rpx;
	background: rgba(255, 255, 255, 0.2);
	border-radius: 50rpx;
	backdrop-filter: blur(10rpx);
	border: 2rpx solid rgba(255, 255, 255, 0.3);
}

.preview-btn-icon {
	font-size: 32rpx;
	color: #fff;
}

.preview-btn-text {
	font-size: 28rpx;
	color: #fff;
	font-weight: 500;
}

.preview-close {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
	background: rgba(255, 255, 255, 0.2);
	display: flex;
	align-items: center;
	justify-content: center;
	backdrop-filter: blur(10rpx);
	border: 2rpx solid rgba(255, 255, 255, 0.3);
}

.preview-close .fa {
	font-size: 32rpx;
	color: #fff;
}

/* 用户头像区域 */
.profile-avatar-section {
	display: flex;
	flex-direction: column;
	align-items: center;
	margin-top: -35px;
	position: relative;
	z-index: 5;
	padding: 0 45rpx;
	padding-bottom: 0;
	box-sizing: border-box;
}

.profile-content-wrapper {
	width: 100%;
	display: flex;
	align-items: flex-end;
	gap: 24rpx;
}

.profile-avatar-column {
	display: flex;
	flex-direction: column;
	align-items: center;
	flex-shrink: 0;
	width: 170rpx;
}

.profile-avatar-wrapper {
	position: relative;
	width: 170rpx;
	height: 170rpx;
}

.profile-avatar {
	width: 170rpx;
	height: 170rpx;
	border-radius: 50%;
	border: 5rpx solid #fff;
	box-sizing: border-box;
}

/* 用户信息区域 */
.user-info-section {
	flex: 1;
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-top: 24rpx;
	padding: 0;
	box-sizing: border-box;
	min-width: 0;
}

.user-info-left {
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	margin-left: 0;
	min-width: 0;
}

.username-section {
	display: flex;
	align-items: center;
	gap: 8rpx;
	margin-top: 8rpx;
}

.username {
	font-size: 36rpx;
	font-weight: 600;
	color: #161616;
}

.gender-icon-wrapper {
	width: 32rpx;
	height: 32rpx;
	border-radius: 50%;
	background: #E41672;
	display: flex;
	align-items: center;
	justify-content: center;
	overflow: hidden;
	transform: translateY(-2rpx);
}

.gender-icon {
	width: 70%;
	height: 70%;
	filter: brightness(0) invert(1);
}

.user-id {
	font-size: 24rpx;
	color: #DADADA;
	margin-top: 2rpx;
}

.user-signature {
	font-size: 28rpx;
	color: #9e9e9e;
	margin-top: 0;
	white-space: normal;
	word-break: break-all;
	text-align: left;
	width: 100%;
	overflow: visible;
}

/* 编辑资料按钮 */
.edit-profile-btn {
	padding: 12rpx 24rpx;
	background: #333;
	border-radius: 40rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
	margin-left: 16rpx;
	align-self: flex-start;
	margin-top: 24rpx;
}

.edit-profile-text {
	font-size: 24rpx;
	color: #fff;
}

/* 个性签名容器 */
.signature-section {
	width: 100%;
	padding: 0 0rpx;
	box-sizing: border-box;
	margin-top: 16rpx;
	display: flex;
	justify-content: flex-start;
}

/* 宠物信息卡片 swiper */
.pet-card-swiper {
	width: calc(100% + 90rpx);
	margin-left: -45rpx;
	margin-right: -45rpx;
	margin-top: 16rpx;
	margin-bottom: 0;
	height: 210rpx;
}

.pet-card-swiper ::v-deep swiper-item {
	height: 100%;
	display: flex;
	align-items: flex-start;
	justify-content: center;
	padding-bottom: 0;
}

.pet-add-card {
	width: 90%;
	height: 5rem;
	border-radius: 31.21875rem;
	background-color: #f5f5f5;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	color: #b3b3b3;
	margin: 1.25rem auto 0;
}

.pet-add-icon {
	font-size: 40rpx;
	line-height: 1;
	margin-bottom: 8rpx;
}

.pet-add-text {
	font-size: 26rpx;
}

/* 宠物信息卡片 */
.pet-card {
	position: relative;
	width: 90%;
	margin: 1.25rem auto 0;
	height: 5rem;
	padding: 0.65rem 0.875rem 0.75rem 0.6rem;
	box-sizing: border-box;
	background: linear-gradient(90deg, #E4FC5D 0%, #CCEB46 100%);
	border-radius: 3.125rem;
	overflow: visible; /* 允许右上角狗狗耳朵探出卡片 */
}

.pet-card-content {
	display: flex;
	align-items: center;
	gap: 1rem;
	position: relative;
	z-index: 2;
	margin-left: 0.1rem;
}

.pet-avatar-wrapper {
	width: 120rpx;
	height: 120rpx;
	flex-shrink: 0;
}

.pet-avatar {
	width: 120rpx;
	height: 120rpx;
	border-radius: 50%;
	box-sizing: border-box;
}

.pet-info {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 16rpx;
}

.pet-name-row {
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.pet-name {
	font-size: 28rpx;
	font-weight: 600;
	color: #161616;
}

.pet-gender-age {
	display: flex;
	align-items: center;
	gap: 8rpx;
	padding: 2rpx 12rpx;
	background: #4A90E2;
	border-radius: 20rpx;
}

.pet-gender-icon {
	font-size: 24rpx;
	color: #fff;
	font-weight: bold;
}

.pet-age {
	font-size: 20rpx;
	color: #fff;
}

.pet-details {
	display: flex;
	gap: 30rpx;
	align-items: flex-start;
}

.pet-detail-item {
	display: flex;
	flex-direction: column;
	gap: 6rpx;
	align-items: flex-start;
}

.pet-detail-label {
	font-size: 22rpx;
	color: #666;
	line-height: 1;
}

.pet-detail-value {
	font-size: 23rpx;
	font-weight: 500;
	color: #161616;
	line-height: 1;
	margin-top: 4rpx;
}

.pet-card-dog {
	position: absolute;
	right: 2rpx;
	top: -20rpx;
	width: 160rpx;
	height: auto;
	z-index: 1;
	pointer-events: none;
}

.pet-edit-btn {
	position: absolute;
	right: -10rpx;
	bottom: -10rpx;
	width: 70rpx;
	height: 70rpx;
	border-radius: 50%;
	background: linear-gradient(90deg, #E4FC5D 0%, #CCEB46 100%);
	border: 8rpx solid #ffffff;
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 3;
	cursor: pointer;
}

.pet-edit-text {
	font-size: 24rpx;
	color: #516300;
	font-weight: 500;
}


.stats-section {
	display: flex;
	justify-content: center;
	gap: 150rpx;
	margin-top: 40rpx;
}

.stat-item {
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 8rpx;
}

.stat-number {
	font-size: 40rpx;
	font-weight: 600;
	color: #161616;
}

.stat-label {
	font-size: 26rpx;
	color: #9e9e9e;
}

/* 标签栏 */
.tabs-section {
	width: 100%;
	padding: 0 45rpx;
	margin-top: 24rpx;
	margin-bottom: 0;
	box-sizing: border-box;
	background: #ffffff;
	position: relative;
	z-index: 1;
}

.tabs-container {
	display: flex;
	justify-content: center;
	align-items: center;
	gap: 2.875rem;
	position: relative;
}

.tab-item {
	position: relative;
	padding: 0;
	display: flex;
	flex-direction: column;
	align-items: center;
	cursor: pointer;
}

.tab-text {
	font-size: 32rpx;
	color: #9e9e9e;
	font-weight: 400;
	transition: all 0.3s ease;
}

.tab-item.tab-active .tab-text {
	color: #161616;
	font-weight: 600;
}

.tab-indicator {
	position: absolute;
	bottom: -8rpx;
	left: 50%;
	transform: translateX(-50%);
	width: 40rpx;
	height: 4rpx;
	background: #161616;
	border-radius: 2rpx;
}

/* 内容网格区域 */
.content-grid-section {
	width: 100%;
	padding: 20rpx 0;
	padding-bottom: 200rpx;
	box-sizing: border-box;
	background: #ffffff;
	position: relative;
	z-index: 1;
}

/* 笔记：瀑布流布局 */
.masonry {
	column-count: 2;
	column-gap: 12rpx;
	padding: 0 32rpx;
	padding-bottom: 0;
	column-fill: balance;
	line-height: 0;
}

.masonry > * {
	line-height: normal;
}

/* 推荐页卡片样式 */
.card {
	background: #ffffff;
	border-radius: 20rpx;
	overflow: hidden;
	display: inline-block;
	width: 100%;
	margin-bottom: 12rpx;
	break-inside: avoid;
	opacity: 1;
	box-sizing: border-box;
	margin-left: 0;
	margin-right: 0;
	margin-top: 0;
	vertical-align: top;
}

.card.card-visible {
	opacity: 1;
}

.image-wrap {
	position: relative;
	width: 100%;
	height: 320rpx;
}

.card.tall .image-wrap {
	height: 360rpx;
}

.card.medium .image-wrap {
	height: 280rpx;
}

.card.wide .image-wrap {
	height: 240rpx;
}

.image {
	width: 100%;
	height: 100%;
}

.play-badge {
	position: absolute;
	top: 12rpx;
	right: 12rpx;
	width: 36rpx;
	height: 36rpx;
	border-radius: 50%;
	aspect-ratio: 1;
	background: rgba(0, 0, 0, 0.55);
	color: #fff;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 18rpx;
	box-sizing: border-box;
}

.play-badge .fa {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	line-height: 1;
	margin-left: 2rpx;
	font-size: 16rpx;
}

/* 观看人数徽章 */
.view-count-badge {
	position: absolute;
	bottom: 12rpx;
	left: 12rpx;
	display: flex;
	align-items: center;
	gap: 6rpx;
	background: rgba(0, 0, 0, 0.5);
	border-radius: 20rpx;
	padding: 6rpx 12rpx;
	z-index: 2;
}

.view-count-badge .view-icon {
	font-size: 20rpx;
	color: #fff;
}

.view-count-number {
	font-size: 22rpx;
	color: #fff;
	font-weight: 500;
}

.title {
	font-size: 26rpx;
	color: #1b1b1b;
	padding: 16rpx 16rpx 10rpx;
	line-height: 1.3;
	min-height: 66rpx;
	box-sizing: border-box;
}

.meta {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 0 16rpx 14rpx;
}

.user {
	display: flex;
	align-items: center;
	gap: 10rpx;
}

.avatar {
	width: 42rpx;
	height: 42rpx;
	border-radius: 50%;
}

.username {
	font-size: 36rpx;
	font-weight: 600;
	color: #161616;
}

/* 卡片中的用户名 */
.card-username {
	font-size: 24rpx;
	color: #4a4a4a;
	font-weight: 500;
}

.likes {
	display: flex;
	align-items: center;
	gap: 8rpx;
	font-size: 24rpx;
	color: #a1a1a1;
	position: relative;
	z-index: 1;
}

.likes .fa {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	line-height: 1;
	transform-origin: center center;
}

.like-num {
	font-size: 21.12rpx;
	color: #a1a1a1;
}

/* 赞过和收藏：网格布局 */
.content-grid {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 8rpx;
	width: 100%;
	padding: 0 45rpx;
	box-sizing: border-box;
}

.content-card {
	position: relative;
	width: 100%;
	aspect-ratio: 0.85;
	border-radius: 16rpx;
	overflow: hidden;
	background: #f5f5f5;
}

.card-image {
	width: 100%;
	height: 100%;
	display: block;
}

.play-icon {
	position: absolute;
	top: 12rpx;
	right: 12rpx;
	width: 40rpx;
	height: 40rpx;
	background: rgba(0, 0, 0, 0.5);
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 2;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0);
}

.play-icon .fa {
	font-size: 18rpx;
	color: #ffffff;
	margin-left: 3rpx;
}

.view-count {
	position: absolute;
	bottom: 12rpx;
	left: 12rpx;
	display: flex;
	align-items: center;
	gap: 6rpx;
	background: rgba(0, 0, 0, 0.5);
	border-radius: 20rpx;
	padding: 6rpx 12rpx;
	z-index: 2;
}

.view-icon {
	font-size: 20rpx;
	color: #fff;
}

.view-number {
	font-size: 22rpx;
	color: #fff;
	font-weight: 500;
}

</style>

