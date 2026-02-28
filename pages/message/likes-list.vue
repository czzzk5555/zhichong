<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
			</view>
			<view class="header-title">赞与收藏</view>
			<view class="header-right"></view>
		</view>

		<!-- 标签切换 -->
		<view class="tabs">
			<view 
				class="tab-item" 
				:class="{ active: currentTab === 'likes' }"
				@tap="switchTab('likes')"
			>
				<text class="tab-text">收到的赞</text>
				<view class="tab-indicator" v-if="currentTab === 'likes'"></view>
			</view>
			<view 
				class="tab-item" 
				:class="{ active: currentTab === 'favorites' }"
				@tap="switchTab('favorites')"
			>
				<text class="tab-text">收藏</text>
				<view class="tab-indicator" v-if="currentTab === 'favorites'"></view>
			</view>
		</view>

		<!-- 内容列表 -->
		<scroll-view class="content-list" scroll-y :show-scrollbar="false">
			<!-- 收到的赞列表 -->
			<view v-if="currentTab === 'likes'" class="likes-list">
				<view 
					class="like-item" 
					v-for="(item, index) in likesList" 
					:key="index"
					@tap="goToNoteDetail(item)"
				>
					<view class="item-left">
						<image :src="item.userAvatar" class="user-avatar" mode="aspectFill"></image>
						<view class="item-content">
							<view class="content-header">
								<text class="username">{{ item.userName }}</text>
								<text class="action-text">赞了你的笔记</text>
							</view>
							<text class="note-title">{{ item.noteTitle }}</text>
							<text class="time-text">{{ formatTime(item.time) }}</text>
						</view>
					</view>
					<image :src="item.noteImage" class="note-thumb" mode="aspectFill"></image>
				</view>
				
				<!-- 空状态 -->
				<view v-if="likesList.length === 0" class="empty-state">
					<text class="fa fa-heart empty-icon"></text>
					<text class="empty-text">还没有收到赞</text>
				</view>
			</view>

			<!-- 收藏列表 -->
			<view v-if="currentTab === 'favorites'" class="favorites-list">
				<view 
					class="favorite-item" 
					v-for="(item, index) in favoritesList" 
					:key="index"
					@tap="goToNoteDetail(item)"
				>
					<view class="item-left">
						<image :src="item.noteImage" class="note-image" mode="aspectFill"></image>
						<view class="item-content">
							<text class="note-title">{{ item.noteTitle }}</text>
							<view class="note-meta">
								<image :src="item.authorAvatar" class="author-avatar" mode="aspectFill"></image>
								<text class="author-name">{{ item.authorName }}</text>
								<text class="time-text">{{ formatTime(item.time) }}</text>
							</view>
						</view>
					</view>
					<view class="favorite-icon-wrapper">
						<text class="fa fa-star favorite-icon"></text>
					</view>
				</view>
				
				<!-- 空状态 -->
				<view v-if="favoritesList.length === 0" class="empty-state">
					<text class="fa fa-star empty-icon"></text>
					<text class="empty-text">还没有收藏</text>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentTab: 'likes', // 'likes' 或 'favorites'
			likesList: [
				{
					userName: '童一敏',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					noteTitle: '周末去哪玩？带狗子去公园撒欢',
					noteImage: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 5 * 60 * 1000, // 5分钟前
					noteId: 1
				},
				{
					userName: '小小肉酱',
					userAvatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					noteTitle: '今天天气很Nice,每天都适合刷街啊',
					noteImage: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 30 * 60 * 1000, // 30分钟前
					noteId: 2
				},
				{
					userName: 'ECHO',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					noteTitle: '宝宝穿新衣服,嘴角就上扬~',
					noteImage: 'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 2 * 60 * 60 * 1000, // 2小时前
					noteId: 3
				},
				{
					userName: '糯米',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					noteTitle: '调皮的汪星人',
					noteImage: 'https://images.unsplash.com/photo-1504595403659-9088ce801e29?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 24 * 60 * 60 * 1000, // 1天前
					noteId: 4
				}
			],
			favoritesList: [
				{
					noteTitle: '周末去哪玩？带狗子去公园撒欢',
					noteImage: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					authorName: '童一敏',
					authorAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					time: Date.now() - 10 * 60 * 1000, // 10分钟前
					noteId: 1
				},
				{
					noteTitle: '今天天气很Nice,每天都适合刷街啊',
					noteImage: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					authorName: '小小肉酱',
					authorAvatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					time: Date.now() - 3 * 60 * 60 * 1000, // 3小时前
					noteId: 2
				},
				{
					noteTitle: '宝宝穿新衣服,嘴角就上扬~',
					noteImage: 'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=900&q=80',
					authorName: 'ECHO',
					authorAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					time: Date.now() - 2 * 24 * 60 * 60 * 1000, // 2天前
					noteId: 3
				}
			]
		}
	},
	methods: {
		handleBack() {
			uni.navigateBack({
				delta: 1
			})
		},
		switchTab(tab) {
			this.currentTab = tab
		},
		formatTime(timestamp) {
			const now = Date.now()
			const diff = now - timestamp
			const minutes = Math.floor(diff / 60000)
			const hours = Math.floor(diff / 3600000)
			const days = Math.floor(diff / 86400000)
			
			if (minutes < 60) {
				return minutes + '分钟前'
			} else if (hours < 24) {
				return hours + '小时前'
			} else if (days < 7) {
				return days + '天前'
			} else {
				const date = new Date(timestamp)
				return `${date.getMonth() + 1}-${date.getDate()}`
			}
		},
		goToNoteDetail(item) {
			// 跳转到笔记详情页
			try {
				uni.setStorageSync('note-detail-temp', {
					title: item.noteTitle,
					image: item.noteImage,
					id: item.noteId
				})
			} catch (e) {
				console.error('缓存笔记详情失败', e)
			}
			uni.navigateTo({
				url: '/pages/note/note-detail'
			})
		}
	}
}
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');

