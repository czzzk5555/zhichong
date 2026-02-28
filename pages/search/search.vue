<template>
	<view class="page">
		<!-- 自定义导航栏 -->
		<view class="custom-navbar" :style="{ paddingTop: statusBarHeight + 'px' }">
			<view class="navbar-content">
				<view class="navbar-left" @tap="handleBack">
					<text class="fa fa-angle-left navbar-icon"></text>
				</view>
				<view class="search-input-wrapper">
					<text class="fa fa-magnifying-glass search-icon"></text>
					<input 
						class="search-input" 
						v-model="searchKeyword" 
						placeholder="搜一搜其它遛宠地或宠物店"
						placeholder-style="color: #999;"
						@confirm="handleSearch"
						:focus="isFocus"
					/>
					<view class="clear-btn" v-if="searchKeyword" @tap="clearSearch">
						<text class="fa fa-times clear-icon"></text>
					</view>
				</view>
			</view>
		</view>

		<!-- 内容区域 -->
		<view class="content-wrapper">
			<!-- 搜索历史 -->
			<view class="history-section" v-if="!hasSearched && searchHistory.length > 0">
				<view class="section-header">
					<text class="section-title">搜索历史</text>
					<text class="clear-history" @tap="clearHistory">清空</text>
				</view>
				<view class="history-tags">
					<view 
						class="history-tag" 
						v-for="(item, index) in searchHistory" 
						:key="index"
						@tap="selectHistory(item)"
					>
						<text class="history-text">{{ item }}</text>
					</view>
				</view>
			</view>

			<!-- 热门搜索 -->
			<view class="hot-section" v-if="!hasSearched">
				<view class="section-header">
					<text class="section-title">热门搜索</text>
				</view>
				<view class="hot-tags">
					<view 
						class="hot-tag" 
						v-for="(item, index) in hotKeywords" 
						:key="index"
						@tap="selectHotKeyword(item)"
					>
						<text class="hot-text">{{ item }}</text>
					</view>
				</view>
			</view>

			<!-- 搜索结果 -->
			<view class="result-section" v-if="hasSearched">
				<!-- 搜索结果统计 -->
				<view class="result-header">
					<text class="result-count">找到 {{ searchResults.length }} 个结果</text>
				</view>

				<!-- 结果列表 -->
				<scroll-view class="result-list" scroll-y>
					<!-- 遛宠地结果 -->
					<view 
						class="result-item"
						v-for="(item, index) in searchResults" 
						:key="index"
						@tap="viewDetail(item)"
					>
						<view class="result-image-wrapper">
							<image :src="item.image" class="result-image" mode="aspectFill"></image>
							<view class="result-type-badge" :class="item.type === 'park' ? 'type-park' : 'type-shop'">
								<text class="type-text">{{ item.type === 'park' ? '遛宠地' : '宠物店' }}</text>
							</view>
						</view>
						
						<view class="result-content">
							<view class="result-name">{{ item.name }}</view>
							<view class="result-info">
								<view class="result-distance">
									<text class="fa fa-compass distance-icon"></text>
									<text class="distance-text">距离你:{{ item.distance }}</text>
								</view>
								<view class="result-rating">
									<text class="rating-label">推荐指数:</text>
									<view class="stars">
										<text 
											v-for="(star, idx) in 5" 
											:key="idx"
											class="fa fa-star star-icon"
											:class="{ 'star-filled': idx < item.rating, 'star-empty': idx >= item.rating }"
										></text>
									</view>
								</view>
							</view>
							<view class="result-tags">
								<text 
									v-for="(tag, idx) in item.tags.slice(0, 3)" 
									:key="idx"
									class="result-tag"
								>#{{ tag }}</text>
							</view>
							<view class="result-address">
								<text class="fa fa-location-dot address-icon"></text>
								<text class="address-text">{{ item.address }}</text>
							</view>
						</view>
						
						<view class="result-arrow">
							<text class="fa fa-chevron-right arrow-icon"></text>
						</view>
					</view>

					<!-- 无结果 -->
					<view class="empty-result" v-if="searchResults.length === 0">
						<text class="fa fa-search empty-icon"></text>
						<text class="empty-text">暂无搜索结果</text>
						<text class="empty-tip">试试其他关键词吧</text>
					</view>
				</scroll-view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			statusBarHeight: 0,
			searchKeyword: '',
			isFocus: true,
			hasSearched: false,
			searchHistory: [],
			hotKeywords: ['公园', '宠物店', '宠物医院', '宠物美容', '遛狗', '天河区', '越秀区'],
			searchResults: [],
			allLocations: [] // 所有地点数据（遛宠地+宠物店）
		}
	},
	onLoad(options) {
		// 获取状态栏高度
		const systemInfo = uni.getSystemInfoSync()
		this.statusBarHeight = systemInfo.statusBarHeight || 0

		// 如果有传入的关键词，直接搜索
		if (options.keyword) {
			this.searchKeyword = decodeURIComponent(options.keyword)
			this.handleSearch()
		}

		// 加载搜索历史
		this.loadSearchHistory()

		// 加载所有地点数据（这里应该从服务器加载，暂时使用模拟数据）
		this.loadAllLocations()
	},
	methods: {
		loadSearchHistory() {
			try {
				const history = uni.getStorageSync('search-history')
				if (history && Array.isArray(history)) {
					this.searchHistory = history
				}
			} catch (e) {
				console.error('加载搜索历史失败', e)
			}
		},
		saveSearchHistory(keyword) {
			if (!keyword || !keyword.trim()) return
			
			try {
				let history = uni.getStorageSync('search-history') || []
				// 移除重复项
				history = history.filter(item => item !== keyword)
				// 添加到开头
				history.unshift(keyword)
				// 最多保存10条
				if (history.length > 10) {
					history = history.slice(0, 10)
				}
				uni.setStorageSync('search-history', history)
				this.searchHistory = history
			} catch (e) {
				console.error('保存搜索历史失败', e)
			}
		},
		clearHistory() {
			uni.showModal({
				title: '确认清空',
				content: '确定要清空所有搜索历史吗？',
				success: (res) => {
					if (res.confirm) {
						try {
							uni.removeStorageSync('search-history')
							this.searchHistory = []
						} catch (e) {
							console.error('清空搜索历史失败', e)
						}
					}
				}
			})
		},
		selectHistory(keyword) {
			this.searchKeyword = keyword
			this.handleSearch()
		},
		selectHotKeyword(keyword) {
			this.searchKeyword = keyword
			this.handleSearch()
		},
		clearSearch() {
			this.searchKeyword = ''
			this.hasSearched = false
			this.searchResults = []
		},
		handleSearch() {
			if (!this.searchKeyword || !this.searchKeyword.trim()) {
				uni.showToast({
					title: '请输入搜索关键词',
					icon: 'none'
				})
				return
			}

			// 保存搜索历史
			this.saveSearchHistory(this.searchKeyword.trim())

			// 执行搜索
			const keyword = this.searchKeyword.trim().toLowerCase()
			this.searchResults = this.allLocations.filter(item => {
				return item.name.toLowerCase().includes(keyword) ||
					item.address.toLowerCase().includes(keyword) ||
					item.tags.some(tag => tag.toLowerCase().includes(keyword))
			})

			this.hasSearched = true
		},
		loadAllLocations() {
			// 模拟数据：遛宠地
			const parks = [
				{
					name: '甘棠村公园',
					distance: '1.3KM',
					rating: 3,
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&q=80',
					tags: ['小狗多', '狗友聊得来', '没有保安', '车辆少'],
					address: '广东省广州市番禺区东环街道甘棠村',
					type: 'park'
				},
				{
					name: '创新中央公园',
					distance: '5.6KM',
					rating: 3,
					image: 'https://images.unsplash.com/photo-1551717743-49959800b1f6?auto=format&fit=crop&w=400&q=80',
					tags: ['草坪大', '可以露营', '停车免费', '封闭式场地'],
					address: '广东省广州市番禺区东环街道甘棠村',
					type: 'park'
				},
				{
					name: '天河公园',
					distance: '2.8KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1517849845537-4d257902454a?auto=format&fit=crop&w=400&q=80',
					tags: ['环境优美', '设施完善', '适合大型犬', '有宠物饮水区'],
					address: '广东省广州市天河区天河路',
					type: 'park'
				},
				{
					name: '越秀公园',
					distance: '4.2KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=400&q=80',
					tags: ['历史悠久', '绿化好', '人少安静', '适合散步'],
					address: '广东省广州市越秀区解放北路',
					type: 'park'
				},
				{
					name: '白云山公园',
					distance: '8.5KM',
					rating: 5,
					image: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=400&q=80',
					tags: ['空气清新', '视野开阔', '适合运动', '风景优美'],
					address: '广东省广州市白云区广园中路',
					type: 'park'
				}
			]

			// 模拟数据：宠物店
			const shops = [
				{
					name: '宠物用品专营店',
					distance: '0.8KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1554118811-1e0d58224f24?auto=format&fit=crop&w=400&q=80',
					tags: ['商品齐全', '价格实惠', '品质保证', '服务周到'],
					address: '广东省广州市天河区天河北路',
					type: 'shop'
				},
				{
					name: '宠物美容店',
					distance: '2.1KM',
					rating: 5,
					image: 'https://images.unsplash.com/photo-1555396273-367ea4eb4db5?auto=format&fit=crop&w=400&q=80',
					tags: ['专业美容', '设备先进', '环境舒适', '服务贴心'],
					address: '广东省广州市越秀区北京路',
					type: 'shop'
				},
				{
					name: '宠物医院',
					distance: '3.5KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?auto=format&fit=crop&w=400&q=80',
					tags: ['24小时', '专业医师', '设备完善', '口碑良好'],
					address: '广东省广州市海珠区新港东路',
					type: 'shop'
				},
				{
					name: '宠物食品店',
					distance: '4.7KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1514933651103-005eec06c04b?auto=format&fit=crop&w=400&q=80',
					tags: ['进口食品', '营养丰富', '品牌多样', '正品保证'],
					address: '广东省广州市天河区珠江新城',
					type: 'shop'
				}
			]

			this.allLocations = [...parks, ...shops]
		},
		handleBack() {
			uni.navigateBack()
		},
		viewDetail(item) {
			// 缓存地点信息
			try {
				uni.setStorageSync('location-detail-temp', item)
			} catch (e) {
				console.error('缓存地点信息失败', e)
			}

			// 根据类型跳转到不同的详情页
			if (item.type === 'park') {
				uni.navigateTo({
					url: '/pages/location/park-detail'
				})
			} else {
				uni.navigateTo({
					url: '/pages/location/shop-detail'
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
	background: #f5f5f5;
}

/* 自定义导航栏 */
.custom-navbar {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background: #ffffff;
	z-index: 1000;
	border-bottom: 1rpx solid #f0f0f0;
}

.navbar-content {
	height: 88rpx;
	display: flex;
	align-items: center;
	gap: 16rpx;
	padding: 0 32rpx;
}

.navbar-left {
	width: 60rpx;
	height: 60rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.navbar-icon {
	font-size: 40rpx;
	color: #1b1b1b;
}

.search-input-wrapper {
	flex: 1;
	display: flex;
	align-items: center;
	height: 64rpx;
	background: #f5f5f5;
	border-radius: 32rpx;
	padding: 0 24rpx;
	gap: 12rpx;
	position: relative;
}

.search-icon {
	font-size: 28rpx;
	color: #999999;
	flex-shrink: 0;
}

.search-input {
	flex: 1;
	height: 100%;
	font-size: 28rpx;
	color: #1b1b1b;
	background: transparent;
	border: none;
	outline: none;
}

.clear-btn {
	width: 40rpx;
	height: 40rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.clear-icon {
	font-size: 24rpx;
	color: #999999;
}

/* 内容区域 */
.content-wrapper {
	width: 100%;
	padding-top: calc(var(--status-bar-height) + 88rpx);
	padding-bottom: 40rpx;
	box-sizing: border-box;
}

/* 搜索历史 */
.history-section {
	padding: 32rpx;
	background: #ffffff;
	margin-bottom: 20rpx;
}

.section-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	margin-bottom: 24rpx;
}

.section-title {
	font-size: 32rpx;
	font-weight: 600;
	color: #1b1b1b;
}

.clear-history {
	font-size: 28rpx;
	color: #999999;
}

.history-tags {
	display: flex;
	flex-wrap: wrap;
	gap: 16rpx;
}

.history-tag {
	padding: 12rpx 24rpx;
	background: #f5f5f5;
	border-radius: 40rpx;
}

.history-text {
	font-size: 28rpx;
	color: #1b1b1b;
}

/* 热门搜索 */
.hot-section {
	padding: 32rpx;
	background: #ffffff;
}

.hot-tags {
	display: flex;
	flex-wrap: wrap;
	gap: 16rpx;
}

.hot-tag {
	padding: 12rpx 24rpx;
	background: #f5f5f5;
	border-radius: 40rpx;
}

.hot-text {
	font-size: 28rpx;
	color: #1b1b1b;
}

/* 搜索结果 */
.result-section {
	width: 100%;
}

.result-header {
	padding: 24rpx 32rpx;
	background: #ffffff;
	border-bottom: 1rpx solid #f0f0f0;
}

.result-count {
	font-size: 28rpx;
	color: #666666;
}

.result-list {
	width: 100%;
	height: calc(100vh - var(--status-bar-height) - 88rpx - 80rpx);
}

.result-item {
	display: flex;
	align-items: center;
	padding: 24rpx 32rpx;
	background: #ffffff;
	border-bottom: 1rpx solid #f0f0f0;
	gap: 24rpx;
}

.result-image-wrapper {
	position: relative;
	width: 200rpx;
	height: 200rpx;
	border-radius: 20rpx;
	overflow: hidden;
	flex-shrink: 0;
}

.result-image {
	width: 100%;
	height: 100%;
}

.result-type-badge {
	position: absolute;
	top: 12rpx;
	right: 12rpx;
	padding: 6rpx 12rpx;
	border-radius: 20rpx;
	background: rgba(0, 0, 0, 0.6);
}

.type-park {
	background: rgba(76, 175, 80, 0.9);
}

.type-shop {
	background: rgba(255, 152, 0, 0.9);
}

.type-text {
	font-size: 20rpx;
	color: #ffffff;
}

.result-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 12rpx;
	min-width: 0;
}

.result-name {
	font-size: 34rpx;
	font-weight: 700;
	color: #1b1b1b;
}

.result-info {
	display: flex;
	flex-direction: column;
	gap: 8rpx;
}

.result-distance {
	display: flex;
	align-items: center;
	gap: 6rpx;
}

.distance-icon {
	font-size: 20rpx;
	color: #999999;
}

.distance-text {
	font-size: 22rpx;
	color: #666666;
}

.result-rating {
	display: flex;
	align-items: center;
	gap: 10rpx;
}

.rating-label {
	font-size: 22rpx;
	color: #666666;
}

.stars {
	display: flex;
	align-items: center;
	gap: 6rpx;
}

.star-icon {
	font-size: 22rpx;
	line-height: 1;
}

.star-filled {
	color: #FFD700;
}

.star-empty {
	color: #ddd;
}

.result-tags {
	display: flex;
	flex-wrap: wrap;
	gap: 10rpx;
}

.result-tag {
	font-size: 20rpx;
	color: #666666;
	padding: 6rpx 14rpx;
	background: #f5f5f5;
	border-radius: 20rpx;
}

.result-address {
	display: flex;
	align-items: center;
	gap: 6rpx;
}

.address-icon {
	font-size: 18rpx;
	color: #999999;
	flex-shrink: 0;
}

.address-text {
	font-size: 20rpx;
	color: #999999;
	flex: 1;
}

.result-arrow {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
	background: #f5f5f5;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.arrow-icon {
	font-size: 22rpx;
	color: #999999;
}

/* 空结果 */
.empty-result {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 120rpx 0;
}

.empty-icon {
	font-size: 120rpx;
	color: #cccccc;
	margin-bottom: 24rpx;
}

.empty-text {
	font-size: 32rpx;
	color: #666666;
	margin-bottom: 12rpx;
}

.empty-tip {
	font-size: 28rpx;
	color: #999999;
}
</style>
