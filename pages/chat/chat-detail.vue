<template>
	<view class="page">
		<!-- 顶部导航栏 -->
		<view class="top-nav">
			<view class="nav-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
				<image class="nav-avatar" :src="chatUser.avatar" mode="aspectFill" />
				<text class="nav-username">{{ chatUser.name }}</text>
			</view>
			<view class="nav-right" @tap.stop="handleMore">
				<text class="fa fa-ellipsis-vertical more-icon"></text>
			</view>
		</view>

		<!-- 聊天内容区域 -->
		<scroll-view 
			class="chat-content" 
			scroll-y 
			:show-scrollbar="false"
			:scroll-top="scrollTop"
			:scroll-into-view="scrollIntoView"
		>
			<!-- 日期时间 -->
			<view class="time-divider">
				<text class="time-text">{{ currentDate }}</text>
			</view>

			<!-- 消息列表 -->
			<view 
				class="message-wrapper"
				v-for="(message, index) in messages"
				:key="index"
				:class="{ 'message-sent': message.isSent, 'message-received': !message.isSent }"
				:id="'msg-' + index"
			>
				<!-- 接收的消息（左侧） -->
				<template v-if="!message.isSent">
					<image class="message-avatar" :src="chatUser.avatar" mode="aspectFill" />
					<view class="message-bubble message-bubble-received">
						<text class="message-text">{{ message.content }}</text>
					</view>
				</template>

				<!-- 发送的消息（右侧） -->
				<template v-else>
					<view class="message-bubble message-bubble-sent">
						<text class="message-text">{{ message.content }}</text>
					</view>
					<image class="message-avatar" :src="currentUser.avatar" mode="aspectFill" />
				</template>
			</view>

			<!-- 时间戳（2分钟前）- 显示在最后一条消息后 -->
			<view class="time-divider" v-if="showTimeAgo && messages.length > 0">
				<text class="time-text">{{ timeAgo }}</text>
			</view>
		</scroll-view>

		<!-- 底部输入栏 -->
		<view class="bottom-input-bar">
			<input 
				class="input-field" 
				type="text" 
				placeholder="发消息..." 
				placeholder-style="color: #999;"
				v-model="inputText"
				@confirm="sendMessage"
			/>
			<view class="input-actions">
				<text class="fa fa-volume-high action-icon" @tap="handleVoice"></text>
				<text class="fa fa-plus action-icon" @tap="handleAdd"></text>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			chatUser: {
				name: '小小肉酱',
				avatar: 'https://images.unsplash.com/photo-1524504388940-b1d0cf377fde?auto=format&fit=facearea&w=96&h=96&q=80'
			},
			currentUser: {
				name: '我',
				avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80'
			},
			messages: [
				{
					content: '你家的是什么狗狗品种呀!',
					isSent: false,
					time: '2025-09-09 14:12'
				},
				{
					content: '我家的是博美',
					isSent: true,
					time: '2025-09-09 14:14'
				},
				{
					content: '还在吗?',
					isSent: true,
					time: '2025-09-09 14:16'
				},
				{
					content: '打我打我大瓦房色鬼广东人个人的个人的高度认同合景泰富怪怪的喝点。',
					isSent: false,
					time: '2025-09-09 14:18'
				}
			],
			inputText: '',
			scrollTop: 0,
			scrollIntoView: '',
			currentDate: '2025-09-09 14.12',
			timeAgo: '',
			showTimeAgo: false
		}
	},
	onLoad(options) {
		// 从路由参数或缓存中获取聊天用户信息和消息记录
		try {
			const cached = uni.getStorageSync('chat-user-temp')
			if (cached) {
				this.chatUser = Object.assign({}, this.chatUser, {
					name: cached.name,
					avatar: cached.avatar
				})
				// 如果有消息记录，使用传递过来的消息
				if (cached.messages && cached.messages.length > 0) {
					this.messages = cached.messages
					// 更新当前日期为最后一条消息的日期
					if (this.messages.length > 0) {
						const lastMessage = this.messages[this.messages.length - 1]
						if (lastMessage.time) {
							this.currentDate = lastMessage.time.split(' ')[0].replace(/-/g, '.')
							// 计算"X分钟前"
							this.calculateTimeAgo(lastMessage.time)
						}
					}
				}
			}
		} catch (e) {
			console.error('加载聊天用户信息失败', e)
		}

		// 滚动到底部
		this.$nextTick(() => {
			this.scrollToBottom()
		})
	},
	methods: {
		handleBack() {
			uni.navigateBack({
				delta: 1
			})
		},
		handleMore() {
			// 显示更多选项菜单
			uni.showToast({
				title: '更多功能待接入',
				icon: 'none'
			})
		},
		sendMessage() {
			if (!this.inputText.trim()) {
				return
			}

			// 添加新消息
			const currentTime = this.getCurrentTime()
			this.messages.push({
				content: this.inputText.trim(),
				isSent: true,
				time: currentTime
			})

			// 发送新消息后，隐藏"X分钟前"（因为新消息就是最新的）
			this.showTimeAgo = false
			this.timeAgo = ''

			// 清空输入框
			this.inputText = ''

			// 滚动到底部
			this.$nextTick(() => {
				this.scrollToBottom()
			})
		},
		handleVoice() {
			uni.showToast({
				title: '语音功能待接入',
				icon: 'none'
			})
		},
		handleAdd() {
			uni.showToast({
				title: '添加功能待接入',
				icon: 'none'
			})
		},
		scrollToBottom() {
			// 滚动到底部
			const lastIndex = this.messages.length - 1
			if (lastIndex >= 0) {
				this.scrollIntoView = 'msg-' + lastIndex
				this.$nextTick(() => {
					this.scrollIntoView = ''
				})
			}
		},
		getCurrentTime() {
			const now = new Date()
			const year = now.getFullYear()
			const month = String(now.getMonth() + 1).padStart(2, '0')
			const day = String(now.getDate()).padStart(2, '0')
			const hours = String(now.getHours()).padStart(2, '0')
			const minutes = String(now.getMinutes()).padStart(2, '0')
			return `${year}-${month}-${day} ${hours}:${minutes}`
		},
		calculateTimeAgo(messageTime) {
			try {
				// 解析消息时间 (格式: "2025-01-12 14:12")
				const [datePart, timePart] = messageTime.split(' ')
				const [year, month, day] = datePart.split('-')
				const [hours, minutes] = timePart.split(':')
				
				const messageDate = new Date(year, parseInt(month) - 1, day, hours, minutes)
				const now = new Date()
				
				// 计算时间差（分钟）
				const diffMinutes = Math.floor((now - messageDate) / (1000 * 60))
				
				// 如果时间差在5分钟内且是今天，显示"X分钟前"
				if (diffMinutes >= 0 && diffMinutes < 5 && 
				    messageDate.getDate() === now.getDate() &&
				    messageDate.getMonth() === now.getMonth() &&
				    messageDate.getFullYear() === now.getFullYear()) {
					this.timeAgo = diffMinutes === 0 ? '刚刚' : `${diffMinutes}分钟前`
					this.showTimeAgo = true
				} else {
					// 超过5分钟或不是今天，不显示"X分钟前"
					this.showTimeAgo = false
					this.timeAgo = ''
				}
			} catch (e) {
				// 解析失败，不显示"X分钟前"
				this.showTimeAgo = false
				this.timeAgo = ''
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
	box-sizing: border-box;
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

/* 顶部导航栏 */
.top-nav {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	padding: calc(var(--status-bar-height) + 20rpx) 24rpx 20rpx;
	box-sizing: border-box;
	background: #ffffff;
	display: flex;
	align-items: center;
	justify-content: space-between;
	z-index: 100;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.03);
}

.nav-left {
	display: flex;
	align-items: center;
	gap: 16rpx;
	flex: 1;
}

.back-icon {
	font-size: 40rpx;
	color: #333;
}

.nav-avatar {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
}

.nav-username {
	font-size: 32rpx;
	color: #1b1b1b;
	font-weight: 500;
}

.nav-right {
	width: 56rpx;
	height: 56rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.more-icon {
	font-size: 36rpx;
	color: #333;
}

/* 聊天内容区域 */
.chat-content {
	flex: 1;
	width: 100%;
	box-sizing: border-box;
	padding-top: calc(var(--status-bar-height) + 100rpx);
	padding-bottom: 120rpx;
	padding-left: 24rpx;
	padding-right: 24rpx;
	background: #ffffff;
}

/* 时间分割线 */
.time-divider {
	display: flex;
	justify-content: center;
	margin: 32rpx 0;
}

.time-text {
	font-size: 24rpx;
	color: #999;
	background: rgba(0, 0, 0, 0.05);
	padding: 8rpx 16rpx;
	border-radius: 8rpx;
}

/* 消息容器 */
.message-wrapper {
	display: flex;
	align-items: flex-start;
	margin-bottom: 32rpx;
	gap: 16rpx;
}

.message-sent {
	justify-content: flex-end;
}

.message-received {
	justify-content: flex-start;
}

.message-avatar {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

/* 消息气泡 */
.message-bubble {
	max-width: 480rpx;
	padding: 20rpx 24rpx;
	border-radius: 16rpx;
	box-sizing: border-box;
	word-wrap: break-word;
	word-break: break-all;
}

.message-bubble-received {
	background: #f0f0f0;
	border-top-left-radius: 4rpx;
}

.message-bubble-sent {
	background: #007AFF;
	border-top-right-radius: 4rpx;
}

.message-text {
	font-size: 28rpx;
	line-height: 1.5;
	color: #1b1b1b;
}

.message-bubble-sent .message-text {
	color: #ffffff;
}

/* 底部输入栏 */
.bottom-input-bar {
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	height: 120rpx;
	padding: 20rpx 24rpx;
	box-sizing: border-box;
	background: #ffffff;
	display: flex;
	align-items: center;
	gap: 16rpx;
	box-shadow: 0 -4rpx 16rpx rgba(0, 0, 0, 0.06);
	z-index: 100;
}

.input-field {
	flex: 1;
	height: 68rpx;
	border-radius: 32rpx;
	background: #f5f5f5;
	padding: 0 26rpx;
	box-sizing: border-box;
	font-size: 28rpx;
	color: #1b1b1b;
}

.input-actions {
	display: flex;
	align-items: center;
	gap: 24rpx;
}

.action-icon {
	font-size: 40rpx;
	color: #666;
}
</style>
