<template>
	<view class="page">
		<!-- 顶部导航 -->
		<view class="header">
			<view class="header-left">
				<view class="location" @tap="switchLocation">
					<text class="fa fa-location-dot location-icon"></text>
					<text class="location-text">{{ location }}</text>
				</view>
			</view>
			
			<view class="header-center">
				<view class="search-wrapper">
					<input 
						class="search-input" 
						type="text" 
						placeholder="搜一搜其它遛宠地或宠物店"
						placeholder-style="color: #999;"
						v-model="searchKeyword"
					/>
					<view class="search-btn" @tap="handleSearch">
						<text class="search-btn-text">搜索</text>
					</view>
				</view>
			</view>
		</view>

		<!-- 地图区域 - 模拟地图 -->
		<view class="map-container">
			<view class="mock-map">
				<!-- 地图背景 -->
				<view class="map-background"></view>
				
				<!-- 河流 -->
				<view class="river river-1"></view>
				<view class="river river-2"></view>
				
				<!-- 公园区域 -->
				<view class="park-area park-1">
					<text class="park-label">东坡体育公园</text>
				</view>
				<view class="park-area park-2"></view>
				<view class="park-area park-3"></view>
				
				<!-- 道路 -->
				<view class="road road-horizontal road-1"></view>
				<view class="road road-horizontal road-2"></view>
				<view class="road road-horizontal road-3"></view>
				<view class="road road-horizontal road-4"></view>
				<view class="road road-vertical road-5"></view>
				<view class="road road-vertical road-6"></view>
				<view class="road road-vertical road-7"></view>
				<view class="road road-vertical road-8"></view>
				
				<!-- 道路标签 -->
				<view class="road-label" style="left: 15%; top: 20%;">沙堰西一街</view>
				<view class="road-label" style="left: 15%; top: 35%;">沙堰街</view>
				<view class="road-label" style="left: 15%; top: 50%;">沙堰西二街</view>
				<view class="road-label" style="left: 15%; top: 70%;">普阳路</view>
				<view class="road-label" style="left: 5%; top: 25%;">东坡南一路</view>
				<view class="road-label" style="left: 25%; top: 15%;">沙堰东街</view>
				<view class="road-label" style="left: 75%; top: 30%;">清风街</view>
				<view class="road-label" style="left: 75%; top: 50%;">香沙街</view>
				
				<!-- 建筑物区块 -->
				<view class="building building-1"></view>
				<view class="building building-2"></view>
				<view class="building building-3"></view>
				<view class="building building-4"></view>
				<view class="building building-5"></view>
				<view class="building building-6"></view>
				<view class="building building-7"></view>
				<view class="building building-8"></view>
				
				<!-- 定位图标 -->
				<view class="location-marker">
					<view class="location-pulse"></view>
					<image 
						src="/static/images/dingwei.png" 
						class="map-location-icon"
						mode="aspectFit"
					></image>
				</view>
				
			</view>
		</view>

		<!-- 内容卡片区域 -->
		<view 
			class="content-card"
			:style="cardStyle"
		>
			<!-- 选项卡 -->
			<view 
				class="tabs" 
				:class="{ 'tabs-flipped': currentTab === 'restaurants' }"
			>
				<view 
					class="tab" 
					:class="{ active: currentTab === 'parks' }"
					@tap="switchTab('parks')"
				>
					<text class="tab-text">附近的遛宠地</text>
				</view>
				<view 
					class="tab" 
					:class="{ active: currentTab === 'restaurants' }"
					@tap="switchTab('restaurants')"
				>
					<text class="tab-text">附近的宠物店</text>
				</view>
			</view>

		<!-- 地点列表 - 遛宠地 -->
		<scroll-view 
			class="location-list" 
			scroll-y
			v-show="currentTab === 'parks'"
			:scroll-top="parksScrollTop"
			@scroll="onParksScroll"
		>
			<view 
				class="location-item"
				v-for="(item, index) in parksList"
				:key="index"
				@tap="viewDetail(item)"
			>
				<!-- 左侧缩略图 -->
				<view class="item-image-wrapper">
					<image :src="item.image" class="item-image" mode="aspectFill"></image>
				</view>
				
				<!-- 右侧信息 -->
				<view class="item-content">
					<!-- 距离 -->
					<view class="item-distance">
						<text class="fa fa-compass distance-icon"></text>
						<text class="distance-text">距离你:{{ item.distance }}</text>
					</view>
					
					<!-- 地点名称 -->
					<view class="item-name">{{ item.name }}</view>
					
					<!-- 推荐指数 -->
					<view class="item-rating">
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
					
					<!-- 标签列表 -->
					<scroll-view class="item-tags" scroll-x :show-scrollbar="false">
						<view class="tags-container">
							<text 
								v-for="(tag, idx) in item.tags" 
								:key="idx"
								class="tag"
							>#{{ tag }}</text>
						</view>
					</scroll-view>
					
					<!-- 地址 -->
					<view class="item-address">
						<text class="fa fa-location-dot address-icon"></text>
						<text class="address-text">{{ item.address }}</text>
					</view>
				</view>
				
				<!-- 右侧箭头 -->
				<view class="item-arrow">
					<text class="fa fa-chevron-right arrow-icon"></text>
				</view>
			</view>
			
			<!-- 空白占位卡片 -->
			<view class="location-item-spacer"></view>
		</scroll-view>

		<!-- 地点列表 - 宠物店 -->
		<scroll-view 
			class="location-list" 
			scroll-y
			v-show="currentTab === 'restaurants'"
			:scroll-top="restaurantsScrollTop"
			@scroll="onRestaurantsScroll"
		>
			<view 
				class="location-item"
				v-for="(item, index) in restaurantsList"
				:key="index"
				@tap="viewDetail(item)"
			>
				<!-- 左侧缩略图 -->
				<view class="item-image-wrapper">
					<image :src="item.image" class="item-image" mode="aspectFill"></image>
				</view>
				
				<!-- 右侧信息 -->
				<view class="item-content">
					<!-- 距离 -->
					<view class="item-distance">
						<text class="fa fa-compass distance-icon"></text>
						<text class="distance-text">距离你:{{ item.distance }}</text>
					</view>
					
					<!-- 地点名称 -->
					<view class="item-name">{{ item.name }}</view>
					
					<!-- 推荐指数 -->
					<view class="item-rating">
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
					
					<!-- 标签列表 -->
					<scroll-view class="item-tags" scroll-x :show-scrollbar="false">
						<view class="tags-container">
							<text 
								v-for="(tag, idx) in item.tags" 
								:key="idx"
								class="tag"
							>#{{ tag }}</text>
						</view>
					</scroll-view>
					
					<!-- 地址 -->
					<view class="item-address">
						<text class="fa fa-location-dot address-icon"></text>
						<text class="address-text">{{ item.address }}</text>
					</view>
				</view>
				
				<!-- 右侧箭头 -->
				<view class="item-arrow">
					<text class="fa fa-chevron-right arrow-icon"></text>
				</view>
			</view>
			
			<!-- 空白占位卡片 -->
			<view class="location-item-spacer"></view>
		</scroll-view>
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
			location: '广州',
			activeTab: 1,
			searchKeyword: '',
			currentTab: 'parks', // 'parks' 或 'restaurants'
			parksScrollTop: 0, // 遛宠地滚动位置
			restaurantsScrollTop: 0, // 宠物店滚动位置
			cardBottom: 0, // 卡片底部位置，动态计算（单位：px）
			cardHeight: '55vh', // 卡片高度，动态计算
			// 模拟地图标记点数据
			mockMarkers: [
				{
					id: 1,
					name: '我的位置',
					type: 'current',
					left: 50,
					top: 50,
					showLabel: false
				},
				{
					id: 2,
					name: '甘棠村公园',
					type: 'park',
					left: 45,
					top: 40,
					showLabel: false
				},
				{
					id: 3,
					name: '创新中央公园',
					type: 'park',
					left: 55,
					top: 60,
					showLabel: false
				},
				{
					id: 4,
					name: '天河公园',
					type: 'park',
					left: 60,
					top: 35,
					showLabel: false
				},
				{
					id: 5,
					name: '宠物用品店',
					type: 'restaurant',
					left: 40,
					top: 55,
					showLabel: false
				},
				{
					id: 6,
					name: '宠物美容店',
					type: 'restaurant',
					left: 65,
					top: 45,
					showLabel: false
				}
			],
			parksList: [
				{
					name: '甘棠村公园',
					distance: '1.3KM',
					rating: 3,
					image: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=400&q=80',
					tags: ['小狗多', '狗友聊得来', '没有保安', '车辆少'],
					address: '广东省广州市番禺区东环街道甘棠村'
				},
				{
					name: '创新中央公园',
					distance: '5.6KM',
					rating: 3,
					image: 'https://images.unsplash.com/photo-1551717743-49959800b1f6?auto=format&fit=crop&w=400&q=80',
					tags: ['草坪大', '可以露营', '停车免费', '封闭式场地'],
					address: '广东省广州市番禺区东环街道甘棠村'
				},
				{
					name: '天河公园',
					distance: '2.8KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1517849845537-4d257902454a?auto=format&fit=crop&w=400&q=80',
					tags: ['环境优美', '设施完善', '适合大型犬', '有宠物饮水区'],
					address: '广东省广州市天河区天河路'
				},
				{
					name: '越秀公园',
					distance: '4.2KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=400&q=80',
					tags: ['历史悠久', '绿化好', '人少安静', '适合散步'],
					address: '广东省广州市越秀区解放北路'
				},
			{
				name: '白云山公园',
				distance: '8.5KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=400&q=80',
				tags: ['空气清新', '视野开阔', '适合运动', '风景优美'],
				address: '广东省广州市白云区广园中路'
			},
			{
				name: '海珠湖公园',
				distance: '6.8KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?auto=format&fit=crop&w=400&q=80',
				tags: ['湖景优美', '适合遛狗', '人少安静', '免费停车'],
				address: '广东省广州市海珠区新滘中路'
			},
			{
				name: '流花湖公园',
				distance: '3.2KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?auto=format&fit=crop&w=400&q=80',
				tags: ['环境优雅', '适合小型犬', '设施齐全', '交通便利'],
				address: '广东省广州市越秀区流花路'
			},
			{
				name: '二沙岛公园',
				distance: '5.1KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?auto=format&fit=crop&w=400&q=80',
				tags: ['江景优美', '适合大型犬', '空间宽敞', '适合拍照'],
				address: '广东省广州市越秀区二沙岛'
			},
			{
				name: '大夫山森林公园',
				distance: '12.3KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1447752875215-b2761acb3c5f?auto=format&fit=crop&w=400&q=80',
				tags: ['天然氧吧', '适合远足', '风景绝美', '适合全天游玩'],
				address: '广东省广州市番禺区禺山西路'
			},
			{
				name: '荔湾湖公园',
				distance: '4.6KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1472214103451-9374bd1c798e?auto=format&fit=crop&w=400&q=80',
				tags: ['文化底蕴', '适合散步', '环境清幽', '宠物友好'],
				address: '广东省广州市荔湾区龙津西路'
			}
		],
			restaurantsList: [
				{
					name: '宠物用品专营店',
					distance: '0.8KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1554118811-1e0d58224f24?auto=format&fit=crop&w=400&q=80',
					tags: ['商品齐全', '价格实惠', '品质保证', '服务周到'],
					address: '广东省广州市天河区天河北路'
				},
				{
					name: '宠物美容店',
					distance: '2.1KM',
					rating: 5,
					image: 'https://images.unsplash.com/photo-1555396273-367ea4eb4db5?auto=format&fit=crop&w=400&q=80',
					tags: ['专业美容', '设备先进', '环境舒适', '服务贴心'],
					address: '广东省广州市越秀区北京路'
				},
				{
					name: '宠物医院',
					distance: '3.5KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?auto=format&fit=crop&w=400&q=80',
					tags: ['24小时', '专业医师', '设备完善', '口碑良好'],
					address: '广东省广州市海珠区新港东路'
				},
				{
					name: '宠物食品店',
					distance: '4.7KM',
					rating: 4,
					image: 'https://images.unsplash.com/photo-1514933651103-005eec06c04b?auto=format&fit=crop&w=400&q=80',
					tags: ['进口食品', '营养丰富', '品牌多样', '正品保证'],
					address: '广东省广州市天河区珠江新城'
				},
			{
				name: '宠物玩具店',
				distance: '6.2KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1559339352-11d035aa65de?auto=format&fit=crop&w=400&q=80',
				tags: ['玩具丰富', '安全环保', '价格合理', '品种多样'],
				address: '广东省广州市荔湾区上下九步行街'
			},
			{
				name: '宠物寄养店',
				distance: '1.5KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1414235077428-338989a2e8c0?auto=format&fit=crop&w=400&q=80',
				tags: ['环境舒适', '专业看护', '价格透明', '服务热情'],
				address: '广东省广州市天河区体育西路'
			},
			{
				name: '宠物训练中心',
				distance: '2.8KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?auto=format&fit=crop&w=400&q=80',
				tags: ['专业训练', '经验丰富', '效果显著', '环境宽敞'],
				address: '广东省广州市越秀区建设大马路'
			},
			{
				name: '宠物用品超市',
				distance: '3.9KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1556910103-1c02745aae4d?auto=format&fit=crop&w=400&q=80',
				tags: ['一站式购物', '商品丰富', '价格优惠', '停车方便'],
				address: '广东省广州市海珠区新港中路'
			},
			{
				name: '宠物洗护店',
				distance: '5.4KM',
				rating: 4,
				image: 'https://images.unsplash.com/photo-1514933651103-005eec06c04b?auto=format&fit=crop&w=400&q=80',
				tags: ['专业洗护', '天然产品', '服务细致', '环境干净'],
				address: '广东省广州市天河区天河南一路'
			},
			{
				name: '宠物用品连锁店',
				distance: '4.3KM',
				rating: 5,
				image: 'https://images.unsplash.com/photo-1579952363873-27f3bade9f55?auto=format&fit=crop&w=400&q=80',
				tags: ['连锁品牌', '品质保证', '服务专业', '会员优惠'],
				address: '广东省广州市越秀区环市东路'
			}
		]
		}
	},
	computed: {
		// 计算卡片样式
		cardStyle() {
			return {
				bottom: this.cardBottom + 'px',
				height: this.cardHeight
			}
		}
	},
	onLoad() {
		// 数据已在 data 中初始化，无需额外加载
	},
	methods: {
		onLogoError(e) {
			console.error('LOGO加载失败:', e)
		},
		switchLocation() {
			uni.showToast({ title: '切换城市，待接入', icon: 'none' })
		},
		handleSearch() {
			if (!this.searchKeyword.trim()) {
				uni.showToast({ title: '请输入搜索关键词', icon: 'none' })
				return
			}
			// 跳转到搜索页面
			uni.navigateTo({
				url: '/pages/search/search?keyword=' + encodeURIComponent(this.searchKeyword.trim())
			})
		},
		switchTab(tab) {
			this.currentTab = tab
		},
		onParksScroll(e) {
			// 遛宠地滚动事件，保存滚动位置
			const scrollTop = e.detail.scrollTop
			this.parksScrollTop = scrollTop
			// 根据滚动位置更新卡片样式
			this.updateCardStyle(scrollTop)
		},
		onRestaurantsScroll(e) {
			// 宠物店滚动事件，保存滚动位置
			const scrollTop = e.detail.scrollTop
			this.restaurantsScrollTop = scrollTop
			// 根据滚动位置更新卡片样式
			this.updateCardStyle(scrollTop)
		},
		// 根据滚动位置更新卡片样式
		updateCardStyle(scrollTop) {
			// 滚动范围：0-300px
			const minScroll = 0
			const maxScroll = 300
			
			// 计算进度 0-1
			let progress = 0
			if (scrollTop <= minScroll) {
				progress = 0
			} else if (scrollTop >= maxScroll) {
				progress = 1
			} else {
				progress = (scrollTop - minScroll) / (maxScroll - minScroll)
			}
			
			// 初始值：bottom: 0px, height: 55vh
			// 目标值：bottom: 0px, height: 85vh
			// 根据进度平滑过渡
			const initialHeight = 55 // 初始高度 55vh
			const targetHeight = 85 // 目标高度 85vh
			
			// 使用缓动函数让过渡更平滑
			const easeProgress = progress * progress * (3 - 2 * progress)
			
			// bottom 保持为 0px（根据图片显示）
			this.cardBottom = 0
			
			// height 从 55vh 平滑过渡到 85vh
			const currentHeight = initialHeight + (targetHeight - initialHeight) * easeProgress
			this.cardHeight = currentHeight + 'vh'
		},
		// 模拟地图相关方法
		onMarkerTap(marker) {
			uni.showToast({ title: marker.name, icon: 'none' })
		},
		viewDetail(item) {
			// 缓存地点信息
			try {
				uni.setStorageSync('location-detail-temp', {
					...item,
					type: this.currentTab === 'parks' ? 'park' : 'shop'
				})
			} catch (e) {
				console.error('缓存地点信息失败', e)
			}

			// 根据类型跳转到不同的详情页
			if (this.currentTab === 'parks') {
				uni.navigateTo({
					url: '/pages/location/park-detail'
				})
			} else {
				uni.navigateTo({
					url: '/pages/location/shop-detail'
				})
			}
		},
		// 与底部导航交互保持一致
		handleTabChange(index) {
			this.switchTabBar(index)
		},
		switchTabBar(index) {
			if (index === 0) {
				// 跳转到首页
				uni.reLaunch({
					url: '/pages/index/index'
				})
			} else if (index === 1) {
				// 当前位置页面，不处理
				this.activeTab = index
			} else if (index === 3) {
				// 第四个按钮：切换到消息页面
				uni.navigateTo({
					url: '/pages/message/message'
				})
			} else if (index === 4) {
				// “我的”页面
				uni.navigateTo({
					url: '/pages/profile/profile'
				})
			} else {
				this.activeTab = index
			}
		},
		handlePublish(isActive) {
			// 发布按钮点击事件
			if (isActive) {
				uni.showToast({ title: '发布入口，待接入', icon: 'none' })
			}
		}
	}
}
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');

