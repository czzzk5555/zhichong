<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
			</view>
			<view class="header-title">系统通知</view>
			<view class="header-right"></view>
		</view>

		<!-- 内容列表 -->
		<scroll-view class="content-list" scroll-y :show-scrollbar="false">
			<view 
				class="notification-item" 
				v-for="(item, index) in notificationsList" 
				:key="index"
				:class="{ 'unread': !item.isRead }"
				@tap="handleNotificationTap(item, index)"
			>
				<view class="notification-icon-wrapper">
					<view class="notification-icon" :class="item.type">
						<text :class="getIconClass(item.type)"></text>
					</view>
					<view class="unread-dot" v-if="!item.isRead"></view>
				</view>
				<view class="notification-content">
					<view class="content-header">
						<text class="notification-title">{{ item.title }}</text>
						<text class="time-text">{{ formatTime(item.time) }}</text>
					</view>
					<text class="notification-text">{{ item.content }}</text>
					<view class="notification-action" v-if="item.actionText">
						<text class="action-text">{{ item.actionText }}</text>
					</view>
				</view>
			</view>
			
			<!-- 空状态 -->
			<view v-if="notificationsList.length === 0" class="empty-state">
				<text class="fa fa-bell empty-icon"></text>
				<text class="empty-text">还没有系统通知</text>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			notificationsList: [
				{
					type: 'welcome',
					title: '欢迎加入',
					content: '恭喜你加入知宠SoulPaw！在这里你可以分享宠物日常，结识更多爱宠人士。',
					actionText: '',
					isRead: false,
					time: Date.now() - 2 * 24 * 60 * 60 * 1000 // 2天前
				},
				{
					type: 'system',
					title: '系统维护',
					content: '系统将于今晚22:00-24:00进行维护升级，期间可能无法正常使用，敬请谅解。',
					actionText: '',
					isRead: false,
					time: Date.now() - 5 * 24 * 60 * 60 * 1000 // 5天前
				},
				{
					type: 'activity',
					title: '活动通知',
					content: '「宠物摄影大赛」正在火热进行中，上传你的宠物照片，有机会获得精美礼品！',
					actionText: '立即参与',
					isRead: true,
					time: Date.now() - 7 * 24 * 60 * 60 * 1000 // 7天前
				},
				{
					type: 'update',
					title: '版本更新',
					content: '新版本v2.0.0已发布，新增了更多有趣的功能，快来体验吧！',
					actionText: '立即更新',
					isRead: true,
					time: Date.now() - 10 * 24 * 60 * 60 * 1000 // 10天前
				},
				{
					type: 'reward',
					title: '奖励通知',
					content: '恭喜你获得「活跃用户」徽章，感谢你的积极参与！',
					actionText: '',
					isRead: true,
					time: Date.now() - 15 * 24 * 60 * 60 * 1000 // 15天前
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
		getIconClass(type) {
			const iconMap = {
				'welcome': 'fa fa-gift',
				'system': 'fa fa-cog',
				'activity': 'fa fa-star',
				'update': 'fa fa-download',
				'reward': 'fa fa-trophy'
			}
			return iconMap[type] || 'fa fa-bell'
		},
		handleNotificationTap(item, index) {
			// 标记为已读
			if (!item.isRead) {
				this.$set(this.notificationsList[index], 'isRead', true)
			}
			
			// 根据类型执行不同操作
			if (item.actionText) {
				uni.showToast({
					title: item.actionText,
					icon: 'none'
				})
			}
		},
		formatTime(timestamp) {
			const now = Date.now()
			const diff = now - timestamp
			const days = Math.floor(diff / 86400000)
			
			if (days === 0) {
				return '今天'
			} else if (days === 1) {
				return '昨天'
			} else if (days < 7) {
				return days + '天前'
			} else {
				const date = new Date(timestamp)
				return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
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
	background: #f5f5f5;
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

.notification-item {
	display: flex;
	align-items: flex-start;
	padding: 32rpx;
	background: #ffffff;
	margin-bottom: 12rpx;
	gap: 24rpx;
}

.notification-item.unread {
	background: #f9f9f9;
}

.notification-icon-wrapper {
	position: relative;
	flex-shrink: 0;
}

.notification-icon {
	width: 88rpx;
	height: 88rpx;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 40rpx;
	color: #ffffff;
}

.notification-icon.welcome {
	background: linear-gradient(135deg, #FF6B6B 0%, #FF8E8E 100%);
}

.notification-icon.system {
	background: linear-gradient(135deg, #4ECDC4 0%, #6EDDD6 100%);
}

.notification-icon.activity {
	background: linear-gradient(135deg, #FFD93D 0%, #FFE66D 100%);
}

.notification-icon.update {
	background: linear-gradient(135deg, #95E1D3 0%, #B5E8DD 100%);
}

.notification-icon.reward {
	background: linear-gradient(135deg, #F38181 0%, #F5A3A3 100%);
}

.unread-dot {
	position: absolute;
	top: -4rpx;
	right: -4rpx;
	width: 20rpx;
	height: 20rpx;
	background: #FF3B30;
	border-radius: 50%;
	border: 3rpx solid #ffffff;
}

.notification-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 12rpx;
	min-width: 0;
}

.content-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16rpx;
}

.notification-title {
	font-size: 30rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.time-text {
	font-size: 24rpx;
	color: #999;
	flex-shrink: 0;
}

.notification-text {
	font-size: 28rpx;
	color: #666;
	line-height: 1.6;
	word-break: break-all;
}

.notification-action {
	margin-top: 8rpx;
}

.action-text {
	font-size: 26rpx;
	color: #007AFF;
	font-weight: 500;
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
