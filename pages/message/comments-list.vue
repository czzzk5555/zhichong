<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
			</view>
			<view class="header-title">评论与@</view>
			<view class="header-right"></view>
		</view>

		<!-- 标签切换 -->
		<view class="tabs">
			<view 
				class="tab-item" 
				:class="{ active: currentTab === 'comments' }"
				@tap="switchTab('comments')"
			>
				<text class="tab-text">评论</text>
				<view class="tab-indicator" v-if="currentTab === 'comments'"></view>
			</view>
			<view 
				class="tab-item" 
				:class="{ active: currentTab === 'mentions' }"
				@tap="switchTab('mentions')"
			>
				<text class="tab-text">@我</text>
				<view class="tab-indicator" v-if="currentTab === 'mentions'"></view>
			</view>
		</view>

		<!-- 内容列表 -->
		<scroll-view class="content-list" scroll-y :show-scrollbar="false">
			<!-- 评论列表 -->
			<view v-if="currentTab === 'comments'" class="comments-list">
				<view 
					class="comment-item" 
					v-for="(item, index) in commentsList" 
					:key="index"
					@tap="goToNoteDetail(item)"
				>
					<image :src="item.userAvatar" class="user-avatar" mode="aspectFill"></image>
					<view class="comment-content">
						<view class="comment-header">
							<text class="username">{{ item.userName }}</text>
							<text class="time-text">{{ formatTime(item.time) }}</text>
						</view>
						<text class="comment-text">{{ item.content }}</text>
						<view class="note-info">
							<text class="note-title">笔记：{{ item.noteTitle }}</text>
						</view>
					</view>
					<image :src="item.noteImage" class="note-thumb" mode="aspectFill"></image>
				</view>
				
				<!-- 空状态 -->
				<view v-if="commentsList.length === 0" class="empty-state">
					<text class="fa fa-comment empty-icon"></text>
					<text class="empty-text">还没有评论</text>
				</view>
			</view>

			<!-- @我列表 -->
			<view v-if="currentTab === 'mentions'" class="mentions-list">
				<view 
					class="mention-item" 
					v-for="(item, index) in mentionsList" 
					:key="index"
					@tap="goToNoteDetail(item)"
				>
					<image :src="item.userAvatar" class="user-avatar" mode="aspectFill"></image>
					<view class="mention-content">
						<view class="mention-header">
							<text class="username">{{ item.userName }}</text>
							<text class="time-text">{{ formatTime(item.time) }}</text>
						</view>
						<text class="mention-text">{{ item.content }}</text>
						<view class="note-info">
							<text class="note-title">笔记：{{ item.noteTitle }}</text>
						</view>
					</view>
					<image :src="item.noteImage" class="note-thumb" mode="aspectFill"></image>
				</view>
				
				<!-- 空状态 -->
				<view v-if="mentionsList.length === 0" class="empty-state">
					<text class="fa fa-at empty-icon"></text>
					<text class="empty-text">还没有@你</text>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentTab: 'comments', // 'comments' 或 'mentions'
			commentsList: [
				{
					userName: '童一敏',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					content: '太可爱了！这是什么品种的狗狗？',
					noteTitle: '周末去哪玩？带狗子去公园撒欢',
					noteImage: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 10 * 60 * 1000, // 10分钟前
					noteId: 1
				},
				{
					userName: '小小肉酱',
					userAvatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					content: '我也想去这里！能分享一下具体位置吗？',
					noteTitle: '今天天气很Nice,每天都适合刷街啊',
					noteImage: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 1 * 60 * 60 * 1000, // 1小时前
					noteId: 2
				},
				{
					userName: 'ECHO',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					content: '好羡慕！我家狗狗也喜欢出去玩',
					noteTitle: '宝宝穿新衣服,嘴角就上扬~',
					noteImage: 'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 3 * 60 * 60 * 1000, // 3小时前
					noteId: 3
				}
			],
			mentionsList: [
				{
					userName: '糯米',
					userAvatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					content: '@EULA 这个公园在哪里？我也想去！',
					noteTitle: '周末去哪玩？带狗子去公园撒欢',
					noteImage: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 30 * 60 * 1000, // 30分钟前
					noteId: 1
				},
				{
					userName: '阳光',
					userAvatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					content: '@EULA 你的狗狗好可爱，能分享一下养狗经验吗？',
					noteTitle: '今天天气很Nice,每天都适合刷街啊',
					noteImage: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					time: Date.now() - 2 * 60 * 60 * 1000, // 2小时前
					noteId: 2
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

/* 评论列表 */
.comments-list,
.mentions-list {
	padding: 0 32rpx;
}

.comment-item,
.mention-item {
	display: flex;
	align-items: flex-start;
	justify-content: space-between;
	padding: 32rpx 0;
	border-bottom: 1rpx solid #f0f0f0;
	gap: 24rpx;
}

.user-avatar {
	width: 88rpx;
	height: 88rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.comment-content,
.mention-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 12rpx;
	min-width: 0;
}

.comment-header,
.mention-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16rpx;
}

.username {
	font-size: 28rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.time-text {
	font-size: 24rpx;
	color: #999;
	flex-shrink: 0;
}

.comment-text,
.mention-text {
	font-size: 28rpx;
	color: #333;
	line-height: 1.6;
	word-break: break-all;
}

.note-info {
	margin-top: 4rpx;
}

.note-title {
	font-size: 24rpx;
	color: #999;
}

.note-thumb {
	width: 120rpx;
	height: 120rpx;
	border-radius: 12rpx;
	flex-shrink: 0;
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
