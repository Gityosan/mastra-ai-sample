# Cloudflare WorkersでMCPサーバーをデプロイする方法

このガイドでは、Cloudflare Workersを使ってMCPサーバーをデプロイする手順をまとめます。

---

## 1. 必要なもの
- Cloudflareアカウント（[公式サイトで無料登録](https://dash.cloudflare.com/sign-up)）
- Node.js（v16以上推奨）
- npm
- テキストエディタ（VSCodeなど）

---

## 2. プロジェクト作成
```bash
# プロジェクトフォルダの作成
mkdir my-mcp-server
cd my-mcp-server

# Cloudflare Workerプロジェクトの初期化
npx create-cloudflare@latest
```
プロンプト例：
- プロジェクト名：mcp-demo
- テンプレート：Hello World
- 言語：TypeScript
- Git：No（任意）
- デプロイ：No（後で実施）

---

## 3. MCPサーバーのセットアップ
```bash
cd mcp-demo
npx workers-mcp setup
```
プロンプトはEnterでOK。`src/index.ts`が自動更新されます。

---

## 4. サンプル機能の追加（例）
`src/index.ts`に以下を追加：
```ts
import { createMCPServer } from "@modelcontextprotocol/server";

export default {
  async fetch(request, env, ctx) {
    const server = createMCPServer({
      tools: [
        {
          name: "greet",
          description: "あいさつを返します",
          parameters: {
            type: "object",
            properties: { name: { type: "string", description: "名前" } },
            required: ["name"],
          },
          handler: async ({ name }) => `こんにちは、${name}さん！`,
        },
      ],
    });
    return server.fetch(request);
  },
};
```

---

## 5. ローカルでテスト
```bash
npx wrangler dev
```
別ターミナルでMCP Inspectorも利用可：
```bash
npx @modelcontextprotocol/inspector@latest
```

---

## 6. Cloudflareにデプロイ
```bash
npx wrangler login # 初回のみ
npx wrangler deploy
```
デプロイ成功時に `https://mcp-demo.your-username.workers.dev` のようなURLが発行されます。

---

## 7. 注意点
- Cloudflare無料枠にはリクエスト数制限あり
- CORSや日本語文字化けなどは公式ドキュメントやQiita記事参照
- 公開用エンドポイントは認証などセキュリティ対策推奨

---

## 参考
- [Qiita記事](https://qiita.com/Nakamura-Kaito/items/1e6aabfa52911ab0ac5e)
- [aws-mcp公式](https://github.com/awslabs/mcp)
