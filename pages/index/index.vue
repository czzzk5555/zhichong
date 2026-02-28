<template>
	<view class="page">
		<!-- 顶部导航 -->
		<!-- 首页导航 -->
		<view class="header" v-if="currentPage === 'home'">
			<view class="header-left">
				<view class="brand">
					<image src="/static/images/LOGO.png" class="brand-logo" mode="widthFix" @error="onLogoError"></image>
				</view>
				<view class="location" @tap="switchLocation">
					<text class="fa fa-location-dot location-icon"></text>
					<text class="location-text">{{ location }}</text>
				</view>
			</view>
			
			<view class="header-right">
				<view class="tabs">
					<view
						v-for="(tab, index) in tabs"
						:key="tab.value"
						class="tab"
						:class="{ active: tab.value === currentTab }"
						@tap="changeTab(tab.value, index)"
					>
						<text>{{ tab.label }}</text>
					</view>
					<!-- 下划线指示器 -->
					<view 
						class="tab-indicator"
						:style="{ 
							left: underlineLeft + 'px', 
							width: underlineWidth + 'px' 
						}"
					></view>
				</view>

				<view class="search-btn" @tap="handleSearchClick">
					<text class="fa fa-magnifying-glass search-icon"></text>
				</view>
			</view>
		</view>
		
		<!-- 位置页导航 -->
		<view class="header header-location" v-if="currentPage === 'location'">
			<view class="location-header-left">
				<view class="location-icon-wrapper" @tap="switchLocation">
					<text class="fa fa-location-dot location-icon-new"></text>
					<text class="location-text-new">{{ location }}</text>
				</view>
			</view>
			
			<view class="location-search-wrapper">
				<input 
					class="location-search-input" 
					type="text" 
					placeholder="搜一搜其它遛宠地或友好餐厅"
					placeholder-style="color: #999;"
				/>
				<view class="location-search-btn" @tap="handleSearch">
					<text class="location-search-btn-text">搜索</text>
				</view>
			</view>
		</view>

		<!-- 内容 -->
		<view class="content-wrapper">
			<!-- 首页内容 -->
			<view v-if="currentPage === 'home'" class="home-content">
			<!-- 滑动容器 -->
			<view 
				class="swiper-container" 
				:class="{ 'swiper-transition': !isTouching }"
				:style="{ transform: `translateX(${swiperTransform}%)` }"
				@touchstart="onTouchStart"
				@touchmove="onTouchMove"
				@touchend="onTouchEnd"
			>
				<!-- 关注页：单列全宽布局 -->
				<scroll-view 
					class="scroll scroll-follow" 
					scroll-y 
					:show-scrollbar="false" 
					:scroll-top="followScrollTop" 
					@scroll="onFollowScroll"
					refresher-enabled="true"
					:refresher-triggered="followRefreshing"
					@refresherrefresh="onFollowRefresh"
					@refresherrestore="onFollowRestore"
					refresher-background="#f5f5f5"
					refresher-default-style="none"
				>
					<!-- 自定义刷新图标 -->
					<view class="custom-refresher-wrapper custom-refresher-follow" v-if="followRefreshing">
						<view class="custom-refresher-icon"></view>
					</view>
					<view class="feed-list">
				<view
					class="post-card"
					v-for="post in followPosts"
					:key="post.id"
					:class="{ 'card-visible': post.visible }"
					:data-id="'card-' + post.id"
				>
					<!-- 用户信息区域 -->
					<view class="user-info">
						<view class="user-left">
							<image :src="post.avatar" class="user-avatar" mode="aspectFill"></image>
							<view class="user-details">
								<view class="username-row">
									<text class="username">{{ post.author }}</text>
									<text class="time-text-inline">发布于{{ formatTime(post.publishTime) }}</text>
								</view>
								<view class="hashtags">
									<text 
										v-for="(tag, idx) in post.tags" 
										:key="idx" 
										class="hashtag"
									>#{{ tag }}</text>
								</view>
							</view>
						</view>
						<view class="message-btn" @tap="sendMessage(post)">
							<text class="fa fa-user message-icon"></text>
							<text class="message-text">发消息</text>
						</view>
					</view>

					<!-- 文本内容 -->
					<view class="post-content" v-if="post.content">
						<text class="content-text">{{ post.content }}</text>
					</view>

					<!-- 媒体内容 -->
					<view class="media-wrap" :class="{ 'media-wrap-single': !post.isVideo && (!post.imageCount || post.imageCount <= 1) && !post.images }">
						<!-- 模糊背景层 - 用于填充图片两侧的黑色区域 -->
						<view 
							class="blur-background"
							:style="getBlurBackgroundStyle(post)"
						></view>
						<!-- 多图轮播 -->
						<view 
							v-if="!post.isVideo && post.imageCount > 1 && post.images" 
							class="image-carousel"
							:data-post-id="post.id"
							:style="post.firstImageHeight ? { height: post.firstImageHeight + 'px' } : ''"
							@touchstart="onImageTouchStart($event, post)"
							@touchmove="onImageTouchMove($event, post)"
							@touchend="onImageTouchEnd($event, post)"
						>
							<view 
								class="image-carousel-container"
								:style="{ transform: `translateX(-${post.currentIndex * 100}%)` }"
							>
								<image 
									v-for="(img, idx) in post.images" 
									:key="idx"
									:src="img" 
									class="media-image carousel-image" 
									:class="{ 'first-image': idx === 0 }"
									:mode="idx === 0 ? 'widthFix' : 'aspectFit'"
									@load="onFirstImageLoad($event, post, idx)"
									@tap="previewImage(post, idx)"
								></image>
							</view>
							<!-- 轮播指示器 -->
							<view class="carousel-dots">
								<view 
									v-for="(dot, idx) in post.imageCount" 
									:key="idx"
									class="dot"
									:class="{ active: idx === post.currentIndex }"
								></view>
							</view>
						</view>
						<!-- 单图或视频 -->
						<template v-else>
							<image 
								v-if="!post.isVideo" 
								:src="post.image" 
								class="media-image" 
								mode="widthFix"
								@load="onSingleImageLoad"
								@tap="previewImage(post)"
							></image>
							<view v-else class="video-wrap" @tap="playVideo(post)">
								<image :src="post.image" class="video-poster" mode="widthFix" @load="onSingleImageLoad"></image>
								<view class="play-button">
									<text class="fa fa-play play-icon" style="display: inline-block;"></text>
								</view>
							</view>
						</template>
					</view>

					<!-- 互动栏 -->
					<view class="interaction-bar">
						<view class="interaction-item" @tap="sharePost(post)">
							<text class="fa fa-share-from-square fa-regular interaction-icon share-icon"></text>
							<text class="interaction-count">{{ formatNumber(post.shares) }}</text>
						</view>
						<view class="interaction-item" @tap.stop="toggleLike(post)">
							<text 
								class="fa interaction-icon like-icon" 
								:class="[
									post.isLiked ? 'fa-heart fa-solid' : 'fa-heart fa-regular'
								]"
								:style="{ color: post.isLiked ? '#f25e5e' : '#1b1b1b' }"
							></text>
							<text class="interaction-count">{{ formatNumber(post.likes) }}</text>
						</view>
						<view class="interaction-item" @tap="commentPost(post)">
							<text class="fa fa-comment fa-regular interaction-icon"></text>
							<text class="interaction-count">{{ formatNumber(post.comments) }}</text>
						</view>
						<view class="interaction-item" @tap="favoritePost(post)">
							<text 
								class="fa interaction-icon favorite-icon" 
								:class="[
									post.isFavorited ? 'fa-star fa-solid' : 'fa-star fa-regular'
								]"
								:style="{ color: post.isFavorited ? '#ffd700' : '#1b1b1b' }"
							></text>
							<text class="interaction-count">{{ formatNumber(post.favorites) }}</text>
						</view>
					</view>
				</view>
					</view>
				</scroll-view>

				<!-- 推荐页：瀑布流布局 -->
				<scroll-view 
					class="scroll scroll-recommend" 
					scroll-y 
					:show-scrollbar="false" 
					:scroll-top="recommendScrollTop" 
					@scroll="onRecommendScroll"
					refresher-enabled="true"
					:refresher-triggered="recommendRefreshing"
					@refresherrefresh="onRecommendRefresh"
					@refresherrestore="onRecommendRestore"
					refresher-background="#f5f5f5"
					refresher-default-style="none"
				>
					<!-- 自定义刷新图标 -->
					<view class="custom-refresher-wrapper" v-if="recommendRefreshing">
						<view class="custom-refresher-icon"></view>
					</view>
					<view class="masonry">
				<!-- 发布卡片 -->
				<view class="card publish-card" @tap="goPublish">
					<!-- 文字内容区域 -->
					<view class="publish-text-wrapper">
						<text class="publish-text-line">快来发布</text>
						<text class="publish-text-line publish-text-line-2">你宝贝的</text>
						<text class="publish-text-line publish-text-line-3">可爱日常吧!</text>
					</view>
					<!-- 柯基犬插画 -->
					<image src="/static/images/gougou.png" class="publish-dog" mode="aspectFit"></image>
					<!-- 相机图标 -->
					<image src="/static/images/xiangji.png" class="publish-camera" mode="aspectFit"></image>
					<!-- GO按钮 -->
					<view class="publish-btn">GO!</view>
				</view>

				<!-- 动态卡片 -->
				<view
					class="card"
					v-for="post in posts"
					:key="post.id"
					:class="[post.size, { 'card-visible': post.visible }]"
					:data-id="'card-' + post.id"
					@tap="handleRecommendCardTap(post)"
				>
					<view class="image-wrap">
						<image :src="post.image" class="image" mode="aspectFill"></image>
						<view v-if="post.isVideo" class="play-badge">
							<text class="fa fa-play"></text>
						</view>
					</view>
					<view class="title" :title="post.title">{{ post.title.length > 10 ? post.title.substring(0, 10) + '...' : post.title }}</view>
					<view class="meta">
						<view class="user">
							<image :src="post.avatar" class="avatar" mode="aspectFill"></image>
							<text class="username">{{ post.author.length > 6 ? post.author.substring(0, 6) + '...' : post.author }}</text>
						</view>
						<view class="likes" @tap.stop="toggleLike(post)">
							<text 
								class="fa fa-heart like-icon-recommend" 
								:class="[
									post.isLiked ? 'fa-solid' : 'fa-regular',
									{ 'fade-in-like': post.isAnimatingLike }
								]"
								:style="{ color: post.isLiked ? '#f25e5e' : '#1b1b1b' }"
							></text>
							<text class="like-num">{{ formatLikes(post.likes) }}</text>
						</view>
					</view>
				</view>
					</view>
				</scroll-view>
			</view>
			</view>
			
			<!-- 位置页内容 -->
			<view v-if="currentPage === 'location'" class="location-content">
				<scroll-view 
					class="scroll scroll-location" 
					scroll-y 
					:show-scrollbar="false"
				>
					<view class="location-page-content">
						<text class="location-content-text">位置页面内容</text>
					</view>
				</scroll-view>
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
			location: '广州',
			activeTab: 0,
			currentPage: 'home', // 当前页面：'home' 首页, 'location' 位置页
			currentTab: 'recommend',
			currentTabIndex: 1, // 默认推荐 (index 1)
			indicatorLeft: 0, // 初始位置，将在 mounted 中计算
			underlineLeft: 0, // 下划线左边距
			underlineWidth: 0, // 下划线宽度
			followScrollTop: 0, // 关注页滚动位置
			recommendScrollTop: 0, // 推荐页滚动位置
			followRefreshing: false, // 关注页刷新状态
			recommendRefreshing: false, // 推荐页刷新状态
			// 滑动相关
			touchStartX: 0, // 触摸开始X坐标
			touchStartY: 0, // 触摸开始Y坐标
			isTouching: false, // 是否正在触摸
			swiperTransform: -50, // 滑动容器的transform值（-50表示推荐页，0表示关注页）
			swiperOffset: 0, // 滑动偏移量
			observers: [], // Intersection Observer 观察器数组
			// 关注页数据（单列全宽布局）
			followPosts: [
				{
					id: 1,
					author: 'ECHO',
					avatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					tags: ['萨摩耶', '柴犬'],
					content: '宝宝穿新衣服,嘴角就福娃娃房娃和人合同打底袜翻大发我福娃福娃建议天津条约',
					image: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					images: [
						'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1504595403659-9088ce801e29?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1501004318641-b39e6451bec6?auto=format&fit=crop&w=1000&q=80'
					],
					isVideo: false,
					imageCount: 5,
					currentIndex: 0,
					shares: 358,
					likes: 3580,
					comments: 125,
					favorites: 876,
					isLiked: false,
					isFavorited: false,
					isAnimating: false,
					isAnimatingLike: false,
					isAnimatingFavorite: false,
					isImageSliding: false,
					publishTime: Date.now() - 23 * 60 * 1000, // 23分钟前
					visible: false
				},
				{
					id: 2,
					author: '童一敏',
					avatar: 'https://images.unsplash.com/photo-1525253086316-d0c936c814f8?auto=format&fit=facearea&w=96&h=96&q=80',
					tags: ['阿拉斯加', '柴犬'],
					content: '今天天气很Nice,每天都适合刷街啊',
					image: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					isVideo: true,
					imageCount: 1,
					currentIndex: 0,
					shares: 210,
					likes: 1435,
					comments: 89,
					favorites: 456,
					isLiked: true,
					isFavorited: false,
					isAnimating: false,
					isAnimatingLike: false,
					isAnimatingFavorite: false,
					isImageSliding: false,
					publishTime: Date.now() - 24 * 60 * 60 * 1000, // 1天前
					visible: false
				}
			],
			// 推荐页数据（瀑布流布局）
			posts: [
				{
					id: 1,
					title: '调皮的汪星人',
					author: 'ECHO',
					likes: 3580,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1507146426996-ef05306b995a?auto=format&fit=crop&w=900&q=80',
					isVideo: true,
					size: 'tall',
					visible: false
				},
				{
					id: 2,
					title: '宝宝穿新衣服，嘴角都上扬~',
					author: '童一敏',
					likes: 1435,
					isLiked: true,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1525253086316-d0c936c814f8?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
					// 为该笔记配置多图，用于详情页轮播展示
					images: [
						'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1518717758536-85ae29035b6d?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1517423440428-a5a00ad493e8?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1534361960057-19889db9621e?auto=format&fit=crop&w=900&q=80'
					],
					isVideo: false,
					size: 'tall',
					visible: false
				},
				{
					id: 3,
					title: '调皮的汪星人',
					author: '小小肉酱',
					likes: 8547,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1505628346881-b72b27e84530?auto=format&fit=crop&w=900&q=80',
					isVideo: true,
					size: 'medium',
					visible: false
				},
				{
					id: 4,
					title: '调皮的汪星人',
					author: '糯米',
					likes: 9120,
					isLiked: true,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1504595403659-9088ce801e29?auto=format&fit=crop&w=900&q=80',
					isVideo: false,
					size: 'medium',
					visible: false
				},
				{
					id: 5,
					title: '调皮的汪星人',
					author: '阳光',
					likes: 4021,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1521572163474-6864f9cf17ab?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1501004318641-b39e6451bec6?auto=format&fit=crop&w=1000&q=80',
					isVideo: true,
					size: 'wide',
					visible: false
				},
				{
					id: 6,
					title: '周末去哪玩？带狗子去公园撒欢',
					author: '阿黄的铲屎官',
					likes: 2100,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1543466835-00a7907e9de1?auto=format&fit=crop&w=900&q=80',
					isVideo: false,
					size: 'tall',
					visible: false
				},
				{
					id: 7,
					title: '这只猫咪的眼神简直太治愈了！',
					author: '喵星人日记',
					likes: 5670,
					isLiked: true,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1527980965255-d3b416303d12?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?auto=format&fit=crop&w=900&q=80',
					isVideo: true,
					size: 'medium',
					visible: false
				},
				{
					id: 8,
					title: '第一次洗澡，委屈巴巴',
					author: '小短腿柯基',
					likes: 3340,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1580489944761-15a19d654956?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1583337130417-3346a1be7dee?auto=format&fit=crop&w=900&q=80',
					isVideo: false,
					size: 'tall',
					visible: false
				},
				{
					id: 9,
					title: '今日份可爱已送达，请签收',
					author: '大橘为重',
					likes: 890,
					isLiked: false,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1519052537078-e6302a4968d4?auto=format&fit=crop&w=900&q=80',
					isVideo: true,
					size: 'medium',
					visible: false
				},
				{
					id: 10,
					title: '睡觉的样子太搞笑了哈哈哈',
					author: '哈士奇拆家大队',
					likes: 12500,
					isLiked: true,
					isAnimating: false,
					isAnimatingLike: false,
					avatar: 'https://images.unsplash.com/photo-1500648767791-00dcc994a43e?auto=format&fit=facearea&w=96&h=96&q=80',
					image: 'https://images.unsplash.com/photo-1541781777629-1229a916cc75?auto=format&fit=crop&w=900&q=80',
					isVideo: false,
					size: 'wide',
					visible: false
				}
			]
		}
	},
	computed: {
		tabs() {
			// 根据当前页面返回不同的标签
			if (this.currentPage === 'location') {
				return [
					{ label: '友好餐厅', value: 'follow' },
					{ label: '遛宠地', value: 'recommend' }
				]
			} else {
				return [
					{ label: '关注', value: 'follow' },
					{ label: '推荐', value: 'recommend' }
				]
			}
		}
	},
	watch: {
		currentTab(newVal) {
			// 同步滑动容器的位置
			this.swiperTransform = newVal === 'follow' ? 0 : -50
			
			// 切换标签时重置滚动位置到顶部
			// 使用技巧：先设置为非0值，再设置为0，确保scroll-view更新
			if (newVal === 'follow') {
				this.followScrollTop = 0
				this.$nextTick(() => {
					this.followScrollTop = 1
					this.$nextTick(() => {
						this.followScrollTop = 0
					})
				})
			} else {
				this.recommendScrollTop = 0
				this.$nextTick(() => {
					this.recommendScrollTop = 1
					this.$nextTick(() => {
						this.recommendScrollTop = 0
					})
				})
			}
			
			// 切换标签时重置卡片可见性并重新初始化观察器
			this.$nextTick(() => {
				// 先销毁旧的观察器
				if (this.observers && this.observers.length > 0) {
					this.observers.forEach(observer => {
						if (observer && observer.disconnect) {
							observer.disconnect()
						}
					})
					this.observers = []
				}
				
				// 重置当前标签下所有卡片的可见性
				const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
				posts.forEach(post => {
					post.visible = false
				})
				
				// 重新初始化观察器，延迟确保DOM已更新
				setTimeout(() => {
					this.initCardObserver()
				}, 100)
			})
		},
		currentPage() {
			// 页面切换时更新指示器位置
			this.$nextTick(() => {
				this.updateIndicator()
			})
		}
	},
	onReady() {
		// 初始计算指示器位置 - onReady 确保 DOM 已完全渲染
		this.$nextTick(() => {
			this.updateIndicator()
			// 如果第一次计算失败，延迟再试一次（处理字体加载等情况）
			setTimeout(() => {
				this.updateIndicator()
			}, 50)
		})
		
		// 初始化卡片可见性监听
		this.$nextTick(() => {
			this.initCardObserver()
		})
	},
	beforeDestroy() {
		// 组件销毁前清理观察器
		if (this.observers && this.observers.length > 0) {
			this.observers.forEach(observer => {
				if (observer && observer.disconnect) {
					observer.disconnect()
				}
			})
			this.observers = []
		}
	},
	onUnload() {
		// 页面卸载时清理观察器
		if (this.observers && this.observers.length > 0) {
			this.observers.forEach(observer => {
				if (observer && observer.disconnect) {
					observer.disconnect()
				}
			})
			this.observers = []
		}
	},
	methods: {
		onLogoError(e) {
			console.error('LOGO加载失败:', e)
			uni.showToast({
				title: 'LOGO加载失败',
				icon: 'none'
			})
		},
		changeTab(val, index) {
			this.currentTab = val
			this.currentTabIndex = index
			this.updateIndicator()
		},
		updateIndicator() {
			// 获取 tabs 容器和当前选中 tab 的位置信息
			const query = uni.createSelectorQuery().in(this)
			query.select('.tabs').boundingClientRect()
			query.selectAll('.tab').boundingClientRect()
			query.exec(res => {
				if (res[0] && res[1] && res[1][this.currentTabIndex]) {
					const tabsRect = res[0]
					const activeTabRect = res[1][this.currentTabIndex]
					
					// 计算指示器中心对齐 tab 中心
					// 使用 transform: translateX(-50%) 居中，所以 left 直接设置为标签中心位置
					// 向右偏移一点，关注标签额外偏移
					const baseOffset = 15
					const followOffset = this.currentTab === 'follow' ? 3 : 0
					this.indicatorLeft = (activeTabRect.left - tabsRect.left) + (activeTabRect.width / 2) + baseOffset + followOffset
					
					// 计算下划线位置和宽度（占按钮宽度的70%）
					const tabWidth = activeTabRect.width
					const underlineWidthPercent = 0.7 // 70%宽度
					this.underlineWidth = tabWidth * underlineWidthPercent
					// 下划线居中于按钮
					this.underlineLeft = (activeTabRect.left - tabsRect.left) + (tabWidth * (1 - underlineWidthPercent) / 2)
				}
			})
		},
		goPublish() {
			// 切换激活状态
			this.isPlusActive = !this.isPlusActive
			
			if (this.isPlusActive) {
				// 激活状态下的逻辑（例如弹出发布菜单）
				// 目前仅演示 UI 变化
			} else {
				// 取消激活状态下的逻辑
			}
		},
		switchLocation() {
			uni.showToast({ title: '切换城市，待接入', icon: 'none' })
		},
		handleSearchClick() {
			// 跳转到搜索页面
			uni.navigateTo({
				url: '/pages/search/search'
			})
		},
		handleSearch() {
			// 位置页的搜索功能
			uni.navigateTo({
				url: '/pages/search/search'
			})
		},
		handleLocationClick() {
			// 跳转到位置页面
			uni.navigateTo({
				url: '/pages/location/location'
			})
		},
		handleTabChange(index) {
			if (index === 0) {
				// 第一个按钮：返回首页
				this.currentPage = 'home'
				this.activeTab = 0
				// 重置标签为默认值并更新指示器
				this.currentTab = 'recommend'
				this.currentTabIndex = 1
				this.$nextTick(() => {
					this.updateIndicator()
				})
			} else if (index === 1) {
				// 第二个按钮：切换到位置页面
				this.handleLocationClick()
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
				if (this.currentPage === 'location') {
					// 如果在位置页，切换其他标签时返回首页
					this.currentPage = 'home'
					// 重置标签为默认值并更新指示器
					this.currentTab = 'recommend'
					this.currentTabIndex = 1
					this.$nextTick(() => {
						this.updateIndicator()
					})
				}
				if (index !== 0 && index !== 1 && index !== 3) {
					uni.showToast({ title: '功能待开发', icon: 'none' })
				}
			}
		},
		handlePublish(isActive) {
			// 发布按钮点击事件
			if (isActive) {
				uni.showToast({ title: '发布入口，待接入', icon: 'none' })
			}
		},
		toggleLike(post, e) {
			// 阻止事件冒泡和默认行为
			if (e) {
				e.stopPropagation && e.stopPropagation()
				e.preventDefault && e.preventDefault()
			}
			
			const wasLiked = post.isLiked
			post.isLiked = !post.isLiked
			
			// 判断是否为推荐页的帖子（在 posts 数组中）
			const isRecommendPost = this.posts.some(p => p.id === post.id)
			
			// 如果从未点赞变为点赞，且是推荐页的帖子，触发透明度渐入动画
			if (!wasLiked && post.isLiked && isRecommendPost) {
				post.isAnimatingLike = true
				setTimeout(() => {
					post.isAnimatingLike = false
				}, 600)
			}
			
			if (post.isLiked) {
				post.likes++
			} else {
				post.likes--
			}
		},
		formatLikes(likes) {
			if (likes >= 10000) {
				return (likes / 10000).toFixed(1) + 'W'
			}
			return likes.toString()
		},
		// 关注页方法
		sendMessage(post) {
			// 检查缓存中是否已有该用户的聊天记录
			try {
				const cached = uni.getStorageSync('chat-user-temp')
				let messages = []
				
				// 如果缓存中有同一个人且消息数量更多，使用缓存中的消息
				if (cached && cached.name === post.author && cached.messages && cached.messages.length > 0) {
					messages = cached.messages
				}
				
				// 缓存聊天用户信息和消息记录，跳转到聊天详情页
				uni.setStorageSync('chat-user-temp', {
					name: post.author,
					avatar: post.avatar,
					messages: messages
				})
				
				uni.navigateTo({
					url: '/pages/chat/chat-detail'
				})
			} catch (e) {
				console.error('缓存聊天用户信息失败', e)
				uni.showToast({ title: '打开聊天失败', icon: 'none' })
			}
		},
		sharePost(post) {
			uni.showToast({ title: '分享功能待开发', icon: 'none' })
		},
		commentPost(post) {
			uni.showToast({ title: '评论功能待开发', icon: 'none' })
		},
		favoritePost(post) {
			const wasFavorited = post.isFavorited
			post.isFavorited = !post.isFavorited
			
			if (post.isFavorited) {
				post.favorites++
			} else {
				post.favorites--
			}
		},
		previewImage(post, index = 0) {
			const urls = post.images && post.images.length > 0 ? post.images : [post.image]
			uni.previewImage({
				urls: urls,
				current: index || 0
			})
		},
		// 第一张图片加载完成，获取高度
		onFirstImageLoad(e, post, index) {
			// 只处理第一张图片（index === 0）
			if (index === 0 && !post.firstImageHeight) {
				this.$nextTick(() => {
					const query = uni.createSelectorQuery().in(this)
					// 查找当前卡片的第一张图片
					query.selectAll(`.post-card[data-id="card-${post.id}"] .first-image`).boundingClientRect((res) => {
						if (res && res.length > 0 && res[0].height) {
							// 将高度存储到 post 对象中（使用 px 单位）
							this.$set(post, 'firstImageHeight', res[0].height)
						}
					}).exec()
				})
			}
		},
		// 单图加载完成处理
		onSingleImageLoad(e) {
			// 单图加载完成后，确保容器高度正确
			// 使用 widthFix 模式时，图片会自动计算高度，这里不需要额外处理
		},
		// 获取模糊背景样式
		getBlurBackgroundStyle(post) {
			let imageUrl = ''
			
			// 如果是多图轮播，使用当前显示的图片
			if (!post.isVideo && post.imageCount > 1 && post.images && post.images.length > 0) {
				const currentIndex = post.currentIndex || 0
				imageUrl = post.images[currentIndex] || post.images[0] || ''
			} else {
				// 单图或视频，使用post.image
				imageUrl = post.image || ''
			}
			
			if (!imageUrl) {
				return {
					display: 'none'
				}
			}
			
			return {
				backgroundImage: `url(${imageUrl})`,
				backgroundSize: 'cover',
				backgroundPosition: 'center',
				backgroundRepeat: 'no-repeat',
				display: 'block'
			}
		},
		// 图片滑动事件处理
		onImageTouchStart(e, post) {
			if (!post.imageTouchStartX) {
				post.imageTouchStartX = 0
				post.imageTouchStartY = 0
				post.imageTouchOffset = 0
			}
			post.imageTouchStartX = e.touches[0].clientX
			post.imageTouchStartY = e.touches[0].clientY
			post.imageTouching = true
			post.isImageSliding = false // 标记是否正在图片滑动
			// 不在这里阻止事件冒泡，让下拉刷新可以正常工作
		},
		onImageTouchMove(e, post) {
			if (!post.imageTouching) return
			
			const currentX = e.touches[0].clientX
			const currentY = e.touches[0].clientY
			const deltaX = currentX - post.imageTouchStartX
			const deltaY = Math.abs(currentY - post.imageTouchStartY)
			
			// 判断是否为横向滑动（横向滑动距离大于纵向滑动距离）
			if (Math.abs(deltaX) > deltaY && Math.abs(deltaX) > 10) {
				post.imageTouchOffset = deltaX
				post.isImageSliding = true // 标记正在图片滑动
				
				// 只有确认是横向滑动时才阻止事件冒泡，避免触发页面滑动
				if (e.stopPropagation) {
					e.stopPropagation()
				}
				if (e.preventDefault) {
					e.preventDefault()
				}
			} else if (deltaY > Math.abs(deltaX) && deltaY > 10) {
				// 如果是纵向滑动（下拉刷新），不阻止事件冒泡，允许下拉刷新
				post.isImageSliding = false
			}
		},
		onImageTouchEnd(e, post) {
			if (!post.imageTouching) return
			
			post.imageTouching = false
			const screenWidth = uni.getSystemInfoSync().windowWidth
			const threshold = screenWidth * 0.2 // 滑动阈值：屏幕宽度的20%
			const deltaX = post.imageTouchOffset
			let imageChanged = false
			
			// 只有确认是横向滑动时才处理图片切换
			if (post.isImageSliding) {
				// 根据滑动距离切换图片
				if (Math.abs(deltaX) > threshold) {
					if (deltaX > 0 && post.currentIndex > 0) {
						// 向右滑动，显示上一张
						this.$set(post, 'currentIndex', post.currentIndex - 1)
						imageChanged = true
					} else if (deltaX < 0 && post.currentIndex < post.imageCount - 1) {
						// 向左滑动，显示下一张
						this.$set(post, 'currentIndex', post.currentIndex + 1)
						imageChanged = true
					}
				}
				
				// 如果发生了图片切换，阻止事件冒泡
				if (imageChanged) {
					if (e.stopPropagation) {
						e.stopPropagation()
					}
					if (e.preventDefault) {
						e.preventDefault()
					}
				}
			}
			// 如果是纵向滑动（下拉刷新），不阻止事件冒泡，允许下拉刷新
			
			// 重置偏移量和标记
			post.imageTouchOffset = 0
			post.isImageSliding = false
		},
		playVideo(post) {
			uni.showToast({ title: '视频播放功能待开发', icon: 'none' })
		},
		formatNumber(num) {
			if (num > 9999) {
				return (num / 10000).toFixed(1) + 'W'
			}
			return num.toString()
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
			} else {
				return days + '天前'
			}
		},
		initCardObserver() {
			// 使用 Intersection Observer 监听卡片进入视口
			this.$nextTick(() => {
				// 确保观察器数组已初始化
				if (!this.observers) {
					this.observers = []
				}
				
				// 先检查初始可见的卡片
				this.checkInitialVisibleCards()
				
				// 根据当前标签获取对应的帖子列表和选择器
				const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
				const selector = this.currentTab === 'follow' ? '.post-card' : '.card'
				const scrollSelector = this.currentTab === 'follow' ? '.scroll-follow' : '.scroll-recommend'
				
				// 确保有帖子数据再创建观察器
				if (!posts || posts.length === 0) {
					return
				}
				
				posts.forEach((post, index) => {
					try {
						const observer = uni.createIntersectionObserver(this, {
							thresholds: [0, 0.1], // 当卡片进入视口时触发
							initialRatio: 0
						})
						
						observer.relativeTo(scrollSelector).observe(`${selector}[data-id="card-${post.id}"]`, (res) => {
							if (res.intersectionRatio > 0 && !post.visible) {
								// 延迟触发，形成错落效果
								setTimeout(() => {
									if (post && !post.visible) {
										post.visible = true
									}
								}, index * 30)
							}
						})
						
						this.observers.push(observer)
					} catch (error) {
						console.error('创建观察器失败:', error)
					}
				})
			})
		},
		checkInitialVisibleCards() {
			// 检查初始可见的卡片（第一屏的卡片）
			const scrollSelector = this.currentTab === 'follow' ? '.scroll-follow' : '.scroll-recommend'
			const query = uni.createSelectorQuery().in(this)
			query.select(scrollSelector).boundingClientRect()
			query.selectViewport().scrollOffset()
			query.exec((res) => {
				if (!res[0] || !res[1]) return
				
				const scrollTop = res[1].scrollTop
				const viewportHeight = res[0].height
				const viewportBottom = scrollTop + viewportHeight
				
				// 根据当前标签获取对应的帖子列表和选择器
				const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
				const selector = this.currentTab === 'follow' ? '.post-card' : '.card'
				
				// 检查前几张卡片（第一屏可见的）
				const cardQuery = uni.createSelectorQuery().in(this)
				// 检查前4张卡片（通常第一屏能看到2-4张）
				for (let i = 0; i < Math.min(4, posts.length); i++) {
					const post = posts[i]
					if (post) {
						cardQuery.select(`${selector}[data-id="card-${post.id}"]`).boundingClientRect()
					}
				}
				cardQuery.exec((cardRects) => {
					cardRects.forEach((rect, index) => {
						if (!rect) return
						const post = posts[index]
						if (!post || post.visible) return
						
						// 如果卡片在视口内，直接显示
						if (rect.top < viewportBottom && rect.bottom > scrollTop) {
							setTimeout(() => {
								post.visible = true
							}, index * 30)
						}
					})
				})
			})
		},
		onFollowScroll(e) {
			// 关注页滚动事件
			this.followScrollTop = e.detail.scrollTop
		},
		onRecommendScroll(e) {
			// 推荐页滚动事件
			this.recommendScrollTop = e.detail.scrollTop
		},
		onScroll(e) {
			// 滚动事件，可以用于节流优化（保留兼容性）
		},
		// 触摸事件处理
		onTouchStart(e) {
			// 检查是否在图片轮播区域内
			const touchX = e.touches[0].clientX
			const touchY = e.touches[0].clientY
			
			// 检查是否有图片正在滑动（检查当前标签对应的帖子列表）
			const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
			const hasImageSliding = posts.some(post => post.isImageSliding)
			if (hasImageSliding) {
				return // 如果有图片正在滑动，不处理页面滑动
			}
			
			this.touchStartX = touchX
			this.touchStartY = touchY
			this.isTouching = true
			this.swiperOffset = 0
		},
		onTouchMove(e) {
			if (!this.isTouching) return
			
			// 检查是否有图片正在滑动（检查当前标签对应的帖子列表）
			const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
			const hasImageSliding = posts.some(post => post.isImageSliding)
			if (hasImageSliding) {
				return // 如果有图片正在滑动，不处理页面滑动
			}
			
			const currentX = e.touches[0].clientX
			const currentY = e.touches[0].clientY
			const deltaX = currentX - this.touchStartX
			const deltaY = Math.abs(currentY - this.touchStartY)
			
			// 判断是否为横向滑动（横向滑动距离大于纵向滑动距离）
			if (Math.abs(deltaX) > deltaY && Math.abs(deltaX) > 10) {
				// 计算滑动百分比（屏幕宽度为100%）
				const screenWidth = uni.getSystemInfoSync().windowWidth
				const offsetPercent = (deltaX / screenWidth) * 100
				
				// 基于当前标签的基础位置计算新位置
				const baseTransform = this.currentTab === 'follow' ? 0 : -50
				let newTransform = baseTransform + offsetPercent
				
				// 限制滑动范围
				if (newTransform > 0) newTransform = 0 // 不能超过关注页
				if (newTransform < -50) newTransform = -50 // 不能超过推荐页
				
				this.swiperOffset = offsetPercent
				this.swiperTransform = newTransform
			}
		},
		onTouchEnd(e) {
			if (!this.isTouching) return
			
			// 检查是否有图片正在滑动（检查当前标签对应的帖子列表）
			const posts = this.currentTab === 'follow' ? this.followPosts : this.posts
			const hasImageSliding = posts.some(post => post.isImageSliding)
			if (hasImageSliding) {
				this.isTouching = false
				this.swiperOffset = 0
				return // 如果有图片正在滑动，不处理页面滑动
			}
			
			this.isTouching = false
			const screenWidth = uni.getSystemInfoSync().windowWidth
			const threshold = screenWidth * 0.2 // 滑动阈值：屏幕宽度的20%
			const deltaX = e.changedTouches[0].clientX - this.touchStartX
			
			// 根据滑动距离和方向切换页面
			if (Math.abs(deltaX) > threshold) {
				if (deltaX > 0 && this.currentTab === 'recommend') {
					// 向右滑动，从推荐页切换到关注页
					this.changeTab('follow', 0)
				} else if (deltaX < 0 && this.currentTab === 'follow') {
					// 向左滑动，从关注页切换到推荐页
					this.changeTab('recommend', 1)
				} else {
					// 滑动距离不够，回弹到当前页面
					this.swiperTransform = this.currentTab === 'follow' ? 0 : -50
				}
			} else {
				// 滑动距离不够，回弹到当前页面
				this.swiperTransform = this.currentTab === 'follow' ? 0 : -50
			}
			
			this.swiperOffset = 0
		},
		// 关注页下拉刷新
		onFollowRefresh() {
			this.followRefreshing = true
			// 模拟刷新数据
			setTimeout(() => {
				// 这里可以调用API刷新数据
				// 刷新完成后重置状态
				this.followRefreshing = false
				uni.showToast({
					title: '刷新成功',
					icon: 'success',
					duration: 1500
				})
			}, 1500)
		},
		onFollowRestore() {
			// 刷新恢复
			this.followRefreshing = false
		},
		// 推荐页下拉刷新
		onRecommendRefresh() {
			this.recommendRefreshing = true
			// 模拟刷新数据
			setTimeout(() => {
				// 这里可以调用API刷新数据
				// 刷新完成后重置状态
				this.recommendRefreshing = false
			}, 1500)
		},
		onRecommendRestore() {
			// 刷新恢复
			this.recommendRefreshing = false
		},
		handleRecommendCardTap(post) {
			// 推荐页瀑布流卡片点击 -> 跳转到笔记详情
			try {
				uni.setStorageSync('note-detail-temp', {
					...post
				})
			} catch (e) {
				console.error('缓存推荐笔记失败', e)
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
	min-height: 100vh;
	background: #f5f5f5;
	box-sizing: border-box;
}

.header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 0 34rpx 14rpx;
	position: fixed; /* 恢复固定定位 */
	top: 0;
	left: 0;
	width: 100%;
	z-index: 999;
	background: #f5f5f5;
	box-sizing: border-box;
	padding-top: calc(var(--status-bar-height) + 15rpx);
}