.page {
	min-height: 100vh;
	background: #f5f5f5;
	box-sizing: border-box;
	position: relative;
}

/* 顶部导航栏 */
.header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 0 34rpx 14rpx;
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	z-index: 999;
	background: #f5f5f5;
	box-sizing: border-box;
	padding-top: calc(var(--status-bar-height) + 25rpx);
	gap: 20rpx;
}

.header-left {
	display: flex;
	align-items: center;
	flex-shrink: 0;
}

.location {
	display: flex;
	align-items: center;
	gap: 8rpx;
	color: #999;
	line-height: 1;
}

.location-icon {
	font-size: 26rpx;
	color: #bbb;
	display: inline-flex;
	align-items: center;
	line-height: 1;
	transform: translateY(2rpx);
}

.location-text {
	font-size: 28rpx;
	color: #999;
	font-weight: 400;
	line-height: 1;
	display: inline-flex;
	align-items: center;
}

.header-center {
	flex: 1;
	display: flex;
	align-items: center;
}

.search-wrapper {
	flex: 1;
	display: flex;
	align-items: center;
	height: 58rpx;
	gap: 0;
	position: relative;
}

.search-input {
	flex: 1;
	height: 100%;
	padding: 0 20rpx;
	padding-right: 140rpx;
	font-size: 28rpx;
	color: #333;
	background: #e5e5e5;
	border: none;
	outline: none;
	line-height: 58rpx;
	box-sizing: border-box;
	border-radius: 50rpx;
	width: 100%;
}

