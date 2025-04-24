# Google Drive MCP連携ガイド

このガイドでは、MastraからGoogle Driveのファイルやフォルダにアクセス・操作するためにMCP連携を行う手順を解説します。

---

## 1. ユースケース概要
- Google Drive内のドキュメントやファイルをMastraエージェントやワークフローから検索・取得・操作したい場合にMCP経由で連携します。

---

## 2. 前提条件
- GoogleアカウントとGoogle Drive APIの認証情報（OAuth2クライアントID等）取得済み
- Mastraプロジェクトがセットアップ済み

---

## 3. MCPサーバー（Google Drive用）の用意
- Google Drive連携用のMCPサーバーを用意します（例: mastra-gdrive-mcp など）
- サーバーの設定でGoogle認証情報を指定

---

## 4. MastraMCPClientの設定例
```js
import { MastraMCPClient } from '@mastra/core';

const gdriveClient = new MastraMCPClient({
  name: 'gdrive-mcp-client',
  server: {
    url: 'https://your-gdrive-mcp.example.com',
    // 認証情報など必要に応じて
  },
});

// 例: ファイル一覧取得
const result = await gdriveClient.callTool({
  tool: 'listFiles',
  input: {},
});
console.log(result);
```

---

## 5. 注意点
- Google APIのスコープや認可設定に注意
- 認証情報は安全に管理
- MCPサーバーの仕様や利用可能ツールは公式ドキュメント参照

---
