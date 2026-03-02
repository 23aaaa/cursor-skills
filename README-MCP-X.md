# X (Twitter) MCP 配置说明

发布端 Skill 依赖 X/Twitter 的 MCP 才能自动发推。本项目已写好 MCP 配置，你只需补全密钥即可。

## 1. 获取 X API 凭证

1. 打开 [X Developer Portal](https://developer.x.com/en/products/x-api)（原 Twitter Developer）。
2. 创建或进入项目与 App，在 **Keys and tokens** 中生成或查看：
   - **API Key**（即 Consumer Key）
   - **API Secret Key**（即 Consumer Secret）
   - **Access Token**
   - **Access Token Secret**
3. 若使用 OAuth 2.0 用户认证，需在 **User authentication settings** 中配置：
   - **App permissions**：至少 **Read and Write**（发推需要 Write）。
   - **Type of App**：例如 **Web App**，Callback URL 可填 `http://localhost/`，Website URL 可填 `https://example.com`（按需修改）。

## 2. 填写到 MCP 配置

编辑项目根目录下的 **`.cursor/mcp.json`**，将 `twitter-mcp` 的 `env` 里四项占位文案替换为你的真实凭证：

- `API_KEY` → 你的 API Key  
- `API_SECRET_KEY` → 你的 API Secret Key  
- `ACCESS_TOKEN` → 你的 Access Token  
- `ACCESS_TOKEN_SECRET` → 你的 Access Token Secret  

**安全提示**：勿将填好真实密钥的 `mcp.json` 提交到公开仓库。若需版本管理，可把 `.cursor/mcp.json` 加入 `.gitignore`，或使用「复制为 `mcp.local.json` 再在本地填写」等方式。

## 3. 重启 Cursor

保存 `mcp.json` 后，**完全退出并重新打开 Cursor**，使 MCP 重新加载。

## 4. 验证

重启后，在对话中让 AI「发一条测试推」或再次调用「发布端」发 OpenClaw 串文。若配置正确，应能出现 `post_tweet` 等 MCP 工具并成功发推。

## 使用的 MCP 服务

- **名称**：twitter-mcp（[@enescinar/twitter-mcp](https://github.com/EnesCinr/twitter-mcp)）
- **能力**：`post_tweet`（发单条）、`search_tweets`（搜索）
- **发串文**：当前实现为多次调用 `post_tweet` 形成回复链；若该 MCP 后续支持 `post_thread`，发布端可改为调用线程接口。