.header-left {
	display: flex;
	align-items: center;
	gap: 24rpx;
}

.brand {
	display: flex;
	align-items: center;
	justify-content: center;
	height: 80rpx;
}

.brand-logo {
	height: 80rpx;
	width: auto;
	min-width: 150rpx;
	max-width: 350rpx;
	display: block;
	object-fit: contain;
}

.header-right {
	display: flex;
	align-items: center;
	gap: 32rpx;
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

.header-right {
	display: flex;
	align-items: center;
	gap: 32rpx;
}

.tabs {
	display: flex;
	gap: 48rpx;
	margin: 0 10rpx;
	position: relative; /* 为指示器定位提供参考 */
}

.tab {
	position: relative;
	font-size: 30rpx;
	color: #ccc;
	font-weight: 400;
	padding-bottom: 4rpx;
	transition: color 0.3s ease; /* 文字颜色过渡 */
}

.tab.active {
	color: #000;
	font-weight: 600;
	font-size: 32rpx;
}

/* 下划线指示器 */
.tab-indicator {
	position: absolute;
	bottom: 0;
	height: 4rpx;
	background: #000;
	border-radius: 2rpx;
	transition: left 0.3s ease, width 0.3s ease;
}

.search-btn {
	width: 64rpx;
	height: 64rpx;
	border-radius: 50%;
	background: #333;
	display: flex;
	align-items: center;
	justify-content: center;
	margin-left: 12rpx;
}

.search-icon {
	font-size: 24rpx;
	color: #fff;
	line-height: 1;
	display: flex;
	align-items: center;
	justify-content: center;
}

/* 内容容器 */
.content-wrapper {
	width: 100%;
	height: 100vh;
	overflow: hidden;
	position: relative;
}

/* 滑动容器 */
.swiper-container {
	display: flex;
	width: 200%;
	height: 100%;
	will-change: transform;
	flex-direction: row;
}

.swiper-container.swiper-transition {
	transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.scroll {
	width: 50%;
	height: 100vh;
	flex-shrink: 0;
	padding-top: calc(var(--status-bar-height) + 115rpx); /* 预留顶部高度 (Header top 15 + Header content ~76 + margin) */
	box-sizing: border-box;
	overflow-x: hidden;
	position: relative; /* 为刷新器提供定位参考 */
}

/* 隐藏滚动条 */
::-webkit-scrollbar {
	width: 0;
	height: 0;
	background: transparent;
	display: none;
}

/* 关注页：单列全宽布局 */
.feed-list {
	padding: 0 32rpx;
	padding-bottom: 200rpx;
	background: #f5f5f5;
}

.post-card {
	background: #fff;
	margin-bottom: 12rpx;
	border-radius: 20rpx;
	overflow: hidden;
	opacity: 0;
	transition: opacity 0.6s ease-out;
}

.post-card.card-visible {
	opacity: 1;
}

/* 用户信息区域 */
.user-info {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 24rpx 20rpx 19rpx;
}

.user-left {
	display: flex;
	align-items: center;
	gap: 16rpx;
	flex: 1;
}

.user-avatar {
	width: 72rpx;
	height: 72rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.user-details {
	display: flex;
	flex-direction: column;
	gap: 6rpx;
	flex: 1;
}

.username-row {
	display: flex;
	align-items: center;
	gap: 12rpx;
	flex-wrap: wrap;
}

.post-card .username {
	font-size: 26rpx;
	font-weight: 600;
	color: #1b1b1b;
	line-height: 1.2;
}

.time-text-inline {
	font-size: 18rpx;
	color: #999;
	line-height: 1.2;
	white-space: nowrap;
}

.hashtags {
	display: flex;
	flex-wrap: wrap;
	gap: 12rpx;
}

.hashtag {
	font-size: 22rpx;
	color: #666;
	line-height: 1.2;
}

.message-btn {
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 12rpx;
	padding: 16rpx 24rpx;
	background: #1b1b1b;
	border-radius: 40rpx;
	flex-shrink: 0;
	min-height: 48rpx;
	box-sizing: border-box;
}

.message-icon {
	font-size: 20rpx;
	color: #fff;
	display: inline-flex;
	align-items: center;
	justify-content: center;
	line-height: 1;
	vertical-align: middle;
	flex-shrink: 0;
}

.message-text {
	font-size: 24rpx;
	color: #fff;
	font-weight: 500;
	line-height: 1;
	display: inline-flex;
	align-items: center;
	flex-shrink: 0;
}

/* 文本内容 */
.post-content {
	padding: 0 20rpx 24rpx;
}

.content-text {
	font-size: 28rpx;
	color: #1b1b1b;
	line-height: 1.6;
	word-break: break-all;
}

/* 媒体内容 */
.media-wrap {
	position: relative;
	width: 100%;
	background: #000;
	/* 确保容器高度自适应内容 */
	min-height: 0;
	overflow: hidden;
}

/* 单图时，容器高度完全由图片决定，移除黑色背景 */
.media-wrap-single {
	background: transparent;
	display: block;
}

/* 模糊背景层 - 填充图片两侧的黑色区域 */
.blur-background {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	width: 100%;
	height: 100%;
	z-index: 0;
	/* 应用模糊效果 */
	filter: blur(30rpx);
	-webkit-filter: blur(30rpx);
	/* 放大背景图片，确保模糊后能覆盖整个区域，包括两侧 */
	transform: scale(1.2);
	/* 确保背景覆盖整个容器 */
	background-size: cover !important;
	background-position: center !important;
	background-repeat: no-repeat !important;
	/* 添加轻微的暗化效果，让前景图片更突出 */
	opacity: 0.7;
	/* 确保背景层在图片下方 */
	pointer-events: none;
}

/* 单图时也显示模糊背景 */
.media-wrap-single .blur-background {
	display: block;
}

.media-image {
	width: 100%;
	display: block;
	/* 确保图片正确显示，避免黑色背景溢出 */
	height: auto;
	vertical-align: top;
	position: relative;
	z-index: 1;
}

/* 图片轮播 */
.image-carousel {
	position: relative;
	width: 100%;
	overflow: hidden;
	touch-action: pan-y;
	/* 初始时使用最小高度，避免黑色背景显示 */
	min-height: 200rpx;
	background: transparent;
	z-index: 1;
}

.image-carousel-container {
	display: flex;
	width: 100%;
	height: 100%;
	transition: transform 0.3s ease-out;
	will-change: transform;
	position: relative;
	z-index: 1;
}

.image-carousel-container .media-image {
	flex-shrink: 0;
	width: 100%;
	display: block;
}

/* 第一张图片正常显示 */
.image-carousel-container .first-image {
	height: auto;
	/* 确保第一张图片能正确显示，避免黑色背景 */
	display: block;
}

/* 后续图片适配显示 - 在固定高度容器内适配 */
.image-carousel-container .carousel-image:not(.first-image) {
	height: 100%;
	/* aspectFit 模式会自动保持宽高比并适配容器 */
}

.video-wrap {
	position: relative;
	width: 100%;
	/* 视频容器也需要自适应高度 */
	display: block;
	z-index: 1;
	/* 确保视频容器在模糊背景上方 */
	/* 创建定位上下文，让播放按钮能正确定位 */
	overflow: visible;
}

.video-poster {
	width: 100%;
	display: block;
	height: auto;
	position: relative;
	z-index: 1;
	/* 确保视频封面在模糊背景上方 */
}

.play-button {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 88rpx;
	height: 88rpx;
	border-radius: 50%;
	background: rgba(0, 0, 0, 0.55);
	display: flex;
	align-items: center;
	justify-content: center;
	backdrop-filter: blur(4rpx);
	z-index: 10;
	/* 确保播放按钮在视频封面上方显示 */
	pointer-events: auto;
	/* 确保按钮可见 */
	opacity: 1;
	visibility: visible;
}

.play-icon {
	font-size: 36rpx;
	color: #fff !important;
	margin-left: 4rpx;
	/* 确保图标可见 */
	display: inline-block !important;
	opacity: 1 !important;
	z-index: 11;
	/* 确保 Font Awesome 图标正确显示 */
	font-family: "Font Awesome 6 Free", "Font Awesome 6 Pro", "FontAwesome" !important;
	font-weight: 900;
}

/* 轮播指示器 */
.carousel-dots {
	position: absolute;
	bottom: 20rpx;
	left: 50%;
	transform: translateX(-50%);
	display: flex;
	gap: 12rpx;
	align-items: center;
	justify-content: center;
	z-index: 2;
}

.dot {
	width: 8rpx;
	height: 8rpx;
	border-radius: 50%;
	background: rgba(255, 255, 255, 0.5);
	transition: all 0.3s ease;
}

.dot.active {
	width: 20rpx;
	height: 8rpx;
	border-radius: 4rpx;
	background: rgba(255, 255, 255, 0.9);
}

/* 互动栏 */
.interaction-bar {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 24rpx 20rpx 34rpx;
	flex-wrap: nowrap;
	box-sizing: border-box;
}

.interaction-item {
	display: flex;
	align-items: center;
	gap: 8rpx;
	flex-shrink: 0;
}

.interaction-icon {
	font-size: 36rpx;
	color: #1b1b1b;
	transform-origin: center center;
	transition: all 0.3s ease;
}

.share-icon {
	font-size: 32rpx;
}

/* 点赞渐入动画 */
.like-icon.animate-like {
	animation: fadeInScale 0.6s ease-out;
}

/* 推荐页点赞透明度渐入动画 */
.like-icon-recommend.fade-in-like {
	animation: fadeInOpacity 0.6s ease-out;
}

@keyframes fadeInOpacity {
	0% {
		opacity: 0;
	}
	100% {
		opacity: 1;
	}
}

/* 收藏渐入动画 */
.favorite-icon.animate-favorite {
	animation: fadeInScale 0.6s ease-out;
}

/* 渐入缩放动画 */
@keyframes fadeInScale {
	0% {
		opacity: 0;
		transform: scale(0.3);
	}
	50% {
		transform: scale(1.2);
	}
	100% {
		opacity: 1;
		transform: scale(1);
	}
}

.interaction-count {
	font-size: 24rpx;
	color: #666;
	line-height: 1;
}

.post-time {
	margin-left: auto;
	display: flex;
	justify-content: flex-end;
	flex-shrink: 0;
	white-space: nowrap;
}

.time-text {
	font-size: 18rpx;
	color: #999;
	line-height: 1;
	white-space: nowrap;
}

/* 推荐页：瀑布流布局 */
.masonry {
	column-count: 2;
	column-gap: 12rpx;
	padding: 0 32rpx 200rpx; /* 增加左右内边距 32rpx 以对齐 Tabbar（左右各 32rpx padding） */
	/* 确保列间隙和行间隙一致 */
	column-fill: balance;
	/* 重置可能影响间隙的样式 */
	line-height: 0;
}

.masonry > * {
	line-height: normal;
}

.card {
	background: #fff;
	border-radius: 20rpx;
	/* box-shadow: 0 10rpx 24rpx rgba(0, 0, 0, 0.05); */
	overflow: hidden;
	display: inline-block;
	width: 100%;
	margin-bottom: 12rpx;
	break-inside: avoid;
	opacity: 0;
	transition: opacity 0.6s ease-out;
	/* 确保所有卡片使用相同的间隙 */
	box-sizing: border-box;
	/* 确保没有额外的 margin 或 padding 影响间隙 */
	margin-left: 0;
	margin-right: 0;
	margin-top: 0;
	vertical-align: top;
}

.card.card-visible {
	opacity: 1;
}

.publish-card {
	opacity: 1 !important; /* 发布卡片直接显示 */
	background: linear-gradient(135deg, #E7FF60 0%, #D4F050 50%, #C8E840 100%);
	height: 340rpx;
	padding: 0;
	margin-bottom: 12rpx; /* 确保与普通卡片使用相同的间隙 */
	box-sizing: border-box;
	position: relative;
	overflow: hidden;
}

/* 文字内容区域 */
.publish-text-wrapper {
	position: absolute;
	top: 85rpx;
	left: 30rpx;
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	transform: rotate(-3deg); /* 整体逆时针倾斜 */
	transform-origin: top left;
	z-index: 2;
}

.publish-text-line {
	font-size: 36rpx;
	font-weight: 900;
	color: #000;
	line-height: 1.3;
	display: block;
	transform: rotate(-3deg); /* 每行文字也保持倾斜 */
}

.publish-text-line-2 {
	margin-left: 20rpx; /* 第二行向右缩进 */
}

.publish-text-line-3 {
	margin-left: 40rpx; /* 第三行进一步向右缩进 */
}

/* 柯基犬插画 */
.publish-dog {
	position: absolute;
	top: 20rpx;
	right: 20rpx;
	width: 130rpx;
	height: 130rpx;
	z-index: 3;
	transform: rotate(-10deg); /* 向左倾斜 */
}

/* 相机图标 */
.publish-camera {
	position: absolute;
	bottom: -35rpx;
	left: 10rpx;
	width: 140rpx;
	height: 140rpx;
	z-index: 1;
	transform: rotate(20deg); /* 向右倾斜 */
}

/* GO按钮 */
.publish-btn {
	position: absolute;
	bottom: 20rpx;
	right: 24rpx;
	background: #1f1f1f;
	color: #E7FF60;
	padding: 12rpx 32rpx;
	border-radius: 999rpx;
	font-size: 24rpx;
	font-weight: 900;
	z-index: 2;
	white-space: nowrap;
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
	margin-left: 2rpx; /* 微调播放图标视觉居中 */
	font-size: 16rpx; /* 缩小播放图标 */
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
	font-size: 21.12rpx;
	color: #4a4a4a;
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

.likes .fa-regular {
	color: #1b1b1b;
}

.likes .fa-solid {
	color: #f25e5e;
}


.like-num {
	color: #777;
	line-height: 1;
	display: inline-flex;
	align-items: center;
}


/* 自定义刷新图标容器 */
.custom-refresher-wrapper {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	display: flex;
	justify-content: center;
	align-items: center;
	background: transparent;
	padding: 20rpx 0;
	width: 100%;
	box-sizing: border-box;
	z-index: 10;
	pointer-events: none;
}

/* 关注页刷新器特殊定位 - 考虑feed-list的padding，居中显示 */
.custom-refresher-follow {
	top: 0;
	left: 32rpx;
	right: 32rpx;
	width: calc(100% - 64rpx);
	justify-content: center;
}

/* 自定义刷新图标 - 旋转圆圈 */
.custom-refresher-icon {
	width: 40rpx;
	height: 40rpx;
	border: 5rpx solid transparent;
	border-top: 5rpx solid #383838;
	border-right: 5rpx solid #383838;
	border-radius: 50%;
	background: transparent;
	animation: customRefreshRotate 1s linear infinite;
	box-sizing: border-box;
}

/* 自定义刷新图标旋转动画 */
@keyframes customRefreshRotate {
	from {
		transform: rotate(0deg);
	}
	to {
		transform: rotate(360deg);
	}
}

/* 位置页样式 */
.location-content {
	width: 100%;
	height: 100vh;
}

.scroll-location {
	width: 100%;
	height: 100vh;
	flex-shrink: 0;
	padding-top: calc(var(--status-bar-height) + 125rpx);
	box-sizing: border-box;
	overflow-x: hidden;
	position: relative;
}

.location-page-content {
	padding: 40rpx;
	text-align: center;
	min-height: calc(100vh - var(--status-bar-height) - 115rpx);
}

.location-content-text {
	font-size: 32rpx;
	color: #666;
}

/* 位置页导航栏样式 */
.header-location {
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

.location-header-left {
	display: flex;
	align-items: center;
	flex-shrink: 0;
}

.location-icon-wrapper {
	display: flex;
	align-items: center;
	gap: 8rpx;
}

.location-icon-new {
	font-size: 26rpx;
	color: #bbb;
	display: inline-flex;
	align-items: center;
	line-height: 1;
}

.location-text-new {
	font-size: 28rpx;
	color: #999;
	font-weight: 400;
	line-height: 1;
	display: inline-flex;
	align-items: center;
}

.location-search-wrapper {
	flex: 1;
	display: flex;
	align-items: center;
	background: #e5e5e5;
	border-radius: 50rpx;
	padding: 6rpx;
	height: 70rpx;
	position: relative;
	overflow: hidden;
	box-sizing: border-box;
}

.location-search-input {
	flex: 1;
	height: 58rpx;
	padding: 0 20rpx;
	font-size: 28rpx;
	color: #333;
	background: transparent;
	border: none;
	outline: none;
	line-height: 58rpx;
}

.location-search-btn {
	height: 58rpx;
	padding: 0 32rpx;
	background: #383838;
	border-radius: 50rpx;
	margin-right: 0;
	margin-left: auto;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}

.location-search-btn-text {
	font-size: 28rpx;
	color: #fff;
	font-weight: 400;
	line-height: 1;
	display: flex;
	align-items: center;
}
</style>
