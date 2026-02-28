<template>
	<view class="page">
		<view class="card">
			<text class="title">注册</text>
			<text class="sub">基础注册页（后续你再做 UI）</text>

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
					placeholder="密码（至少6位）"
					placeholder-style="color:#999;"
				/>
				<input
					class="input"
					v-model="form.confirmPassword"
					type="password"
					placeholder="确认密码"
					placeholder-style="color:#999;"
				/>
				<button class="btn" :disabled="loading" @tap="onRegister">
					{{ loading ? '注册中...' : '注册' }}
				</button>
			</view>

			<view class="links">
				<text class="link" @tap="toLogin">已有账号？去登录</text>
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
				password: '',
				confirmPassword: ''
			}
		}
	},
	methods: {
		async onRegister() {
			if (!this.form.account || !this.form.password || !this.form.confirmPassword) {
				return uni.showToast({ title: '请填写完整信息', icon: 'none' })
			}
			if (this.form.password.length < 6) {
				return uni.showToast({ title: '密码至少6位', icon: 'none' })
			}
			if (this.form.password !== this.form.confirmPassword) {
				return uni.showToast({ title: '两次密码不一致', icon: 'none' })
			}

			this.loading = true
			try {
				// TODO: 这里接入你的注册接口（uni-id / 自建后端均可）
				await new Promise((r) => setTimeout(r, 400))

				uni.showToast({ title: '注册成功(占位)', icon: 'none' })
				uni.redirectTo({ url: '/pages/auth/login' })
			} catch (e) {
				uni.showToast({ title: e?.message || '注册失败', icon: 'none' })
			} finally {
				this.loading = false
			}
		},
		toLogin() {
			uni.navigateBack()
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

