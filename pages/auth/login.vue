<template>
	<view class="page">
		<view class="card">
			<text class="title">登录</text>
			<text class="sub">先用基础页面占位，后续你再按设计稿美化</text>

			<view class="form">
				<input
					class="input"
					v-model.trim="form.account"
					placeholder="手机号/邮箱/用户名"
					placeholder-style="color:#999;"
				/>
				<input
					class="input"
					v-model="form.password"
					type="password"
					placeholder="密码"
					placeholder-style="color:#999;"
				/>
				<button class="btn" :disabled="loading" @tap="onLogin">
					{{ loading ? '登录中...' : '登录' }}
				</button>
			</view>

			<view class="links">
				<text class="link" @tap="toRegister">注册账号</text>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			loading: false,
			form: {
				account: '',
				password: ''
			}
		}
	},
	methods: {
		async onLogin() {
			if (!this.form.account || !this.form.password) {
				return uni.showToast({ title: '请填写账号和密码', icon: 'none' })
			}

			this.loading = true
			try {
				// TODO: 这里接入你的登录接口（uni-id / 自建后端均可）
				// 示例：uniCloud.callFunction({ name: 'uni-id-cf', data: { action: 'login', params: this.form } })
				await new Promise((r) => setTimeout(r, 400))

				// 占位：写入一个token，供Tabbar拦截逻辑判断是否已登录
				uni.setStorageSync('token', 'mock-token')
				uni.showToast({ title: '登录成功(占位)', icon: 'none' })
				// 使用 reLaunch 回到 App 首页（你用的是自定义 tabbar，而不是 pages.json 的 tabBar）
				uni.reLaunch({ url: '/pages/index/index' })
			} catch (e) {
				uni.showToast({ title: e?.message || '登录失败', icon: 'none' })
			} finally {
				this.loading = false
			}
		},
		toRegister() {
			uni.navigateTo({ url: '/pages/auth/register' })
		}
	}
}
</script>

<style scoped>
.page {
	min-height: 100vh;
	background: #f5f5f5;
	padding: 48rpx 32rpx;
	box-sizing: border-box;
}

.card {
	background: #fff;
	border-radius: 24rpx;
	padding: 40rpx 32rpx;
	box-sizing: border-box;
}

.title {
	font-size: 44rpx;
	font-weight: 700;
	color: #111;
	display: block;
}

.sub {
	margin-top: 10rpx;
	font-size: 26rpx;
	color: #777;
	display: block;
}

.form {
	margin-top: 28rpx;
	display: flex;
	flex-direction: column;
	gap: 18rpx;
}

.input {
	height: 92rpx;
	padding: 0 24rpx;
	border-radius: 18rpx;
	background: #f6f6f6;
	box-sizing: border-box;
	font-size: 30rpx;
	color: #111;
}

.btn {
	margin-top: 10rpx;
	height: 92rpx;
	border-radius: 18rpx;
	background: #111;
	color: #fff;
	font-size: 32rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.btn[disabled] {
	opacity: 0.6;
}

.links {
	margin-top: 24rpx;
	display: flex;
	justify-content: center;
}

.link {
	color: #111;
	font-size: 28rpx;
}
</style>

