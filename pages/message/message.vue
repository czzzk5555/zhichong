<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-title-wrapper">
				<text class="header-title">消息</text>
				<text class="header-unread-count" v-if="totalUnreadCount > 0">（{{ totalUnreadCount > 99 ? '99+' : totalUnreadCount }}）</text>
			</view>
		</view>

		<!-- 快速访问卡片 -->
		<view class="quick-access">
			<view class="access-card card-like" @tap="goToLikes">
				<view class="card-icon-wrapper">
					<image src="/static/images/xiaoxiye/zan.svg" class="card-icon" mode="aspectFit"></image>
					<view class="badge badge-red" v-if="likesCount > 0">{{ likesCount > 99 ? '99+' : likesCount }}</view>
				</view>
				<text class="card-text">赞与收藏</text>
			</view>
			
			<view class="access-card card-comment" @tap="goToComments">
				<view class="card-icon-wrapper">
					<image src="/static/images/xiaoxiye/pinglun.svg" class="card-icon" mode="aspectFit"></image>
				</view>
				<text class="card-text">评论与@</text>
			</view>
			
			<view class="access-card card-follow" @tap="goToFollows">
				<view class="card-icon-wrapper">
					<image src="/static/images/xiaoxiye/guanzhu.svg" class="card-icon" mode="aspectFit"></image>
					<view class="badge badge-red" v-if="followsCount > 0">{{ followsCount > 99 ? '99+' : followsCount }}</view>
				</view>
				<text class="card-text">新增关注</text>
			</view>
		</view>

		<!-- 消息列表 -->
		<scroll-view class="message-list" scroll-y>
			<!-- 系统通知 -->
			<view class="message-item" @tap="openSystemNotification">
				<view class="message-avatar system-avatar">
					<image src="/static/images/xiaoxiye/xitongtongzhi.svg" class="avatar-icon" mode="aspectFit"></image>
					<view class="badge badge-red badge-small" v-if="systemNotificationCount > 0">{{ systemNotificationCount }}</view>
				</view>
				<view class="message-content">
					<view class="message-header">
						<text class="message-title">系统通知</text>
						<text class="message-time">2026-1-12</text>
					</view>
					<text class="message-preview">恭喜你加入知宠SoulPaw!!</text>
				</view>
			</view>

			<!-- 群聊中心 -->
			<view class="message-item" @tap="openGroupChat">
				<view class="message-avatar group-avatar">
					<image src="/static/images/xiaoxiye/qunliao.svg" class="avatar-icon" mode="aspectFit"></image>
					<view class="badge badge-red badge-small" v-if="groupChatCount > 0">{{ groupChatCount }}</view>
				</view>
				<view class="message-content">
					<view class="message-header">
						<text class="message-title">群聊中心</text>
						<text class="message-time">21:37</text>
					</view>
					<text class="message-preview">甘棠村公园:有人出来遛狗吗?</text>
				</view>
			</view>

			<!-- 个人聊天列表 -->
			<view 
				class="message-item-wrapper" 
				:class="{ 'item-deleting': chat.isDeleting }"
				v-for="(chat, index) in chatList" 
				:key="index"
				:style="{ transform: chat.isDeleting ? `translateX(-100%)` : 'none', opacity: chat.isDeleting ? 0 : 1 }"
			>
				<view 
					class="message-item-slide"
					:style="{ transform: chat.isDeleting ? 'none' : `translateX(${chat.swipeOffset || 0}px)` }"
					@touchstart="onSwipeStart($event, chat, index)"
					@touchmove="onSwipeMove($event, chat, index)"
					@touchend="onSwipeEnd($event, chat, index)"
					@tap="openChat(chat)"
				>
					<view class="message-avatar">
						<image :src="chat.avatar" class="avatar-image" mode="aspectFill"></image>
						<view class="badge badge-red badge-small" v-if="chat.unreadCount > 0">{{ chat.unreadCount }}</view>
					</view>
					<view class="message-content">
						<view class="message-header">
							<text class="message-title">{{ chat.name }}</text>
							<text class="message-time">{{ chat.time }}</text>
						</view>
						<text class="message-preview">{{ chat.preview }}</text>
					</view>
				</view>
				<view 
					class="message-delete-btn" 
					:class="{ 'delete-btn-visible': chat.swipeOffset < 0 }"
					@tap="deleteChat(index)"
				>
					<text class="fa fa-trash"></text>
					<text class="delete-text">删除</text>
				</view>
			</view>
			
			<!-- 空白聊天框 -->
			<view class="message-item message-item-empty">
				<view class="message-avatar">
					<view class="avatar-placeholder"></view>
				</view>
				<view class="message-content">
					<view class="message-header">
						<text class="message-title message-title-empty"></text>
						<text class="message-time message-time-empty"></text>
					</view>
					<text class="message-preview message-preview-empty"></text>
				</view>
			</view>
		</scroll-view>

		<!-- 删除确认弹窗 -->
		<view class="delete-modal-mask" v-if="showDeleteModal" @tap="cancelDelete">
			<view class="delete-modal" @tap.stop>
				<view class="delete-modal-text">删除后所有聊天记录将被清除</view>
				<view class="delete-modal-divider"></view>
				<view class="delete-modal-btn delete-modal-confirm" @tap="confirmDelete">确认删除</view>
				<view class="delete-modal-btn-divider"></view>
				<view class="delete-modal-btn delete-modal-cancel" @tap="cancelDelete">取消</view>
			</view>
		</view>

		<!-- 底部导航 -->
		<CustomTabbar :activeIndex="activeTab" @change="handleTabChange" @publish="handlePublish" />
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
			activeTab: 3,
			likesCount: 99,
			followsCount: 50,
			systemNotificationCount: 1,
			groupChatCount: 1,
			swipeStartX: 0,
			swipeStartY: 0,
			swipingIndex: -1,
			startSwipeOffset: 0,
			showDeleteModal: false,
			deleteIndex: -1,
			chatList: [
				{
					name: '童一敏',
					avatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					preview: '',
					time: '',
					unreadCount: 3,
					swipeOffset: 0,
					messages: [
						{
							content: '你家的是什么狗狗品种呀!',
							isSent: false,
							time: '2025-01-12 14:12'
						},
						{
							content: '我家的是博美',
							isSent: true,
							time: '2025-01-12 14:14'
						},
						{
							content: '还在吗?',
							isSent: true,
							time: '2025-01-12 14:16'
						},
						{
							content: '博美很可爱呢！我也养了一只，有机会可以一起遛狗',
							isSent: false,
							time: '2025-01-12 14:18'
						},
						{
							content: '好的，没问题！',
							isSent: true,
							time: '2025-01-12 14:20'
						}
					]
				}
			]
		}
	},
	computed: {
		totalUnreadCount() {
			// 计算系统通知和群聊中心的未读数量
			let count = 0
			if (this.systemNotificationCount > 0) {
				count += this.systemNotificationCount
			}
			if (this.groupChatCount > 0) {
				count += this.groupChatCount
			}
			// 计算个人聊天列表中的未读数量
			this.chatList.forEach(chat => {
				if (chat.unreadCount && chat.unreadCount > 0) {
					count += chat.unreadCount
				}
			})
			return count
		}
	},
	mounted() {
		// 初始化时更新所有聊天项的预览信息
		this.updateChatPreviews()
	},
	onShow() {
		// 从其他页面返回时，尝试从缓存中读取最新的消息记录
		try {
			const cached = uni.getStorageSync('chat-user-temp')
			if (cached && cached.name && cached.messages) {
				// 找到对应的聊天项并更新消息记录
				const chatIndex = this.chatList.findIndex(chat => chat.name === cached.name)
				if (chatIndex !== -1) {
					// 创建新数组确保响应式更新
					const newMessages = [...cached.messages]
					// 使用 $set 确保响应式更新
					this.$set(this.chatList[chatIndex], 'messages', newMessages)
					// 立即更新预览信息
					this.updateChatPreviews()
				}
			}
		} catch (e) {
			console.error('读取聊天消息缓存失败', e)
		}
	},
	methods: {
		// 更新聊天预览信息
		updateChatPreviews() {
			this.chatList.forEach((chat, index) => {
				if (chat.messages && chat.messages.length > 0) {
					// 获取最后一条消息
					const lastMessage = chat.messages[chat.messages.length - 1]
					// 使用 $set 确保响应式更新
					this.$set(this.chatList[index], 'preview', lastMessage.content)
					this.$set(this.chatList[index], 'time', this.formatChatTime(lastMessage.time))
				}
			})
		},
		// 格式化聊天时间显示
		formatChatTime(timeStr) {
			if (!timeStr) return ''
			
			try {
				const [datePart, timePart] = timeStr.split(' ')
				const [year, month, day] = datePart.split('-')
				const [hours, minutes] = timePart.split(':')
				
				const messageDate = new Date(year, parseInt(month) - 1, day, hours, minutes)
				const now = new Date()
				
				// 计算时间差（分钟）
				const diffMinutes = Math.floor((now - messageDate) / (1000 * 60))
				
				// 如果是今天
				if (messageDate.getDate() === now.getDate() &&
				    messageDate.getMonth() === now.getMonth() &&
				    messageDate.getFullYear() === now.getFullYear()) {
					if (diffMinutes < 1) {
						return '刚刚'
					} else if (diffMinutes < 60) {
						return `${diffMinutes}分钟前`
					} else {
						return `${Math.floor(diffMinutes / 60)}小时前`
					}
				}
				
				// 如果是昨天
				const yesterday = new Date(now)
				yesterday.setDate(yesterday.getDate() - 1)
				if (messageDate.getDate() === yesterday.getDate() &&
				    messageDate.getMonth() === yesterday.getMonth() &&
				    messageDate.getFullYear() === yesterday.getFullYear()) {
					return '昨天'
				}
				
				// 如果是今年，显示月-日
				if (messageDate.getFullYear() === now.getFullYear()) {
					return `${parseInt(month)}-${parseInt(day)}`
				}
				
				// 其他情况显示年-月-日
				return `${year}-${month}-${day}`
			} catch (e) {
				return timeStr
			}
		},
		handleTabChange(index) {
			if (index === 0) {
				// 跳转到首页
				uni.reLaunch({
					url: '/pages/index/index'
				})
			} else if (index === 1) {
				uni.navigateTo({
					url: '/pages/location/location'
				})
			} else if (index === 3) {
				// 当前页面（消息），不处理
			} else if (index === 4) {
				// 跳转到“我的”
				uni.navigateTo({
					url: '/pages/profile/profile'
				})
			} else {
				this.activeTab = index
			}
		},
		handlePublish(isActive) {
			// 发布按钮点击事件，可以根据 isActive 状态处理
			if (isActive) {
				uni.showToast({ title: '发布入口，待接入', icon: 'none' })
			}
		},
		goToLikes() {
			uni.navigateTo({
				url: '/pages/message/likes-list'
			})
		},
		goToComments() {
			uni.navigateTo({
				url: '/pages/message/comments-list'
			})
		},
		goToFollows() {
			uni.navigateTo({
				url: '/pages/message/follows-list'
			})
		},
		openSystemNotification() {
			uni.navigateTo({
				url: '/pages/message/notifications-list'
			})
		},
		openGroupChat() {
			uni.navigateTo({
				url: '/pages/message/group-chat-list'
			})
		},
		openChat(chat) {
			// 如果正在滑动，不打开聊天
			if (chat.swipeOffset && chat.swipeOffset < 0) {
				return
			}
			// 清除未读消息数
			chat.unreadCount = 0
			// 缓存聊天用户信息和消息记录，跳转到聊天详情页
			try {
				// 先检查缓存中是否有更新的消息记录
				const cached = uni.getStorageSync('chat-user-temp')
				let messagesToCache = chat.messages || []
				
				// 如果缓存中有同一个人且消息数量更多，使用缓存中的消息
				if (cached && cached.name === chat.name && cached.messages && 
				    cached.messages.length > messagesToCache.length) {
					messagesToCache = cached.messages
					// 同时更新本地 chatList 中的消息记录
					const chatIndex = this.chatList.findIndex(c => c.name === chat.name)
					if (chatIndex !== -1) {
						this.$set(this.chatList[chatIndex], 'messages', [...cached.messages])
					}
				}
				
				uni.setStorageSync('chat-user-temp', {
					name: chat.name,
					avatar: chat.avatar,
					messages: messagesToCache
				})
			} catch (e) {
				console.error('缓存聊天用户信息失败', e)
			}
			uni.navigateTo({
				url: '/pages/chat/chat-detail'
			})
		},
		onSwipeStart(e, chat, index) {
			// 关闭其他已打开的滑动项
			this.chatList.forEach((item, idx) => {
				if (idx !== index && item.swipeOffset < 0) {
					item.swipeOffset = 0
				}
			})
			
			this.swipeStartX = e.touches[0].clientX
			this.swipeStartY = e.touches[0].clientY
			this.swipingIndex = index
			// 记录滑动开始时的偏移量
			this.startSwipeOffset = chat.swipeOffset || 0
		},
		onSwipeMove(e, chat, index) {
			if (this.swipingIndex !== index) return
			
			const currentX = e.touches[0].clientX
			const currentY = e.touches[0].clientY
			const deltaX = currentX - this.swipeStartX
			const deltaY = Math.abs(currentY - this.swipeStartY)
			
			// 判断是否为横向滑动（横向滑动距离大于纵向滑动距离）
			if (Math.abs(deltaX) > deltaY && Math.abs(deltaX) > 10) {
				// 计算新的偏移量（基于开始时的偏移量）
				const newOffset = this.startSwipeOffset + deltaX
				// 限制滑动范围：只能向左滑动（负值）
				// 获取删除按钮宽度
				const systemInfo = uni.getSystemInfoSync()
				const rpxToPx = systemInfo.windowWidth / 750
				const maxOffset = -160 * rpxToPx
				chat.swipeOffset = Math.max(Math.min(newOffset, 0), maxOffset)
			}
		},
		onSwipeEnd(e, chat, index) {
			if (this.swipingIndex !== index) return
			
			// 删除按钮宽度 160rpx，转换为px（假设1rpx = 0.5px，根据设备不同可能不同）
			// 使用 uni.getSystemInfoSync() 获取设备信息
			const systemInfo = uni.getSystemInfoSync()
			const rpxToPx = systemInfo.windowWidth / 750 // 750是设计稿宽度
			const deleteBtnWidth = 160 * rpxToPx
			
			// 如果滑动超过一半，自动滑出；否则滑回
			if (chat.swipeOffset < -deleteBtnWidth / 2) {
				chat.swipeOffset = -deleteBtnWidth
			} else {
				chat.swipeOffset = 0
			}
			
			this.swipingIndex = -1
		},
		deleteChat(index) {
			this.deleteIndex = index
			this.showDeleteModal = true
		},
		confirmDelete() {
			if (this.deleteIndex >= 0 && this.deleteIndex < this.chatList.length) {
				// 关闭删除弹窗
				this.showDeleteModal = false
				
				// 立即重置卡片偏移（无动画），确保卡片和删除按钮作为一个整体
				const chat = this.chatList[this.deleteIndex]
				chat.swipeOffset = 0
				
				// 设置删除状态，让整个wrapper（包括卡片和删除按钮）一起往左滑出
				this.$set(chat, 'isDeleting', true)
				
				// 等待滑出动画完成后再删除（动画时间约0.3s），删除后下面的卡片会自动往上补位
				setTimeout(() => {
					this.chatList.splice(this.deleteIndex, 1)
					this.deleteIndex = -1
				}, 300)
			} else {
				this.showDeleteModal = false
				this.deleteIndex = -1
			}
		},
		cancelDelete() {
			this.showDeleteModal = false
			// 取消删除，滑回
			if (this.deleteIndex >= 0 && this.deleteIndex < this.chatList.length) {
				if (this.chatList[this.deleteIndex]) {
					this.chatList[this.deleteIndex].swipeOffset = 0
				}
			}
			this.deleteIndex = -1
		}
	}
}
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');

