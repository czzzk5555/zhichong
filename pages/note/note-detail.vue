<template>
	<view class="page">
		<!-- 自定义顶部导航栏 -->
		<view class="top-nav">
			<view class="nav-left" @tap="handleBack">
				<text class="fa fa-angle-left back-icon"></text>
				<image class="nav-avatar" :src="note.avatar" mode="aspectFill" />
				<text class="nav-username">{{ note.author }}</text>
			</view>
			<view class="nav-right">
				<!-- 自己的笔记：只显示三个点 -->
				<view v-if="isOwnNote" class="nav-share" @tap.stop="handleShare">
					<text class="fa fa-ellipsis-vertical share-icon"></text>
				</view>
				<!-- 别人的笔记：显示关注按钮和分享图标 -->
				<template v-else>
					<view
						class="nav-follow-btn"
						:class="{ 'nav-followed': note.isFollowed }"
						@tap.stop="handleFollow"
					>
						<text class="nav-follow-text">
							{{ note.isFollowed ? '已关注' : '立即关注' }}
						</text>
					</view>
					<view class="nav-share" @tap.stop="handleShare">
						<text class="fa fa-arrow-up-from-bracket share-icon"></text>
					</view>
				</template>
			</view>
		</view>

		<scroll-view class="scroll" scroll-y :show-scrollbar="false">
			<!-- 顶部封面图：样式对齐关注页的媒体框架 -->
			<view
				class="cover-wrapper"
				v-if="imageList.length"
				:class="{ 'cover-wrapper-single': imageList.length === 1 }"
			>
				<!-- 模糊背景层 -->
				<view
					class="blur-background"
					:style="getDetailBlurBackgroundStyle()"
				></view>

				<view class="cover-inner">
					<!-- 单图 -->
					<image
						v-if="imageList.length === 1"
						class="cover-image"
						:src="imageList[0]"
						mode="widthFix"
						@tap="previewImage(0)"
					/>

					<!-- 多图轮播：所有图片高度跟第一张保持一致 -->
					<swiper
						v-else
						class="cover-swiper"
						:indicator-dots="false"
						circular
						:style="{ height: currentSwiperHeight + 'px' }"
						@change="onSwiperChange"
					>
						<swiper-item
							v-for="(img, index) in imageList"
							:key="index"
						>
							<image
								class="cover-image"
								:class="{ 'first-image': index === 0 }"
								:src="img"
								:mode="index === 0 ? 'widthFix' : 'aspectFit'"
								style="height: 100%;"
								@tap="previewImage(index)"
							/>
						</swiper-item>
					</swiper>

					<!-- 自定义轮播指示器，样式对齐关注页 -->
					<view
						v-if="imageList.length > 1"
						class="detail-carousel-dots"
					>
						<view
							v-for="(img, index) in imageList"
							:key="index"
							class="detail-dot"
							:class="{ active: index === currentSwiperIndex }"
						></view>
					</view>
				</view>
			</view>

			<!-- 标题区：主标题 + 副标题 -->
			<view class="header-section">
				<view class="title-row">
					<view class="title-left">
						<text class="main-title">
							{{ note.title || '周末去哪玩？带狗子去公园撒欢' }}
						</text>
						<text class="sub-title">
							{{ note.subTitle || '我们当初选择拿起剪刀，哪个不是怀里揣着一份对毛孩子最纯粹的爱？那份看到狗狗从脏兮兮、毛发打结，到在我们手里变得清爽、漂亮、神气活现时，内心涌起的成就感和幸福感。但是很多美容师心力憔悴，收入和付出完全不成正比。导致很多人对宠物行业失望转行。' }}
						</text>
					</view>
				</view>

				<!-- 标签 + 发布时间行：左侧标签，右侧时间 -->
				<view class="tags-time-row">
					<!-- 左侧笔记标签：若无数据则使用默认标签 -->
					<view class="tags-row" v-if="tagsToShow.length">
						<view
							class="tag-item"
							v-for="(tag, index) in tagsToShow"
							:key="index"
						>
							<text class="tag-text">#{{ tag }}</text>
						</view>
					</view>

					<!-- 右侧发布时间 -->
					<text class="publish-time">
						{{ note.publishTimeText || '发布于3小时前' }}
					</text>
				</view>
			</view>

			<!-- 正文内容 -->
			<view class="content-section" v-if="note.content">
				<text class="content-text">
					{{ note.content }}
				</text>
			</view>

			<!-- 评论区（简单占位结构，方便你后续自行扩展） -->
			<view class="comment-section">
				<view class="comment-divider"></view>
				<view class="comment-header">
					<view class="comment-header-left">
						<image
							class="comment-header-avatar"
							:src="note.avatar"
							mode="aspectFill"
						/>
						<view class="comment-header-input">
							<text class="comment-header-input-text">说点什么...</text>
						</view>
					</view>
					<text class="comment-header-count">
						共{{ note.comments }}条评论
					</text>
				</view>

				<view
					v-for="(comment, index) in comments"
					:key="index"
				>
					<view
						class="comment-item"
						:class="{ 'first-comment': index === 0 }"
					>
						<image
							class="comment-avatar"
							:src="comment.avatar"
							mode="aspectFill"
						/>
						<view class="comment-content">
							<view class="comment-top-row">
								<text class="comment-author">{{ comment.author }}</text>
								<view
									class="comment-author-tag"
									v-if="comment.isAuthor"
								>
									<text class="comment-author-tag-text">作者</text>
								</view>
							</view>
							<view class="comment-middle-row">
								<text class="comment-text">{{ comment.text }}</text>
								<view class="comment-meta">
									<text class="comment-time">{{ comment.time }}</text>
									<text class="comment-reply">回复</text>
								</view>
							</view>

						</view>
						<view class="comment-like" @tap="toggleCommentLike(comment)">
							<text
								class="fa fa-heart fa-solid"
								:style="{ color: comment.isLiked ? '#f25e5e' : '#ccc' }"
							></text>
							<text class="comment-like-count">
								{{ formatNumber(comment.likes) }}
							</text>
						</view>
					</view>

					<!-- 二级评论：放到主评论行下面，用同样的三列布局，保证右侧点赞对齐 -->
					<view
						class="reply-list-outer"
						v-if="comment.replies && comment.replies.length"
					>
						<view
							class="reply-item-outer"
							v-for="(reply, rIndex) in comment.replies"
							:key="rIndex"
						>
							<view class="reply-left">
								<image
									class="reply-avatar"
									:src="reply.avatar"
									mode="aspectFill"
								/>
								<view class="reply-content">
									<view class="reply-top-row">
										<text class="reply-author">{{ reply.author }}</text>
										<view
											class="reply-author-tag"
											v-if="reply.isAuthor"
										>
											<text class="reply-author-tag-text">作者</text>
										</view>
									</view>
									<view class="reply-second-row">
										<text class="reply-text">{{ reply.text }}</text>
										<view class="reply-meta-row">
											<text class="reply-meta">{{ reply.time }}</text>
											<text class="reply-meta">回复</text>
										</view>
									</view>
								</view>
							</view>
							<view class="comment-like reply-like-outer" @tap.stop="toggleReplyLike(reply)">
								<text
									class="fa fa-heart fa-solid"
									:style="{ color: reply.isLiked ? '#f25e5e' : '#ccc' }"
								></text>
								<text class="comment-like-count">
									{{ formatNumber(reply.likes) }}
								</text>
							</view>
						</view>
					</view>

					<!-- 分割线 -->
					<view
						class="comment-divider-line"
					></view>
				</view>
			</view>
		</scroll-view>

		<!-- 弹出菜单 -->
		<view v-if="showMenu" class="menu-mask" @tap="closeMenu">
			<view class="menu-popup" @tap.stop>
				<view class="menu-item" @tap="handleMessageFriend">
					<text class="fa fa-envelope menu-icon"></text>
					<text class="menu-text">私信好友</text>
				</view>
				<view class="menu-item" @tap="handleEdit" v-if="isOwnNote">
					<text class="fa fa-edit menu-icon"></text>
					<text class="menu-text">编辑</text>
				</view>
				<view class="menu-item" @tap="handleCopyLink">
					<text class="fa fa-link menu-icon"></text>
					<text class="menu-text">复制链接</text>
				</view>
				<view class="menu-item" @tap="handlePermission" v-if="isOwnNote">
					<text class="fa fa-shield menu-icon"></text>
					<text class="menu-text">权限管理</text>
				</view>
				<view class="menu-item menu-item-danger" @tap="handleDelete" v-if="isOwnNote">
					<text class="fa fa-trash menu-icon"></text>
					<text class="menu-text">删除</text>
				</view>
			</view>
		</view>

		<!-- 底部操作栏（按截图顺序：点赞-评论-收藏-输入框） -->
		<view class="bottom-bar">
			<view class="bottom-action" @tap="handleLike">
					<text
						class="fa fa-heart bottom-icon"
						:class="note.isLiked ? 'fa-solid' : 'fa-regular'"
						:style="{ color: note.isLiked ? '#ff0000' : '#1b1b1b' }"
					></text>
				<text class="bottom-count">
					{{ formatNumber(note.likes) }}
				</text>
			</view>
			<view class="bottom-action" @tap="handleCommentTap">
					<text class="fa-regular fa-comment bottom-icon" style="color: #1b1b1b;"></text>
				<text class="bottom-count">
					{{ formatNumber(note.comments) }}
				</text>
			</view>
			<view class="bottom-action" @tap="handleCollect">
					<text
						class="fa fa-star bottom-icon"
						:class="note.isCollected ? 'fa-solid' : 'fa-regular'"
						:style="{ color: note.isCollected ? '#ffd700' : '#1b1b1b' }"
					></text>
				<text class="bottom-count">
					{{ formatNumber(note.collects) }}
				</text>
			</view>
			<view class="input-placeholder" @tap="handleWriteComment">
				<text class="fa fa-pen bottom-input-icon"></text>
				<text class="input-placeholder-text">说点什么...</text>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			note: {
				id: null,
				title: '',
				subTitle: '',
				author: 'EULA',
				avatar:
					'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=facearea&w=96&h=96&q=80',
				image: '',
				content: '',
				tags: [],
				views: 0,
				likes: 0,
				collects: 0,
				comments: 0,
				isLiked: false,
				isCollected: false,
				isFollowed: false,
				isOfficial: false,
				publishTimeText: ''
			},
			// 默认展示用的标签（当没有传入 tags 时兜底）
			defaultTags: ['宠物', '摄影', '宠物店转让'],
			// 顶部轮播图列表，兼容单图与多图
			imageList: [],
			// 多图轮播时，每张图片对应的高度（px），这里会统一使用第一张的高度
			swiperHeights: [],
			// 当前展示图片对应的高度（px），用于设置 swiper 高度
			currentSwiperHeight: 0,
			// 当前轮播索引，用于模糊背景选择图片
			currentSwiperIndex: 0,
			comments: [],
			// 是否是自己的笔记
			isOwnNote: false,
			// 是否显示弹出菜单
			showMenu: false
		}
	},
	computed: {
		// 优先使用真实标签，缺失时使用默认兜底标签，保证界面有内容
		tagsToShow() {
			if (Array.isArray(this.note.tags) && this.note.tags.length) {
				return this.note.tags
			}
			return this.defaultTags
		}
	},
	onLoad(options) {
		try {
			const cached = uni.getStorageSync('note-detail-temp')
			if (cached) {
				this.note = Object.assign({}, this.note, cached)
				// 判断是否是自己的笔记：优先使用缓存中的 isOwnNote，否则判断作者是否是当前用户
				if (cached.isOwnNote !== undefined) {
					this.isOwnNote = cached.isOwnNote
				} else {
					// 判断作者是否是当前用户（这里假设当前用户是 'EULA'，后续可以改为从全局状态获取）
					this.isOwnNote = cached.author === 'EULA'
				}
			} else {
				// 没有缓存时，使用一个简单占位
				this.note = Object.assign({}, this.note, {
					title: '笔记详情占位标题',
					subTitle: '这里是副标题占位文案',
					content:
						'这里是笔记详情的占位文案，后续你可以接入真实数据，展示完整的图文内容、话题标签和互动信息。',
					// 占位多图
					images: [
						'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1517849845537-4d257902454a?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1517840933442-d2d1a05edb75?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?auto=format&fit=crop&w=900&q=80',
						'https://images.unsplash.com/photo-1596495578065-6c4195bb8389?auto=format&fit=crop&w=900&q=80'
					],
					views: 1234,
					likes: 256,
					collects: 876,
					comments: 12,
					tags: ['遛狗日常', '宠物摄影']
				})
			}

			// 统一整理轮播图数据，优先使用 note.images，其次回退到单图 note.image
			const images = []
			if (Array.isArray(this.note.images) && this.note.images.length) {
				images.push(...this.note.images)
			} else if (this.note.image) {
				images.push(this.note.image)
			}
			this.imageList = images

			// 计算多图轮播中每张图片的显示高度（全部跟第一张保持一致）
			if (this.imageList.length > 1) {
				this.initSwiperHeights()
			}
		} catch (e) {
			console.error('加载笔记详情失败', e)
		}

		// 初始化简单评论占位数据（两条评论；第二条带一条回复，样式对齐你给的图）
		this.comments = [
			{
				author: '小小肉酱',
				avatar:
					'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
				time: '3小时前',
				text: '多少钱转',
				likes: 20,
				isLiked: false
			},
			{
				author: '小王',
				avatar:
					'https://images.unsplash.com/photo-1520813792240-56fc4a3765a7?auto=format&fit=facearea&w=96&h=96&q=80',
				time: '2小时前',
				text: '多少钱转',
				likes: 5,
				isLiked: false,
				replies: [
					{
						author: '小小肉酱',
						isAuthor: true,
						avatar:
							'https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=facearea&w=96&h=96&q=80',
						time: '3小时前',
						text: '私信我',
						likes: 5,
						isLiked: false
					}
				]
			}
		]
	},
	methods: {
		formatNumber(num) {
			const n = Number(num) || 0
			if (n >= 10000) {
				return (n / 10000).toFixed(1) + 'W'
			}
			return n.toString()
		},
		// 计算轮播中图片的显示高度（单位：px），所有图片高度跟第一张保持一致
		initSwiperHeights() {
			const systemInfo = uni.getSystemInfoSync()
			const windowWidth = systemInfo.windowWidth || 375 // px

			if (!this.imageList.length) return

			const firstSrc = this.imageList[0]

			this.swiperHeights = []
			this.currentSwiperHeight = 0

			uni.getImageInfo({
				src: firstSrc,
				success: res => {
					const ratio = res.height / res.width || 1
					const displayHeight = windowWidth * ratio
					// 所有图片高度都跟第一张保持一致
					this.swiperHeights = this.imageList.map(() => displayHeight)
					this.currentSwiperHeight = displayHeight
				},
				fail: () => {
					// 获取失败时给一个合理的默认高度
					const fallback = windowWidth * (3 / 4)
					this.swiperHeights = this.imageList.map(() => fallback)
					this.currentSwiperHeight = fallback
				}
			})
		},
		// 轮播切换时，根据当前图片调整容器高度
		onSwiperChange(e) {
			const current = e.detail.current || 0
			this.currentSwiperIndex = current
			const h = this.swiperHeights[current]
			if (h) {
				this.currentSwiperHeight = h
			}
		},
		previewImage(currentIndex = 0) {
			if (!this.imageList.length) return
			uni.previewImage({
				current: currentIndex,
				urls: this.imageList
			})
		},
		// 详情页模糊背景样式（对齐关注页媒体框架）
		getDetailBlurBackgroundStyle() {
			if (!this.imageList.length) {
				return {
					display: 'none'
				}
			}

			const index = this.currentSwiperIndex || 0
			const imageUrl = this.imageList[index] || this.imageList[0]

			return {
				backgroundImage: `url(${imageUrl})`,
				backgroundSize: 'cover',
				backgroundPosition: 'center',
				backgroundRepeat: 'no-repeat',
				display: 'block'
			}
		},
		handleFollow() {
			this.note.isFollowed = !this.note.isFollowed
			uni.showToast({
				title: this.note.isFollowed ? '已关注作者' : '已取消关注',
				icon: 'none'
			})
		},
		handleLike() {
			this.note.isLiked = !this.note.isLiked
			if (this.note.isLiked) {
				this.note.likes = (this.note.likes || 0) + 1
			} else {
				this.note.likes = Math.max(0, (this.note.likes || 0) - 1)
			}
		},
		handleCollect() {
			this.note.isCollected = !this.note.isCollected
			uni.showToast({
				title: this.note.isCollected ? '已收藏' : '已取消收藏',
				icon: 'none'
			})
		},
	handleCommentTap() {
		// 预留：可滚动到评论区或聚焦输入框
		this.handleWriteComment()
	},
		handleWriteComment() {
			uni.showToast({
				title: '评论输入框后续接入',
				icon: 'none'
			})
		},
		toggleCommentLike(comment) {
			comment.isLiked = !comment.isLiked
			if (comment.isLiked) {
				comment.likes = (comment.likes || 0) + 1
			} else {
				comment.likes = Math.max(0, (comment.likes || 0) - 1)
			}
		},
		toggleReplyLike(reply) {
			reply.isLiked = !reply.isLiked
			if (reply.isLiked) {
				reply.likes = (reply.likes || 0) + 1
			} else {
				reply.likes = Math.max(0, (reply.likes || 0) - 1)
			}
		},
		handleBack() {
			uni.navigateBack({
				delta: 1
			})
		},
		handleShare() {
			// 显示弹出菜单
			this.showMenu = true
		},
		closeMenu() {
			// 关闭弹出菜单
			this.showMenu = false
		},
		handleMessageFriend() {
			this.closeMenu()
			uni.showToast({
				title: '私信好友功能待接入',
				icon: 'none'
			})
		},
		handleEdit() {
			this.closeMenu()
			uni.showToast({
				title: '编辑功能待接入',
				icon: 'none'
			})
		},
		handleCopyLink() {
			this.closeMenu()
			// 复制链接到剪贴板
			// 这里需要根据实际情况生成笔记链接
			const link = `https://zhichong.com/note/${this.note.id || '123'}`
			uni.setClipboardData({
				data: link,
				success: () => {
					uni.showToast({
						title: '链接已复制',
						icon: 'success'
					})
				},
				fail: () => {
					uni.showToast({
						title: '复制失败',
						icon: 'none'
					})
				}
			})
		},
		handlePermission() {
			this.closeMenu()
			uni.showToast({
				title: '权限管理功能待接入',
				icon: 'none'
			})
		},
		handleDelete() {
			this.closeMenu()
			uni.showModal({
				title: '确认删除',
				content: '确定要删除这条笔记吗？删除后无法恢复。',
				confirmText: '删除',
				confirmColor: '#ff0000',
				success: (res) => {
					if (res.confirm) {
						// 将删除的笔记标识存储到缓存，供 profile.vue 使用
						try {
							// 使用 title + author 作为唯一标识
							const deleteKey = `${this.note.title}_${this.note.author}`
							uni.setStorageSync('note-delete-temp', {
								title: this.note.title,
								author: this.note.author
							})
							
							uni.showToast({
								title: '删除成功',
								icon: 'success',
								duration: 1500
							})
							
							// 延迟返回，让用户看到删除成功提示
							setTimeout(() => {
								uni.navigateBack({
									delta: 1
								})
							}, 1500)
						} catch (e) {
							console.error('删除笔记失败', e)
							uni.showToast({
								title: '删除失败，请重试',
								icon: 'none'
							})
						}
					}
				}
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
	box-sizing: border-box;
	position: relative;
}

.scroll {
	width: 100%;
	height: 100vh;
	box-sizing: border-box;
	/* 对齐你截图里的 3.75rem，按 1rem≈16px≈32rpx 折算，大约是 120rpx */
	padding-top: calc(var(--status-bar-height) + 120rpx);
	padding-bottom: 120rpx; /* 预留底部操作栏空间，确保内容不被遮挡 */
}

.top-nav {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	padding: calc(var(--status-bar-height) + 30rpx) 24rpx 30rpx;
	box-sizing: border-box;
	background: #ffffff;
	display: flex;
	align-items: center;
	justify-content: space-between;
	z-index: 20;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.03);
}

.nav-left {
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.back-icon {
	font-size: 40rpx;
	color: #333;
}

.nav-avatar {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
}

.nav-username {
	font-size: 28rpx;
	color: #1b1b1b;
	font-weight: 500;
}

.nav-right {
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.nav-follow-btn {
	min-width: 150rpx;
	height: 60rpx;
	padding: 0 28rpx;
	border-radius: 999rpx;
	background: linear-gradient(135deg, #E7FF60 0%, #D4F050 50%, #C8E840 100%);
	display: flex;
	align-items: center;
	justify-content: center;
}

.nav-followed {
	min-width: 120rpx; /* 关注后稍微窄一点 */
	background: #f0f0f0; /* 灰色标签背景 */
}

.nav-follow-text {
	font-size: 24rpx;
	color: #1b1b1b;
	font-weight: 500;
}

.nav-share {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
	border: none;
	display: flex;
	align-items: center;
	justify-content: center;
}

.share-icon {
	font-size: 38rpx;
	color: #111;
}

.cover-wrapper {
	position: relative;
	width: 100%;
	min-height: 0;
	/* 背景改为白色，与页面统一 */
	background: #ffffff;
	overflow: hidden;
}

.cover-wrapper-single {
	background: transparent;
}

.cover-inner {
	position: relative;
	z-index: 1;
}

/* 模糊背景层，对齐关注页样式 */
.blur-background {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	width: 100%;
	height: 100%;
	z-index: 0;
	filter: blur(30rpx);
	-webkit-filter: blur(30rpx);
	transform: scale(1.2);
	background-size: cover !important;
	background-position: center !important;
	background-repeat: no-repeat !important;
	opacity: 0.7;
	pointer-events: none;
}

.cover-swiper {
	width: 100%;
	height: 100%;
}

.cover-image {
	width: 100%;
	height: auto;
	display: block;
	position: relative;
	z-index: 1;
}

/* 详情页轮播指示器，对齐关注页样式 */
.detail-carousel-dots {
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

.detail-dot {
	width: 8rpx;
	height: 8rpx;
	border-radius: 50%;
	background: rgba(255, 255, 255, 0.5);
	transition: all 0.3s ease;
}

.detail-dot.active {
	width: 20rpx;
	height: 8rpx;
	border-radius: 4rpx;
	background: rgba(255, 255, 255, 0.9);
}

.header-section {
	padding: 24rpx 32rpx 12rpx;
	background: #fff;
}

.title-row {
	display: flex;
	align-items: flex-start;
}

.title-left {
	width: 100%;
}

.main-title {
	display: block;
	font-size: 32rpx;
	font-weight: 700;
	color: #1b1b1b;
	line-height: 1.5;
	margin-bottom: 8rpx;
}

.sub-title {
	display: block;
	font-size: 26rpx;
	color: #1b1b1b;
	line-height: 1.5;
}

.tags-time-row {
	margin-top: 16rpx;
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16rpx;
}

.tags-row {
	flex: 1;
	display: flex;
	flex-wrap: wrap;
	gap: 12rpx;
}

.tag-item {
	padding: 6rpx 14rpx;
	border-radius: 20rpx;
	background: #f5f5f5;
}

.tag-text {
	font-size: 22rpx;
	color: #666;
}

.publish-time {
	font-size: 22rpx;
	color: #999;
	flex-shrink: 0;
	white-space: nowrap;
}

.content-section {
	padding: 12rpx 32rpx 24rpx;
	background: #fff;
}

.content-text {
	font-size: 28rpx;
	color: #1b1b1b;
	line-height: 1.8;
	white-space: pre-wrap;
}

.comment-section {
	/* 整体往上贴一点，让上方那条分割线更靠近内容 */
	margin-top: 0;
	padding: 20rpx 32rpx 40rpx;
	background: #fff;
}

.comment-divider {
	width: 100%;
	height: 1px;
	background: #f0f0f0;
	margin: 0 0 24rpx;
}

.comment-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	margin-bottom: 24rpx;
}

.comment-header-left {
	flex: 1;
	display: flex;
	align-items: center;
	gap: 16rpx;
}

.comment-header-avatar {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.comment-header-input {
	flex: 1;
	height: 60rpx;
	border-radius: 999rpx;
	background: #f3f3f3;
	display: flex;
	align-items: center;
	padding: 0 24rpx;
	box-sizing: border-box;
}

.comment-header-input-text {
	font-size: 24rpx;
	color: #999;
}

.comment-header-count {
	font-size: 24rpx;
	color: #999;
	margin-left: 16rpx;
	white-space: nowrap;
}

.comment-item {
	display: flex;
	/* 让头像与昵称这一行对齐（不要垂直居中到两行中间） */
	align-items: flex-start;
	gap: 20rpx;
	margin-bottom: 12rpx;
}

.comment-item.first-comment {
	margin-top: 35rpx;
}

.comment-avatar {
	width: 56rpx;
	height: 56rpx;
	border-radius: 50%;
	flex-shrink: 0;
	/* 轻微下移，视觉上更贴合昵称基线 */
	margin-top: 10rpx;
}

.comment-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 6rpx;
}

.comment-top-row {
	display: flex;
	align-items: center;
	gap: 12rpx;
}

.comment-author {
	font-size: 24rpx;
	color: #333;
	font-weight: 500;
}

.comment-author-tag {
	padding: 2rpx 10rpx;
	border-radius: 999rpx;
	background: #E7FF60;
	display: flex;
	align-items: center;
	justify-content: center;
}

.comment-author-tag-text {
	font-size: 18rpx;
	color: #1b1b1b;
}

.comment-like {
	width: 60rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: flex-start;
	gap: 4rpx;
	padding-top: 4rpx;
}

.comment-like text.fa {
	width: 24rpx;
	text-align: center;
	font-size: 24rpx;
	display: inline-block;
	line-height: 1;
}

.comment-like-count {
	font-size: 22rpx;
	color: #999;
	width: 100%;
	text-align: center;
	min-width: 30rpx;
}

/* 单条评论下方的分割线 */
.comment-divider-line {
	/* 略短一些并靠右对齐，效果类似你截图中的红线 */
	width: 89%;
	height: 1px;
	background: #f0f0f0;
	margin: 16rpx 0 16rpx auto;
}

.comment-middle-row {
	display: flex;
	align-items: center;
	justify-content: flex-start;
	gap: 8rpx;
}

.comment-text {
	font-size: 26rpx;
	color: #1b1b1b;
	line-height: 1.6;
}

.comment-meta {
	display: flex;
	align-items: center;
	gap: 12rpx;
	flex-shrink: 0;
	white-space: nowrap;
}

.comment-time {
	font-size: 20rpx;
	color: #aaa;
}

.comment-reply {
	font-size: 22rpx;
	color: #999;
}

/* 二级评论样式 */
/* 二级评论：独立一块，右侧点赞与主评论同列对齐 */
.reply-list-outer {
	margin-top: 12rpx;
	/* 从主评论头像右侧开始对齐 */
	padding-left: 76rpx; /* 56(主头像) + 20(gap) */
	display: flex;
	flex-direction: column;
	gap: 12rpx;
}

.reply-item-outer {
	display: flex;
	align-items: flex-start;
	justify-content: space-between;
	gap: 20rpx;
}

.reply-left {
	flex: 1;
	min-width: 0;
	display: flex;
	align-items: flex-start;
	gap: 12rpx;
}

.reply-avatar {
	width: 40rpx;
	height: 40rpx;
	border-radius: 50%;
	flex-shrink: 0;
}

.reply-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 2rpx;
	min-width: 0;
}

.reply-like-outer {
	padding-top: 2rpx; /* 让心形更贴合回复首行 */
}

.reply-top-row {
	display: flex;
	align-items: center;
	gap: 8rpx;
}

.reply-author {
	font-size: 24rpx;
	color: #333;
	font-weight: 500;
}

.reply-author-tag {
	padding: 2rpx 10rpx;
	border-radius: 999rpx;
	background: #E7FF60;
	display: flex;
	align-items: center;
	justify-content: center;
}

.reply-author-tag-text {
	font-size: 18rpx;
	color: #1b1b1b;
}

.reply-second-row {
	display: flex;
	align-items: center;
	justify-content: flex-start;
	gap: 16rpx;
}

.reply-text {
	font-size: 26rpx;
	color: #1b1b1b;
	line-height: 1.5;
}

.reply-meta {
	font-size: 20rpx;
	color: #999;
}

.reply-meta-row {
	display: flex;
	align-items: center;
	gap: 12rpx;
	flex-shrink: 0;
}

/* 回复点赞：样式与主评论点赞对齐（固定宽度，避免跳位） */
.reply-like {
	width: 60rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: flex-start;
	gap: 4rpx;
	padding-top: 4rpx;
	flex-shrink: 0;
}

.reply-like text.fa {
	width: 24rpx;
	text-align: center;
	font-size: 24rpx;
	display: inline-block;
	line-height: 1;
}

.reply-like-count {
	font-size: 22rpx;
	color: #999;
	width: 100%;
	text-align: center;
	min-width: 30rpx;
}

.bottom-bar {
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	height: 120rpx;
	padding: 0 28rpx;
	box-sizing: border-box;
	background: rgba(255, 255, 255, 0.96);
	display: flex;
	align-items: center;
	gap: 24rpx;
	box-shadow: 0 -4rpx 16rpx rgba(0, 0, 0, 0.06);
}

.input-placeholder {
	flex: 1;
	height: 68rpx;
	border-radius: 32rpx;
	background: #f5f5f5;
	display: flex;
	align-items: center;
	padding: 0 26rpx;
	box-sizing: border-box;
}

.input-placeholder-text {
	font-size: 24rpx;
	color: #999;
}

.bottom-action {
	display: flex;
	align-items: center;
	gap: 8rpx;
	min-width: 60rpx;
	justify-content: center;
}

.bottom-icon {
	width: 32rpx;
	text-align: center;
	display: inline-block;
	font-size: 28rpx;
	line-height: 1;
}

.bottom-count {
	font-size: 30rpx;
	color: #1b1b1b;
}

.bottom-input-icon {
	font-size: 28rpx;
	color: #999;
	margin-right: 12rpx;
}

/* 弹出菜单样式 */
.menu-mask {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.5);
	z-index: 9999;
	display: flex;
	align-items: flex-end;
	justify-content: center;
}

.menu-popup {
	width: 100%;
	background: #ffffff;
	border-radius: 32rpx 32rpx 0 0;
	padding: 32rpx 0;
	box-sizing: border-box;
	animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
	from {
		transform: translateY(100%);
	}
	to {
		transform: translateY(0);
	}
}

.menu-item {
	display: flex;
	align-items: center;
	padding: 32rpx 48rpx;
	gap: 24rpx;
	transition: background-color 0.2s;
}

.menu-item:active {
	background-color: #f5f5f5;
}

.menu-item-danger {
	color: #ff0000;
}

.menu-item-danger .menu-icon,
.menu-item-danger .menu-text {
	color: #ff0000;
}

.menu-icon {
	font-size: 36rpx;
	color: #1b1b1b;
	width: 40rpx;
	text-align: center;
}

.menu-text {
	font-size: 32rpx;
	color: #1b1b1b;
	flex: 1;
}
</style>