.search-btn {
	height: 100%;
	padding: 0 32rpx;
	background: #383838;
	border-radius: 50rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
	position: absolute;
	right: 0;
	z-index: 1;
}

.search-btn-text {
	font-size: 28rpx;
	color: #fff;
	font-weight: 400;
	line-height: 1;
	display: inline-flex;
	align-items: center;
}

/* 地图区域 */
.map-container {
	position: fixed;
	top: calc(var(--status-bar-height) + 100rpx);
	left: 0;
	right: 0;
	height: 45vh;
	min-height: 500rpx;
	background: #e5e5e5;
	z-index: 1;
	overflow: hidden;
}

/* 模拟地图 */
.mock-map {
	width: 100%;
	height: 100%;
	position: relative;
	background: #f5f5f3;
	overflow: hidden;
}

/* 地图背景 */
.map-background {
	position: absolute;
	width: 100%;
	height: 100%;
	background: #f5f5f3;
	z-index: 1;
}

/* 河流 */
.river {
	position: absolute;
	background: #a8d5e2;
	border-radius: 20rpx;
	z-index: 2;
}

.river-1 {
	right: 0;
	top: 10%;
	width: 8%;
	height: 80%;
	border-radius: 40rpx 0 0 40rpx;
}

.river-2 {
	left: 5%;
	top: 20%;
	width: 15%;
	height: 25%;
	border-radius: 20rpx;
}

