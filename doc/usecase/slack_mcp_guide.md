# Slack MCP連携ガイド

このガイドでは、MastraからSlackのチャンネルやメッセージへアクセス・操作するためにMCP連携を行う手順を解説します。

---

## 1. ユースケース概要
- Slackのチャンネル情報取得やメッセージ送信などをMastraエージェントやワークフローから実行する場合にMCP経由で連携します。

---

## 2. 前提条件
- SlackワークスペースとBotトークン取得済み
- Mastraプロジェクトがセットアップ済み

---

## 3. MCPサーバー（Slack用）の用意
- Slack連携用のMCPサーバーを用意します（例: mastra-slack-mcp など）
- サーバーの設定でSlack Botトークンを指定

---

## 4. MastraMCPClientの設定例
```js
import { MastraMCPClient } from '@mastra/core';

const slackClient = new MastraMCPClient({
  name: 'slack-mcp-client',
  server: {
    url: 'https://your-slack-mcp.example.com',
    // 認証情報など必要に応じて
  },
});

// 例: チャンネル一覧取得
const result = await slackClient.callTool({
  tool: 'listChannels',
  input: {},
});
console.log(result);
```

---

## 5. 注意点
- Botの権限スコープやワークスペース設定に注意
- セキュリティのためBotトークンは安全に管理
- MCPサーバーの仕様や利用可能ツールは公式ドキュメント参照

---