.page {
	min-height: 100vh;
	background: #fff;
	box-sizing: border-box;
	padding-bottom: 0rem;
}

.header {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	height: calc(var(--status-bar-height) + 88rpx);
	background: #fff;
	z-index: 999;
	display: flex;
	align-items: center;
	justify-content: center;
	padding-top: calc(var(--status-bar-height) + 10rpx);
	box-sizing: border-box;
}

.header-title-wrapper {
	display: flex;
	align-items: center;
	gap: 0;
}

.header-title {
	font-size: 1rem;
	font-weight: 500;
	color: #161616;
	line-height: 1;
}

.header-unread-count {
	font-size: 1rem;
	font-weight: 500;
	color: #161616;
	line-height: 1;
}

.quick-access {
	display: flex;
	gap: 0.75rem;
	padding: 1rem;
	padding-top: calc(var(--status-bar-height) + 2.75rem);
	background: #fff;
}

.access-card {
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 32rpx 20rpx;
	border-radius: 24rpx;
	position: relative;
}

.card-like {
	background: transparent;
}

.card-comment {
	background: transparent;
}

.card-follow {
	background: transparent;
}

.card-icon-wrapper {
	position: relative;
	width: 100rpx;
	height: 100rpx;
	margin-bottom: 16rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.card-icon {
	width: 100rpx;
	height: 100rpx;
}

.card-text {
	font-size: 0.8rem;
	color: #161616;
	font-weight: 500;
	line-height: 1.4;
}

.badge {
	position: absolute;
	top: -8rpx;
	right: -8rpx;
	width: 36rpx;
	height: 36rpx;
	background: #FF3B30;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 0;
	box-sizing: border-box;
	font-size: 22rpx;
	color: #fff;
	font-weight: 600;
	line-height: 1;
	border: 2rpx solid #fff;
}

.badge-small {
	position: absolute;
	width: 36rpx;
	height: 36rpx;
	font-size: 22rpx;
	padding: 0;
	border-radius: 50%;
	top: 2rpx;
	right: 2rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	text-align: center;
}

.badge-red {
	background: #FF3B30;
}

.message-list {
	height: calc(100vh - calc(var(--status-bar-height) + 88rpx) - 280rpx);
	padding: 0;
	background: #fff;
}

.message-item-wrapper {
	position: relative;
	overflow: hidden;
	background: #fff;
	width: 100%;
}

.message-item-wrapper.item-deleting {
	transition: transform 0.3s ease, opacity 0.3s ease;
}

.message-item-wrapper.item-deleting .message-item-slide {
	transition: none;
}

.message-item-slide {
	display: flex;
	align-items: center;
	padding: 24rpx 32rpx;
	gap: 24rpx;
	background: #fff;
	transition: transform 0.3s ease;
	position: relative;
	z-index: 1;
	will-change: transform;
}

.message-item-slide:active {
	transition: none;
}

.message-item {
	display: flex;
	align-items: center;
	padding: 24rpx 32rpx;
	gap: 24rpx;
}

.message-delete-btn {
	position: absolute;
	right: 0;
	top: 0;
	bottom: 0;
	width: 160rpx;
	background: #FF3B30;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 8rpx;
	color: #fff;
	z-index: 0;
	opacity: 0;
	transition: opacity 0.2s ease;
	pointer-events: none;
	height: 100%;
	box-sizing: border-box;
}

.message-delete-btn.delete-btn-visible {
	opacity: 1;
	pointer-events: auto;
}

.message-delete-btn .fa {
	font-size: 36rpx;
	color: #fff;
}

.delete-text {
	font-size: 24rpx;
	color: #fff;
}

.message-item-empty {
	pointer-events: none;
	opacity: 0;
	height: 70rpx;
	overflow: hidden;
}

.avatar-placeholder {
	width: 100%;
	height: 100%;
	border-radius: 50%;
	background: transparent;
}

.message-title-empty,
.message-time-empty,
.message-preview-empty {
	background: transparent;
	color: transparent;
	display: inline-block;
}

.message-avatar {
	width: 96rpx;
	height: 96rpx;
	border-radius: 50%;
	position: relative;
	flex-shrink: 0;
	display: flex;
	align-items: center;
	justify-content: center;
	background: #f5f5f5;
}

.system-avatar {
	background: transparent;
}

.group-avatar {
	background: transparent;
}

.avatar-icon {
	width: 90%;
	height: 90%;
}

.avatar-image {
	width: 90%;
	height: 90%;
	border-radius: 50%;
}

.message-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 0.18rem;
	min-width: 0;
}