/* 公园区域 */
.park-area {
	position: absolute;
	background: #c8e6c9;
	border-radius: 8rpx;
	z-index: 3;
}

.park-1 {
	left: 2%;
	top: 15%;
	width: 20%;
	height: 30%;
	border-radius: 12rpx;
	display: flex;
	align-items: flex-start;
	justify-content: center;
	padding-top: 8rpx;
}

.park-2 {
	left: 30%;
	top: 60%;
	width: 12%;
	height: 15%;
	border-radius: 8rpx;
}

.park-3 {
	right: 15%;
	top: 75%;
	width: 10%;
	height: 12%;
	border-radius: 8rpx;
}

.park-label {
	font-size: 18rpx;
	color: #2e7d32;
	font-weight: 500;
	white-space: nowrap;
}

/* 道路 */
.road {
	position: absolute;
	background: #e0e0e0;
	z-index: 4;
}

.road-horizontal {
	height: 6rpx;
	width: 100%;
}

.road-vertical {
	width: 6rpx;
	height: 100%;
}

.road-1 {
	top: 20%;
	left: 0;
}

.road-2 {
	top: 35%;
	left: 0;
}

.road-3 {
	top: 50%;
	left: 0;
}

.road-4 {
	top: 70%;
	left: 0;
}

.road-5 {
	left: 15%;
	top: 0;
}

