<template>
  <view class="settings-page">
    <!-- 账号与安全 -->
    <view class="section">
      <view class="section-title">账号与安全</view>
      <view class="card">
        <view class="cell" @tap="handleTap('phone')">
          <view class="cell-left">
            <text class="cell-label">手机号</text>
          </view>
          <view class="cell-right">
            <text class="cell-desc">{{ phoneMasked }}</text>
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>

        <view class="cell" @tap="handleTap('password')">
          <view class="cell-left">
            <text class="cell-label">登录密码</text>
          </view>
          <view class="cell-right">
            <text class="cell-desc">已设置</text>
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>

        <view class="cell" @tap="handleTap('realname')">
          <view class="cell-left">
            <text class="cell-label">实名认证</text>
          </view>
          <view class="cell-right">
            <text class="cell-desc">{{ realnameStatus }}</text>
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>

        <view class="cell" @tap="handleTap('wechat')">
          <view class="cell-left">
            <text class="cell-label">绑定微信账号</text>
          </view>
          <view class="cell-right">
            <text class="cell-desc">{{ wechatStatus }}</text>
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>

        

        <view class="cell cell-danger" @tap="handleTap('delete')">
          <view class="cell-left">
            <text class="cell-label cell-danger-text">注销账号</text>
          </view>
          <view class="cell-right">
            <text class="fa fa-angle-right cell-arrow cell-danger-text"></text>
          </view>
        </view>
      </view>
    </view>

    <!-- 隐私设置 -->
    <view class="section">
      <view class="section-title">隐私设置</view>
      <view class="card">
        <!-- 评论、@ 我 -->
        <view class="group-label">互动权限</view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">只允许我关注的人评论我</text>
          </view>
          <view class="cell-right">
            <switch :checked="onlyFollowedCanComment" @change="onSwitchChange('onlyFollowedCanComment', $event)" />
          </view>
        </view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">只允许我关注的人 @ 我</text>
          </view>
          <view class="cell-right">
            <switch :checked="onlyFollowedCanAtMe" @change="onSwitchChange('onlyFollowedCanAtMe', $event)" />
          </view>
        </view>

        <!-- 谁可以私信我 -->
        <view class="group-label">私信与收藏</view>
        <view class="cell" @tap="openDmPicker">
          <view class="cell-left">
            <text class="cell-label">谁可以私信我</text>
          </view>
          <view class="cell-right">
            <text class="cell-desc">{{ dmOptions[dmIndex] }}</text>
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>

        <picker mode="selector" :range="dmOptions" :value="dmIndex" @change="onDmPickerChange">
          <view class="hidden-picker"></view>
        </picker>

        <!-- 我的收藏 -->
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">公开我的收藏</text>
          </view>
          <view class="cell-right">
            <switch :checked="publicFavorites" @change="onSwitchChange('publicFavorites', $event)" />
          </view>
        </view>

        <!-- 找到我的方式 -->
        <view class="group-label">找到我的方式</view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">不把我推荐给可能认识的人</text>
          </view>
          <view class="cell-right">
            <switch :checked="disableRecommendToContacts" @change="onSwitchChange('disableRecommendToContacts', $event)" />
          </view>
        </view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">不展示在他人关系列表</text>
          </view>
          <view class="cell-right">
            <switch :checked="hideFromRelationList" @change="onSwitchChange('hideFromRelationList', $event)" />
          </view>
        </view>

        <!-- 关注与粉丝列表 -->
        <view class="group-label">关注与粉丝列表</view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">隐藏我的关注列表</text>
          </view>
          <view class="cell-right">
            <switch :checked="hideFollowingList" @change="onSwitchChange('hideFollowingList', $event)" />
          </view>
        </view>
        <view class="cell">
          <view class="cell-left">
            <text class="cell-label">隐藏我的粉丝列表</text>
          </view>
          <view class="cell-right">
            <switch :checked="hideFollowerList" @change="onSwitchChange('hideFollowerList', $event)" />
          </view>
        </view>
      </view>
    </view>

    <!-- 关于知宠 -->
    <view class="section">
      <view class="section-title">关于知宠</view>
      <view class="card">
        <view class="cell" @tap="handleTap('rules')">
          <view class="cell-left">
            <text class="cell-label">社区规范</text>
          </view>
          <view class="cell-right">
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>
        <view class="cell" @tap="handleTap('tos')">
          <view class="cell-left">
            <text class="cell-label">服务条款</text>
          </view>
          <view class="cell-right">
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>
        <view class="cell" @tap="handleTap('privacy')">
          <view class="cell-left">
            <text class="cell-label">隐私政策</text>
          </view>
          <view class="cell-right">
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>
        <view class="cell" @tap="handleTap('diagnose')">
          <view class="cell-left">
            <text class="cell-label">诊断网络</text>
          </view>
          <view class="cell-right">
            <text class="fa fa-angle-right cell-arrow"></text>
          </view>
        </view>
      </view>
    </view>

    <!-- 底部操作 -->
    <view class="bottom-actions">
      <button class="btn-outline" @tap="handleSwitchAccount">切换账号</button>
      <button class="btn-danger" @tap="handleLogout">退出登录</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      phone: '13800001234',
      realnameStatus: '未认证',
      wechatStatus: '未绑定',
      onlyFollowedCanComment: false,
      onlyFollowedCanAtMe: false,
      publicFavorites: true,
      disableRecommendToContacts: false,
      hideFromRelationList: false,
      hideFollowingList: false,
      hideFollowerList: false,
      dmOptions: ['默认', '我关注的人', '互相关注的人', '禁止私信'],
      dmIndex: 0
    }
  },
  computed: {
    phoneMasked() {
      if (!this.phone || this.phone.length < 7) return this.phone || ''
      return this.phone.substring(0, 3) + '****' + this.phone.substring(7)
    }
  },
  methods: {
    handleTap(type) {
      // 这里可以根据 type 跳转到对应页面或打开弹窗
      let message = ''
      switch (type) {
        case 'phone':
          message = '修改手机号'
          break
        case 'password':
          message = '修改登录密码'
          break
        case 'realname':
          message = '实名认证'
          break
        case 'wechat':
          message = '绑定微信账号'
          break
        case 'delete':
          message = '注销账号'
          break
        case 'rules':
          message = '社区规范'
          break
        case 'tos':
          message = '服务条款'
          break
        case 'privacy':
          message = '隐私政策'
          break
        case 'diagnose':
          message = '诊断网络'
          break
        default:
          message = ''
      }
      if (message) {
        uni.showToast({
          title: message + '（待接入）',
          icon: 'none'
        })
      }
    },
    onSwitchChange(key, e) {
      const value = e.detail ? e.detail.value : e.target.value
      this[key] = value
    },
    openDmPicker() {
      // uni-app 的 selector picker 需要点击 picker 本身，这里只做提示
      // 可以在不同端用条件编译封装更好的选择器
    },
    onDmPickerChange(e) {
      this.dmIndex = Number(e.detail.value || 0)
    },
    handleSwitchAccount() {
      uni.showToast({
        title: '切换账号（待接入）',
        icon: 'none'
      })
      // 这里可以跳转到登录页或账号选择页
      // uni.navigateTo({ url: '/pages/auth/login' })
    },
    handleLogout() {
      uni.showModal({
        title: '退出登录',
        content: '确定要退出当前账号吗？',
        success: (res) => {
          if (res.confirm) {
            // 清理本地登录状态后跳转登录页
            uni.reLaunch({
              url: '/pages/auth/login'
            })
          }
        }
      })
    }
  }
}
</script>

