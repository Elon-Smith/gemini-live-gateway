# Gemini Live Gateway

[Chinese Version](README.MD)

A real-time multimodal web app and OpenAI-compatible API gateway built on the Gemini Live API.

With only a Gemini API Key, you can deploy a Gemini multimodal chat website that supports text, voice, camera input, and screen sharing. The project also provides an OpenAI-compatible API proxy for clients such as ChatBox, Cursor, and other tools that support the OpenAI API format.

## Project Source

This project is modified from [tech-shrimp/gemini-playground](https://github.com/tech-shrimp/gemini-playground).

Original author: Tech Shrimp / Jishu Papaxia. The original author uses the same name on Bilibili, YouTube, Douyin, and WeChat Official Account.

## Improvements

- Updated the Live API path: the WebSocket connection was changed from `v1alpha` to `v1beta` to match the current Gemini Live API documentation.
- Fixed the response modality format: `responseModalities` now uses an array format, such as `["TEXT"]` or `["AUDIO"]`.
- Improved real-time input reliability: audio and video chunks are sent separately through `realtimeInput.audio` and `realtimeInput.video`, avoiding unstable mixed media chunks on the newer API.
- Added an audio end signal: when the microphone is stopped or the connection is closed, the client sends `audioStreamEnd`, reducing cases where the model waits too long after voice input ends.
- Sends text through the real-time channel: text messages are sent through `realtimeInput.text`, aligning with the newer Live API input structure.
- Keeps the OpenAI-compatible proxy: `/v1/models`, `/v1/chat/completions`, `/v1/embeddings`, and related paths are still supported.
- Keeps both deployment options: the project can still be deployed to Deno Deploy or Cloudflare Workers.

## Usage

### 1. Prepare a Gemini API Key

Create an API Key in Google AI Studio:

```text
https://aistudio.google.com
```

After opening the website, enter the API Key in the UI and click `Connect`.

### 2. Cloudflare Workers Deployment (Recommended)

Click the button below to start the Cloudflare Workers deployment flow directly from the GitHub repository:

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/Elon-Smith/gemini-live-gateway)

If you use GitHub Actions for automatic deployment, add `CF_ACCOUNT_ID` and `CF_API_TOKEN` under `Settings -> Secrets and variables -> Actions` in your repository. If you deploy locally, you can also log in to Cloudflare with `wrangler login`.

After installing dependencies, run:

```bash
npm run deploy
```

If you deploy from the Cloudflare dashboard, make sure the entry point in `wrangler.toml` is:

```text
main = "src/index.js"
```

The static assets directory should be:

```text
assets = { directory = "./src/static" }
```

<details>
<summary>See image</summary>

![Cloudflare Workers custom domain](/docs/images/cloudflare-domain-routing.png)

</details>

### 3. Deno Deploy

1. Fork this project.
2. Log in to or register for Deno Deploy.
3. Create a new project and select your forked repository.
4. Set the entry point to:

```text
src/deno_index.ts
```

5. Click Deploy Project.
6. After deployment, the domain assigned by Deno can be used as both the web entry point and the OpenAI-compatible API proxy address.

## Features

The web UI supports:

- Text chat
- Microphone voice input
- Camera input
- Screen sharing
- Real-time interaction with the Gemini Live API over WebSocket

The API proxy supports the OpenAI-compatible format.

List models:

```bash
curl --location 'https://your.domain.com/v1/models' \
  --header 'Authorization: Bearer YOUR-GEMINI-API-KEY'
```

Chat completions:

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

## Common Scenarios

### I only want to deploy a Gemini multimodal website

Deno Deploy is the simplest option. Fork the repository, set `src/deno_index.ts` as the entry point, and deploy.

### I want to use it with ChatBox or Cursor

After deployment, set the API Base URL to:

```text
https://your.domain.com/v1
```

Use your Gemini API Key as the API Key.

### What if the default Live model is unavailable?

The default frontend model is configured in:

```text
src/static/js/config/config.js
```

If `models/gemini-3.1-flash-live-preview` is unavailable for your account or region, replace it with a Live API model name available in Google AI Studio.

## Project Structure

```text
gemini-live-gateway/
  src/
    index.js                  # Cloudflare Workers entry point
    deno_index.ts             # Deno Deploy entry point
    api_proxy/
      worker.mjs              # OpenAI-compatible API proxy
    static/
      index.html              # Web UI entry point
      css/                    # Page styles
      js/
        audio/                # Audio recording, playback, and worklets
        config/               # Model and system instruction configuration
        core/                 # WebSocket Live API client
        tools/                # Tool calls
        video/                # Camera and screen recording logic
  test/                       # Vitest tests
  docs/images/                # Images inherited from the original project
  wrangler.toml               # Cloudflare Workers configuration
  package.json
  LICENSE
```

## Development Commands

Start the local Cloudflare Workers development server:

```bash
npm run dev
```

Run tests:

```bash
npm test
```

Deploy to Cloudflare Workers:

```bash
npm run deploy
```

Run locally with Deno:

```bash
deno run --allow-net --allow-read src/deno_index.ts
```

## Notes

- The Gemini Live API may still change model names or message structures. Use Google AI Studio and the official documentation as the source of truth.
- The region of Cloudflare nodes may affect Gemini API availability. If you encounter regional restrictions, try Deno Deploy or bind a custom domain.
- API Keys should only be entered in your local browser or trusted clients. Do not commit API Keys to the repository.

## Acknowledgments

Thanks to the original [tech-shrimp/gemini-playground](https://github.com/tech-shrimp/gemini-playground) project and its author, Tech Shrimp / Jishu Papaxia.

The code adjustments and README organization for this fork were assisted by Codex.

## License

MIT License.

This project is modified from the original project and continues to comply with the original MIT License. See [`LICENSE`](LICENSE) for the copyright notice.