.road-6 {
	left: 30%;
	top: 0;
}

.road-7 {
	left: 50%;
	top: 0;
}

.road-8 {
	left: 75%;
	top: 0;
}

/* 道路标签 */
.road-label {
	position: absolute;
	font-size: 18rpx;
	color: #666;
	background: rgba(255, 255, 255, 0.8);
	padding: 2rpx 6rpx;
	border-radius: 4rpx;
	z-index: 5;
	white-space: nowrap;
	transform: translate(-50%, -50%);
}

/* 建筑物区块 */
.building {
	position: absolute;
	background: #d0d0d0;
	border-radius: 4rpx;
	z-index: 3;
}

.building-1 {
	left: 20%;
	top: 25%;
	width: 8%;
	height: 8%;
}

.building-2 {
	left: 35%;
	top: 25%;
	width: 10%;
	height: 6%;
}

.building-3 {
	left: 55%;
	top: 30%;
	width: 12%;
	height: 10%;
}

.building-4 {
	left: 20%;
	top: 55%;
	width: 10%;
	height: 8%;
}

.building-5 {
	left: 35%;
	top: 55%;
	width: 8%;
	height: 10%;
}

.building-6 {
	left: 55%;
	top: 60%;
	width: 10%;
	height: 8%;
}

.building-7 {
	left: 20%;
	top: 75%;
	width: 12%;
	height: 6%;
}

