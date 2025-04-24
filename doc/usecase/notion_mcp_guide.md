# Notion MCP連携ガイド

このガイドでは、MastraからNotionのデータにアクセス・操作するためにMCP連携を行う手順を解説します。

---

## 1. ユースケース概要
- NotionのページやデータベースをMastraエージェントやワークフローから扱いたい場合にMCP経由で連携します。

---

## 2. 前提条件
- NotionアカウントおよびAPIトークン取得済み
- Mastraプロジェクトがセットアップ済み

---

## 3. MCPサーバー（Notion用）の用意
- Notion連携用のMCPサーバーを用意します（例: mastra-notion-mcp など）
- サーバーの設定でNotion APIトークンを指定

---

## 4. MastraMCPClientの設定例
```js
import { MastraMCPClient } from '@mastra/core';

const notionClient = new MastraMCPClient({
  name: 'notion-mcp-client',
  server: {
    url: 'https://your-notion-mcp.example.com',
    // 認証情報など必要に応じて
  },
});

// 例: Notionページ一覧取得
const result = await notionClient.callTool({
  tool: 'listPages',
  input: {},
});
console.log(result);
```

---

## 5. 注意点
- Notion APIのスコープや共有設定に注意
- セキュリティのためAPIトークンは安全に管理
- MCPサーバーの仕様や利用可能ツールは公式ドキュメント参照

---
