# Figma API 访问问题解决方案

## 问题
遇到 429 错误（请求频率过高），无法访问 Figma 设计稿。

## 解决方案

### 方案一：使用个人访问令牌（推荐）

1. **获取 Figma 个人访问令牌**：
   - 登录 Figma 网站
   - 点击右上角头像 → Settings
   - 在左侧菜单找到 "Account"
   - 滚动到 "Personal Access Tokens" 部分
   - 点击 "Create new token"
   - 输入令牌名称（如：Cursor AI）
   - 复制生成的令牌（只显示一次，请妥善保存）

2. **使用令牌访问**：
   - 将令牌提供给 AI 助手
   - AI 会使用令牌重新访问 Figma 设计稿

### 方案二：等待后重试

429 错误通常会在 5-10 分钟后自动解除。可以：
- 等待一段时间后重新请求
- 或者先基于图片描述进行开发，后续再精确调整

### 方案三：手动提供设计稿

如果无法获取令牌或等待时间过长，可以：
- 导出 Figma 设计稿为图片
- 或者提供设计稿的详细截图
- AI 可以基于图片进行开发

## Figma API 速率限制

- 免费账户：每分钟 60 次请求
- 专业账户：每分钟 120 次请求
- 使用个人访问令牌可以提高速率限制

## 参考链接

- Figma API 文档：https://www.figma.com/developers/api
- 速率限制说明：https://www.figma.com/developers/api#rate-limits