.building-8 {
	left: 40%;
	top: 80%;
	width: 8%;
	height: 8%;
}

/* 定位图标 */
.location-marker {
	position: absolute;
	left: 55%;
	top: 50%;
	transform: translate(-50%, -50%);
	z-index: 20;
	display: flex;
	align-items: center;
	justify-content: center;
}

.location-pulse {
	position: absolute;
	left: 55%;
	top: 50%;
	transform: translate(-50%, -50%);
	width: 120rpx;
	height: 120rpx;
	border-radius: 50%;
	background: rgba(231, 255, 96, 0.4);
	animation: locationPulse 2s infinite;
}

.map-location-icon {
	width: 90rpx;
	height: 90rpx;
	position: relative;
	z-index: 2;
}

@keyframes locationPulse {
	0% {
		transform: translate(-50%, -50%) scale(1);
		opacity: 0.6;
	}
	100% {
		transform: translate(-50%, -50%) scale(2);
		opacity: 0;
	}
}

/* POI点 */
.poi {
	position: absolute;
	z-index: 10;
	transform: translate(-50%, -50%);
	display: flex;
	flex-direction: column;
	align-items: center;
}

.poi-icon {
	font-size: 32rpx;
	background: #fff;
	border-radius: 50%;
	width: 48rpx;
	height: 48rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.15);
}

