<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
			</view>
			<view class="header-title">我关注的人</view>
			<view class="header-right"></view>
		</view>

		<!-- 内容列表 -->
		<scroll-view class="content-list" scroll-y :show-scrollbar="false">
			<view 
				class="follow-item" 
				v-for="(item, index) in followsList" 
				:key="index"
			>
				<view class="item-left">
					<image :src="item.avatar" class="user-avatar" mode="aspectFill"></image>
					<view class="user-info">
						<view class="user-header">
							<text class="username">{{ item.name }}</text>
							<text class="badge-official" v-if="item.isOfficial">官方</text>
						</view>
						<text class="user-desc">{{ item.description || '这个人很懒，什么都没有留下' }}</text>
						<text class="time-text">{{ formatTime(item.time) }}</text>
					</view>
				</view>
				<view 
					class="follow-btn" 
					:class="{ 'followed': item.isFollowed }"
					@tap.stop="toggleFollow(item, index)"
				>
					<text class="follow-btn-text">{{ item.isFollowed ? '已关注' : '关注' }}</text>
				</view>
			</view>
			
			<!-- 空状态 -->
			<view v-if="followsList.length === 0" class="empty-state">
				<text class="fa fa-user-plus empty-icon"></text>
				<text class="empty-text">你还没有关注任何人</text>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			// 这里暂时使用静态数据，后续可以替换为接口返回的“我关注的人”列表
			followsList: [
				{
					name: '童一敏',
					avatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					description: '宠物爱好者，喜欢分享养宠日常',
					isFollowed: true,
					isOfficial: false,
					time: Date.now() - 5 * 60 * 1000 // 5分钟前
				},
				{
					name: '小小肉酱',
					avatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					description: '专业宠物摄影师',
					isFollowed: true,
					isOfficial: false,
					time: Date.now() - 30 * 60 * 1000 // 30分钟前
				},
				{
					name: '知宠SoulPaw',
					avatar: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
					description: '官方账号，分享宠物社区最新动态',
					isFollowed: true,
					isOfficial: true,
					time: Date.now() - 2 * 60 * 60 * 1000 // 2小时前
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
		toggleFollow(item, index) {
			item.isFollowed = !item.isFollowed
			uni.showToast({
				title: item.isFollowed ? '已关注' : '取消关注',
				icon: 'none',
				duration: 1500
			})
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

.follow-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 32rpx;
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
	width: 96rpx;
	height: 96rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.user-info {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
	min-width: 0;
}

.user-header {
	display: flex;
	align-items: center;
	gap: 12rpx;
	flex-wrap: wrap;
}

.username {
	font-size: 30rpx;
	color: #1b1b1b;
	font-weight: 600;
}

.badge-official {
	font-size: 20rpx;
	color: #ffffff;
	background: linear-gradient(135deg, #E7FF60 0%, #D4F050 50%, #C8E840 100%);
	padding: 2rpx 10rpx;
	border-radius: 4rpx;
	font-weight: 500;
}

.user-desc {
	font-size: 26rpx;
	color: #666;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.time-text {
	font-size: 24rpx;
	color: #999;
}

.follow-btn {
	min-width: 120rpx;
	padding: 12rpx 24rpx;
	border-radius: 999rpx;
	border: 1rpx solid #1b1b1b;
	display: flex;
	align-items: center;
	justify-content: center;
}

.follow-btn.followed {
	background: #f5f5f5;
	border-color: #ddd;
}

.follow-btn-text {
	font-size: 26rpx;
	color: #1b1b1b;
}

.follow-btn.followed .follow-btn-text {
	color: #999;
}

.empty-state {
	padding: 120rpx 32rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 16rpx;
	color: #999;
}

.empty-icon {
	font-size: 72rpx;
	margin-bottom: 8rpx;
}

.empty-text {
	font-size: 26rpx;
	color: #999;
}
</style>

