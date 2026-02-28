<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
			</view>
			<view class="header-title">群聊中心</view>
			<view class="header-right"></view>
		</view>

		<!-- 内容列表 -->
		<scroll-view class="content-list" scroll-y :show-scrollbar="false">
			<view 
				class="group-item" 
				v-for="(item, index) in groupList" 
				:key="index"
				@tap="openGroupChat(item)"
			>
				<view class="group-avatar-wrapper">
					<image :src="item.avatar" class="group-avatar" mode="aspectFill"></image>
					<view class="badge badge-red badge-small" v-if="item.unreadCount > 0">{{ item.unreadCount > 99 ? '99+' : item.unreadCount }}</view>
				</view>
				<view class="group-content">
					<view class="group-header">
						<text class="group-name">{{ item.name }}</text>
						<text class="group-time">{{ formatTime(item.lastMessageTime) }}</text>
					</view>
					<text class="group-preview">{{ item.lastMessage }}</text>
					<view class="group-meta">
						<text class="member-count">{{ item.memberCount }}人</text>
						<text class="group-tag" v-if="item.tag">{{ item.tag }}</text>
					</view>
				</view>
			</view>
			
			<!-- 空状态 -->
			<view v-if="groupList.length === 0" class="empty-state">
				<text class="fa fa-users empty-icon"></text>
				<text class="empty-text">还没有加入群聊</text>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			groupList: [
				{
					name: '甘棠村公园',
					avatar: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=200&q=80',
					lastMessage: '有人出来遛狗吗?',
					lastMessageTime: Date.now() - 10 * 60 * 1000, // 10分钟前
					memberCount: 128,
					tag: '附近',
					unreadCount: 3,
					groupId: 1
				},
				{
					name: '宠物摄影交流',
					avatar: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=200&q=80',
					lastMessage: '今天拍的照片真好看！',
					lastMessageTime: Date.now() - 30 * 60 * 1000, // 30分钟前
					memberCount: 256,
					tag: '热门',
					unreadCount: 0,
					groupId: 2
				},
				{
					name: '博美犬交流群',
					avatar: 'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=200&q=80',
					lastMessage: '我家博美最近有点挑食',
					lastMessageTime: Date.now() - 2 * 60 * 60 * 1000, // 2小时前
					memberCount: 89,
					tag: '',
					unreadCount: 1,
					groupId: 3
				},
				{
					name: '周末遛狗小分队',
					avatar: 'https://images.unsplash.com/photo-1504595403659-9088ce801e29?auto=format&fit=crop&w=200&q=80',
					lastMessage: '这周末一起去公园吧',
					lastMessageTime: Date.now() - 5 * 60 * 60 * 1000, // 5小时前
					memberCount: 45,
					tag: '活跃',
					unreadCount: 0,
					groupId: 4
				},
				{
					name: '宠物健康咨询',
					avatar: 'https://images.unsplash.com/photo-1501004318641-b39e6451bec6?auto=format&fit=crop&w=200&q=80',
					lastMessage: '疫苗什么时候打比较好？',
					lastMessageTime: Date.now() - 24 * 60 * 60 * 1000, // 1天前
					memberCount: 312,
					tag: '官方',
					unreadCount: 0,
					groupId: 5
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
		openGroupChat(item) {
			// 清除未读消息数
			item.unreadCount = 0
			
			// 跳转到群聊详情（可以复用聊天详情页，或者创建专门的群聊页面）
			uni.showToast({
				title: `进入${item.name}群聊`,
				icon: 'none'
			})
			
			// 这里可以跳转到群聊详情页
			// uni.navigateTo({
			// 	url: `/pages/chat/group-chat-detail?groupId=${item.groupId}`
			// })
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
			} else if (days === 1) {
				return '昨天'
			} else if (days < 7) {
				return days + '天前'
			} else {
				const date = new Date(timestamp)
				return `${date.getMonth() + 1}-${date.getDate()}`
			}
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

/* 内容列表 */
.content-list {
	flex: 1;
	width: 100%;
	padding-top: calc(var(--status-bar-height) + 88rpx);
	box-sizing: border-box;
}

.group-item {
	display: flex;
	align-items: center;
	padding: 32rpx;
	border-bottom: 1rpx solid #f0f0f0;
	gap: 24rpx;
}

.group-avatar-wrapper {
	position: relative;
	flex-shrink: 0;
}

.group-avatar {
	width: 96rpx;
	height: 96rpx;
	border-radius: 12rpx;
}

.badge {
	position: absolute;
	top: -8rpx;
	right: -8rpx;
	min-width: 36rpx;
	height: 36rpx;
	background: #FF3B30;
	border-radius: 18rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 0 8rpx;
	box-sizing: border-box;
	font-size: 20rpx;
	color: #fff;
	font-weight: 600;
	line-height: 1;
	border: 3rpx solid #fff;
}

.badge-small {
	width: 36rpx;
	height: 36rpx;
	font-size: 20rpx;
	padding: 0;
	border-radius: 50%;
	top: 2rpx;
	right: 2rpx;
}

.badge-red {
	background: #FF3B30;
}

.group-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
	min-width: 0;
}

.group-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16rpx;
}

.group-name {
	font-size: 30rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.group-time {
	font-size: 24rpx;
	color: #999;
	flex-shrink: 0;
}

.group-preview {
	font-size: 28rpx;
	color: #666;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.group-meta {
	display: flex;
	align-items: center;
	gap: 16rpx;
	margin-top: 4rpx;
}

.member-count {
	font-size: 24rpx;
	color: #999;
}

.group-tag {
	font-size: 22rpx;
	color: #007AFF;
	background: #E6F2FF;
	padding: 2rpx 10rpx;
	border-radius: 4rpx;
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