.poi-label {
	font-size: 18rpx;
	color: #333;
	background: rgba(255, 255, 255, 0.95);
	padding: 4rpx 8rpx;
	border-radius: 4rpx;
	margin-top: 4rpx;
	white-space: nowrap;
	box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
}

.poi-sub-label {
	font-size: 16rpx;
	color: #666;
	background: rgba(255, 255, 255, 0.95);
	padding: 2rpx 6rpx;
	border-radius: 4rpx;
	margin-top: 2rpx;
	white-space: nowrap;
	box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
}

/* 地图标记点 */
.map-marker {
	position: absolute;
	transform: translate(-50%, -50%);
	z-index: 10;
}

.marker-dot {
	width: 24rpx;
	height: 24rpx;
	border-radius: 50%;
	background: #fff;
	border: 3rpx solid;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.2);
	position: relative;
}

.marker-dot.current {
	border-color: #E7FF60;
	background: #E7FF60;
	width: 20rpx;
	height: 20rpx;
}

.marker-dot.park {
	border-color: #4CAF50;
	background: #4CAF50;
}

.marker-dot.restaurant {
	border-color: #FF9800;
	background: #FF9800;
}

.marker-label {
	position: absolute;
	top: 30rpx;
	left: 50%;
	transform: translateX(-50%);
	background: rgba(0, 0, 0, 0.7);
	color: #fff;
	padding: 4rpx 12rpx;
	border-radius: 8rpx;
	font-size: 20rpx;
	white-space: nowrap;
}

/* 地图中心位置指示 */
.map-center {
	position: absolute;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);
	z-index: 5;
}

.center-pulse {
	position: absolute;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);
	width: 40rpx;
	height: 40rpx;
	border-radius: 50%;
	background: rgba(231, 255, 96, 0.3);
	animation: pulse 2s infinite;
}

.center-location-icon {
	position: absolute;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);
	width: 48rpx;
	height: 48rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	background: #E7FF60;
	border-radius: 50%;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.3);
	z-index: 6;
}

.center-location-icon .fa {
	font-size: 28rpx;
	color: #333;
	line-height: 1;
}

@keyframes pulse {
	0% {
		transform: translate(-50%, -50%) scale(1);
		opacity: 0.8;
	}
	100% {
		transform: translate(-50%, -50%) scale(2);
		opacity: 0;
	}
}

/* 内容卡片区域 */
.content-card {
	position: fixed;
	left: 0;
	right: 0;
	background: #fff;
	border-radius: 1.25rem 1.25rem 0 0;
	z-index: 100;
	display: flex;
	flex-direction: column;
	box-sizing: border-box;
	box-shadow: none;
	transition: height 0.2s ease-out;
}

/* 选项卡 */
.tabs {
	display: flex;
	padding: 0rem 0rem 1.25rem;
	gap: 0;
	background: #fff;
	border-radius: 1.25rem 1.25rem 0 0;
	align-items: center;
	position: relative;
}

/* 选项卡背景图片容器 */
.tabs::before {
	content: '';
	position: absolute;
	top: -1.8rem;
	left: 0rem;
	right: 0rem;
	height: 5rem;
	background-image: url('/static/images/liugou.png');
	background-size: cover;
	background-position: center;
	background-repeat: no-repeat;
	border-radius: 0rem;
	pointer-events: none;
	z-index: 0;
	box-sizing: border-box;
	transition: transform 0.3s ease;
}

/* 激活宠物店时，显示 liugou1.png 素材 */
.tabs.tabs-flipped::before {
	background-image: url('/static/images/liugou1.png');
	transform: none;
}