.message-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16rpx;
}

.message-title {
	font-size: 0.95rem;
	color: #161616;
	font-weight: 500;
	line-height: 0.4;
	flex: 1;
}

.message-time {
	font-size: 24rpx;
	color: #999;
	line-height: 1.4;
	flex-shrink: 0;
}

.message-preview {
	font-size: 0.8rem;
	color: #666;
	line-height: 1.5;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

/* 删除确认弹窗 */
.delete-modal-mask {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.5);
	backdrop-filter: blur(4rpx);
	z-index: 9999;
	display: flex;
	align-items: flex-end;
	justify-content: center;
	animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
	from {
		opacity: 0;
	}
	to {
		opacity: 1;
	}
}

.delete-modal {
	width: 100%;
	background: #fff;
	border-radius: 1rem 1rem 0 0;
	padding: 1rem 0px;
	display: flex;
	flex-direction: column;
	align-items: center;
	animation: slideUp 0.3s ease;
}

@keyframes slideUp {
	from {
		transform: translateY(100%);
	}
	to {
		transform: translateY(0);
	}
}

.delete-modal-text {
	font-size: 0.775rem;
	color: #a3a3a3;
	margin-bottom: 1rem;
	text-align: center;
}

.delete-modal-divider {
	width: 100%;
	height: 0.03125rem;
	background: #f9f9f9;
	margin-bottom: 1rem;
}

.delete-modal-btn {
	width: 100%;
	padding: 32rpx 0;
	text-align: center;
	font-size: 32rpx;
	line-height: 1;
}

.delete-modal-confirm {
	color: #FF3B30;
	font-weight: 500;
}

.delete-modal-btn-divider {
	width: 100%;
	height: 0.5125rem;
	background: #f5f5f5;
}

.delete-modal-cancel {
	color: #313131;
	margin-top: 0.5rem;
}

</style>