.page {
	width: 100%;
	height: 100vh;
	background: #ffffff;
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

/* 顶部导航 */
.header {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	height: calc(var(--status-bar-height) + 88rpx);
	background: #ffffff;
	z-index: 999;
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: calc(var(--status-bar-height) + 10rpx) 32rpx 0;
	box-sizing: border-box;
	border-bottom: 1rpx solid #f0f0f0;
}

.header-left {
	width: 80rpx;
	display: flex;
	align-items: center;
}

.back-icon {
	font-size: 40rpx;
	color: #1b1b1b;
}

.header-title {
	font-size: 32rpx;
	font-weight: 600;
	color: #1b1b1b;
	flex: 1;
	text-align: center;
}

.header-right {
	width: 80rpx;
}

/* 标签切换 */
.tabs {
	position: fixed;
	top: calc(var(--status-bar-height) + 88rpx);
	left: 0;
	right: 0;
	height: 88rpx;
	background: #ffffff;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 80rpx;
	z-index: 998;
	border-bottom: 1rpx solid #f0f0f0;
}

.tab-item {
	position: relative;
	padding: 0 8rpx;
	height: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
}

.tab-text {
	font-size: 28rpx;
	color: #999;
	font-weight: 400;
	transition: color 0.3s ease;
}

.tab-item.active .tab-text {
	color: #1b1b1b;
	font-weight: 600;
}

.tab-indicator {
	position: absolute;
	bottom: 0;
	left: 50%;
	transform: translateX(-50%);
	width: 60rpx;
	height: 4rpx;
	background: #1b1b1b;
	border-radius: 2rpx;
}

/* 内容列表 */
.content-list {
	flex: 1;
	width: 100%;
	padding-top: calc(var(--status-bar-height) + 176rpx);
	box-sizing: border-box;
}

/* 收到的赞列表 */
.likes-list {
	padding: 0 32rpx;
}

.like-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 32rpx 0;
	border-bottom: 1rpx solid #f0f0f0;
	gap: 24rpx;
}

.item-left {
	flex: 1;
	display: flex;
	align-items: center;
	gap: 24rpx;
	min-width: 0;
}

.user-avatar {
	width: 88rpx;
	height: 88rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.item-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
	min-width: 0;
}

.content-header {
	display: flex;
	align-items: center;
	gap: 8rpx;
	flex-wrap: wrap;
}

.username {
	font-size: 28rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.action-text {
	font-size: 28rpx;
	color: #666;
}

.note-title {
	font-size: 26rpx;
	color: #999;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.time-text {
	font-size: 24rpx;
	color: #999;
}

.note-thumb {
	width: 120rpx;
	height: 120rpx;
	border-radius: 12rpx;
	flex-shrink: 0;
}

/* 收藏列表 */
.favorites-list {
	padding: 0 32rpx;
}

.favorite-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 32rpx 0;
	border-bottom: 1rpx solid #f0f0f0;
	gap: 24rpx;
}

.note-image {
	width: 120rpx;
	height: 120rpx;
	border-radius: 12rpx;
	flex-shrink: 0;
}

.note-meta {
	display: flex;
	align-items: center;
	gap: 12rpx;
	margin-top: 12rpx;
}

.author-avatar {
	width: 32rpx;
	height: 32rpx;
	border-radius: 50%;
}

.author-name {
	font-size: 24rpx;
	color: #666;
}

.favorite-icon-wrapper {
	width: 64rpx;
	height: 64rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.favorite-icon {
	font-size: 36rpx;
	color: #ffd700;
}

/* 空状态 */
.empty-state {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 200rpx 0;
}

.empty-icon {
	font-size: 120rpx;
	color: #e0e0e0;
	margin-bottom: 32rpx;
}

.empty-text {
	font-size: 28rpx;
	color: #999;
}
</style>