.tab {
	flex: 1;
	padding: 18rpx 24rpx;
	background: transparent;
	text-align: center;
	transition: all 0.3s ease;
	position: relative;
	box-sizing: border-box;
	z-index: 1;
	margin: 0;
	height: 64rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

/* 左侧选项卡：左侧圆角，右侧直角 */
.tab:first-child {
	border-radius: 32rpx 0 0 32rpx;
}

/* 右侧选项卡：右侧圆角，左侧直角 */
.tab:last-child {
	border-radius: 0 32rpx 32rpx 0;
}

/* 如果只有一个选项卡，则四个角都是圆角 */
.tab:only-child {
	border-radius: 32rpx;
}

/* 激活的选项卡 */
.tab.active {
	z-index: 2;
}

.tab-text {
	font-size: 32rpx;
	color: #666;
	font-weight: 400;
	line-height: 1.4;
	display: block;
	position: relative;
	z-index: 1;
}

/* 激活选项卡的文字：黑色加粗 */
.tab.active .tab-text {
	color: #1b1b1b;
	font-weight: 600;
}

/* 未激活选项卡的文字：深灰色 */
.tab:not(.active) .tab-text {
	color: #333;
	font-weight: 500;
}

/* 地点列表 */
.location-list {
	flex: 1;
	overflow-y: auto;
	padding: 0rem;
	padding-bottom: 0rem;
	-webkit-overflow-scrolling: touch;
}

.location-item {
	display: flex;
	align-items: center;
	padding: 10rpx 32rpx;
	margin-bottom: 16rpx;
	background: transparent;
	border-radius: 32rpx;
	gap: 24rpx;
	box-shadow: none;
	overflow: hidden;
}

.location-item:last-child {
	margin-bottom: 0;
}

/* 空白占位卡片 */
.location-item-spacer {
	height: 5.25rem;
	width: 100%;
	background: transparent;
	flex-shrink: 0;
}

/* 左侧缩略图 */
.item-image-wrapper {
	width: 200rpx;
	height: 200rpx;
	border-radius: 20rpx;
	overflow: hidden;
	flex-shrink: 0;
	background: #f0f0f0;
}

.item-image {
	width: 100%;
	height: 100%;
	display: block;
}

/* 右侧内容 */
.item-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 0.1rem;
	min-width: 0;
}

/* 距离 */
.item-distance {
	display: flex;
	align-items: center;
	gap: 6rpx;
}

.distance-icon {
	font-size: 0.8rem;
	color: #999;
	display: inline-flex;
	align-items: center;
	justify-content: center;
	line-height: 1;
	vertical-align: middle;
}

.distance-text {
	font-size: 22rpx;
	color: #666;
	line-height: 1.5;
	display: inline-flex;
	align-items: center;
}

/* 地点名称 */
.item-name {
	font-size: 34rpx;
	color: #1b1b1b;
	font-weight: 700;
	line-height: 1.4;
}

/* 推荐指数 */
.item-rating {
	display: flex;
	align-items: center;
	gap: 10rpx;
}

.rating-label {
	font-size: 22rpx;
	color: #666;
	line-height: 1;
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

/* 标签列表 */
.item-tags {
	width: 100%;
	white-space: nowrap;
	/* 隐藏滚动条 */
	::-webkit-scrollbar {
		display: none;
	}
	scrollbar-width: none; /* Firefox */
	-ms-overflow-style: none; /* IE 10+ */
}

.tags-container {
	display: inline-flex;
	gap: 10rpx;
	white-space: nowrap;
	padding-right: 20rpx;
}

.tag {
	font-size: 20rpx;
	color: #666;
	line-height: 1.4;
	padding: 6rpx 14rpx;
	background: #f5f5f5;
	border-radius: 20rpx;
	white-space: nowrap;
	flex-shrink: 0;
}

/* 地址 */
.item-address {
	display: flex;
	align-items: center;
	gap: 6rpx;
}

.address-icon {
	font-size: 18rpx;
	color: #999;
	display: inline-flex;
	align-items: center;
	justify-content: center;
	line-height: 1;
	flex-shrink: 0;
	vertical-align: middle;
}

.address-text {
	font-size: 20rpx;
	color: #999;
	line-height: 1.5;
	flex: 1;
	display: inline-flex;
	align-items: center;
}

/* 右侧箭头 */
.item-arrow {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
	background: #f5f5f5;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
	align-self: flex-start;
}

.arrow-icon {
	font-size: 22rpx;
	color: #999;
	display: inline-flex;
	align-items: center;
	line-height: 1;
}

/* 底部导航栏 */
</style>