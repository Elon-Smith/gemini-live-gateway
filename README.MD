# Gemini Live Gateway

一个基于 Gemini Live API 的实时多模态网页和 OpenAI 兼容 API 网关。

你只需要准备一个 Gemini API Key，就可以部署一个支持文字、语音、摄像头、屏幕共享的 Gemini 多模态对话网站。项目同时提供 OpenAI 格式的 API 代理，方便接入 ChatBox、Cursor 等兼容 OpenAI API 的客户端。

## 项目来源

本项目基于 [tech-shrimp/gemini-playground](https://github.com/tech-shrimp/gemini-playground) 修改。

原项目作者：Tech Shrimp / 技术爬爬虾。原作者在 Bilibili、YouTube、抖音、公众号等平台使用同名账号。

## 优化内容

- 适配新的 Live API 路径：WebSocket 连接从 `v1alpha` 调整为 `v1beta`，匹配当前 Gemini Live API 文档。
- 修正响应类型格式：`responseModalities` 改为数组形式，例如 `["TEXT"]` 或 `["AUDIO"]`。
- 提升实时输入可靠性：音频和视频 chunk 分别按 `realtimeInput.audio` / `realtimeInput.video` 发送，避免混合 media chunk 在新版接口下不稳定。
- 增加音频结束信号：停止麦克风或断开连接时发送 `audioStreamEnd`，减少语音输入结束后模型迟迟不响应的问题。
- 文本输入走实时通道：文本消息通过 `realtimeInput.text` 发送，和新版 Live API 的实时输入结构保持一致。
- 保留 OpenAI 兼容代理：继续支持 `/v1/models`、`/v1/chat/completions`、`/v1/embeddings` 等接口路径。
- 保留双部署方式：仍然可以部署到 Deno Deploy 或 Cloudflare Workers。

## 使用方式

### 1. 准备 Gemini API Key

前往 Google AI Studio 创建 API Key：

```text
https://aistudio.google.com
```

打开网页后，在界面里填写 API Key，点击 `Connect` 即可开始使用。

### 2. Cloudflare Workers 部署(推荐)

点击按钮可以从 GitHub 仓库直接进入 Cloudflare Workers 部署流程：

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/Elon-Smith/gemini-live-gateway)

如果使用 GitHub Actions 自动部署，需要在仓库 `Settings -> Secrets and variables -> Actions` 中添加 `CF_ACCOUNT_ID` 和 `CF_API_TOKEN`；如果只在本地部署，也可以使用 `wrangler login` 登录 Cloudflare。

安装依赖后执行：

```bash
npm run deploy
```

如果你使用 Cloudflare 控制台部署，请确认 `wrangler.toml` 中的入口为：

```text
main = "src/index.js"
```

静态资源目录为：

```text
assets = { directory = "./src/static" }
```

<details>
<summary>如图</summary>

![Cloudflare Workers custom domain](/docs/images/cloudflare-domain-routing.png)

</details>

### 3. Deno Deploy 部署

1. Fork 本项目。
2. 登录或注册 Deno Deploy。
3. 创建新项目并选择你的 fork 仓库。
4. Entrypoint 填写：

```text
src/deno_index.ts
```

5. 点击 Deploy Project。
6. 部署完成后，Deno 分配的域名既可以作为网页入口，也可以作为 OpenAI 兼容 API 代理地址。

## 功能效果

网页端支持：

- 文字对话
- 麦克风语音输入
- 摄像头输入
- 屏幕共享
- Gemini Live API WebSocket 实时交互

API 代理支持 OpenAI 兼容格式。

模型列表：

```bash
curl --location 'https://your.domain.com/v1/models' \
  --header 'Authorization: Bearer YOUR-GEMINI-API-KEY'
```

聊天补全：

```bash
curl --location 'https://your.domain.com/v1/chat/completions' \
  --header 'Authorization: Bearer YOUR-GEMINI-API-KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "gemini-2.5-flash",
    "messages": [
      {
        "role": "user",
        "content": "Testing. Just say hi and nothing else."
      }
    ]
  }'
```

## 常见场景

### 我只想部署一个 Gemini 多模态网页

优先使用 Deno Deploy。只需要 fork 仓库、填写 `src/deno_index.ts` 作为入口，然后部署。

### 我想接入 ChatBox 或 Cursor

部署完成后，把 API Base URL 设置为：

```text
https://your.domain.com/v1
```

API Key 填你的 Gemini API Key。

### 当前默认 Live 模型不可用怎么办

当前前端默认模型配置在：

```text
src/static/js/config/config.js
```

如果 `models/gemini-3.1-flash-live-preview` 在你的账号或地区不可用，请改成 Google AI Studio 中可用的 Live API 模型名称。

## 项目结构

```text
gemini-live-gateway/
  src/
    index.js                  # Cloudflare Workers 入口
    deno_index.ts             # Deno Deploy 入口
    api_proxy/
      worker.mjs              # OpenAI 兼容 API 代理
    static/
      index.html              # 网页入口
      css/                    # 页面样式
      js/
        audio/                # 音频录制、播放和 worklet
        config/               # 模型和系统提示配置
        core/                 # WebSocket Live API 客户端
        tools/                # 工具调用
        video/                # 摄像头、录屏相关逻辑
  test/                       # Vitest 测试
  docs/images/                # 原项目说明图片
  wrangler.toml               # Cloudflare Workers 配置
  package.json
  LICENSE
```

## 开发命令

本地启动 Cloudflare Workers 开发服务：

```bash
npm run dev
```

运行测试：

```bash
npm test
```

部署到 Cloudflare Workers：

```bash
npm run deploy
```

使用 Deno 本地运行：

```bash
deno run --allow-net --allow-read src/deno_index.ts
```

## 已知注意事项

- Gemini Live API 仍可能调整模型名称和消息结构，请以 Google AI Studio 和官方文档为准。
- Cloudflare 节点所在地区可能影响 Gemini API 可用性；如果遇到地区限制，可以优先尝试 Deno Deploy 或绑定自定义域名。
- API Key 只应在本地浏览器或你信任的客户端中填写，不要提交到仓库。

## 致谢

感谢 [tech-shrimp/gemini-playground](https://github.com/tech-shrimp/gemini-playground) 原项目和作者 Tech Shrimp / 技术爬爬虾。

本 fork 的代码调整和 README 整理过程中使用了 Codex 辅助完成。

## License

MIT License。

本项目基于原项目修改并继续遵守原项目的 MIT License。版权声明见 [`LICENSE`](LICENSE)。