<style scoped>
.settings-page {
  min-height: 100vh;
  padding: 32rpx 32rpx 120rpx;
  box-sizing: border-box;
  background-color: #f5f5f5;
}

.section {
  margin-bottom: 32rpx;
}

.section-title {
  font-size: 28rpx;
  color: #9e9e9e;
  margin-bottom: 16rpx;
}

.card {
  background-color: #ffffff;
  border-radius: 24rpx;
  overflow: hidden;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.03);
}

.group-label {
  padding: 20rpx 32rpx 8rpx;
  font-size: 24rpx;
  color: #b0b0b0;
}

.cell {
  padding: 24rpx 32rpx;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-top-width: 1rpx;
  border-top-style: solid;
  border-top-color: #f3f3f3;
}

.cell:first-of-type {
  border-top-width: 0;
}

.cell-left {
  flex: 1;
  min-width: 0;
}

.cell-right {
  display: flex;
  align-items: center;
  gap: 12rpx;
  margin-left: 24rpx;
}

.cell-label {
  font-size: 28rpx;
  color: #1b1b1b;
}

.cell-desc {
  font-size: 24rpx;
  color: #9e9e9e;
}

.cell-arrow {
  font-size: 28rpx;
  color: #d0d0d0;
}

.cell-danger {
  margin-top: 8rpx;
}

.cell-danger-text {
  color: #ff4b4b;
}

.bottom-actions {
  margin-top: 40rpx;
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.btn-outline,
.btn-danger {
  height: 88rpx;
  line-height: 88rpx;
  border-radius: 999rpx;
  font-size: 28rpx;
}

.btn-outline {
  background-color: #ffffff;
  color: #1b1b1b;
  border: 1rpx solid #e0e0e0;
}

.btn-danger {
  background-color: #ff4b4b;
  color: #ffffff;
  border: none;
}

.hidden-picker {
  width: 0;
  height: 0;
  overflow: hidden;
}
</style>

