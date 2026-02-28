# Figma MCP 配置说明

## 已配置的令牌
- **Personal Access Token**: `figd_Ti7y978Oe42CJkcAiEKKAqG6vP0PhBnUSOarD-nP`
- **File Key**: `S3owse8rBXszS33rYWwrJY`
- **Node ID**: `141-117`

## 配置文件位置

### 1. Cursor MCP 配置
配置文件已创建在：`.cursor/mcp.json`

### 2. 项目配置
配置文件已创建在：`mcp-config.json`

## 使用方法

### 在 Cursor 中配置 MCP
1. 打开 Cursor 设置（Settings）
2. 找到 MCP Servers 配置
3. 添加以下配置：

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-figma"
      ],
      "env": {
        "FIGMA_PERSONAL_ACCESS_TOKEN": "figd_Ti7y978Oe42CJkcAiEKKAqG6vP0PhBnUSOarD-nP"
      }
    }
  }
}
```

### 环境变量方式
也可以在系统环境变量中设置：
- Windows: `set FIGMA_PERSONAL_ACCESS_TOKEN=figd_Ti7y978Oe42CJkcAiEKKAqG6vP0PhBnUSOarD-nP`
- Mac/Linux: `export FIGMA_PERSONAL_ACCESS_TOKEN=figd_Ti7y978Oe42CJkcAiEKKAqG6vP0PhBnUSOarD-nP`

## 注意事项
- 令牌已保存在配置文件中，请确保不要将配置文件提交到公开的代码仓库
- 如果遇到 429 错误，可能是 MCP 服务器端的速率限制
- 建议将 `.cursor/mcp.json` 和 `mcp-config.json` 添加到 `.gitignore` 中
