# DynamoDB + Lambda + APIGateway プロジェクトにMastraエンドポイント・AWS MCPサーバーを活用してDBデバッグするガイド

このガイドでは、既存のAWS DynamoDB + Lambda + APIGateway構成のプロジェクトに、MastraからDBデバッグ用エンドポイントを追加する方法、さらにAWS MCPサーバーを活用してよりセキュア・柔軟にDynamoDB/Lambdaデバッグを行う方法を解説します。

---

## 1. ユースケース概要
- 既存のサーバーレス構成（DynamoDB + Lambda + APIGateway）に、Mastra経由でDynamoDBへアクセスできるデバッグ用エンドポイントを追加します。
- さらにAWS MCPサーバーを活用することで、Lambda/DB/他AWSサービスのデバッグをより安全・柔軟に実現できます。
- MastraエージェントやツールからDynamoDBやLambdaの内容を参照・操作できるようにし、開発・運用時のDBデバッグを効率化します。

---

## 2. 前提条件
- 既存のAWSインフラ（DynamoDB, Lambda, APIGateway）がセットアップ済み
- Node.js環境でMastraが利用できる
- AWS SDKの権限設定済み

---

## 3. AWS MCPサーバーを活用したデバッグ（推奨）

### 1. MastraからAWS MCPサーバーへ接続する設定例
```js
import { MastraMCPClient } from '@mastra/core';

const awsClient = new MastraMCPClient({
  name: 'aws-mcp-client',
  server: {
    url: 'https://your-aws-mcp.example.com', // AWS MCPサーバーのURL
    // 認証情報が必要な場合はここで設定
  },
});

// 例: DynamoDBテーブルの内容取得
type KeyType = { id: string };
const result = await awsClient.callTool({
  tool: 'dynamodbGetItem',
  input: { tableName: 'YourTable', key: { id: 'xxx' } },
});
console.log(result);
```

### 2. MCPConfigurationを使った複数サーバー管理例
```js
import { MCPConfiguration } from '@mastra/core';

const mcp = new MCPConfiguration({
  servers: {
    aws: {
      url: 'https://your-aws-mcp.example.com',
      // 認証情報など
    },
  },
});

const toolsets = await mcp.getToolsets();
// agentやworkflowでtoolsetsを指定して利用
```

### 3. エージェント/ワークフローでAWS MCPツールを使う
- エージェントやワークフローの`tools`に`aws`サーバーのDynamoDB/Lambdaツールを追加し、自然言語でDB参照・デバッグが可能

---

## 4. Lambda関数にデバッグ用ハンドラを追加（補足: 既存手法）
1. 既存のLambdaプロジェクトの`src/handlers/`等に、Mastra用のエンドポイントハンドラ（例：`mastraDebugHandler.js`）を追加します。
2. DynamoDBへクエリ・スキャンできる関数を実装します。

```js
// src/handlers/mastraDebugHandler.js
const AWS = require('aws-sdk');
const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
  const { table, key } = JSON.parse(event.body);
  // 例: キー指定で取得
  const result = await dynamo.get({ TableName: table, Key: key }).promise();
  return {
    statusCode: 200,
    body: JSON.stringify(result.Item),
  };
};
```

---

## 4. APIGatewayで新規エンドポイントを追加
- 例: `/mastra-debug` というパスでLambda関数を呼び出すようAPIGatewayの設定を追加
- POSTメソッドでテーブル名・キー等を受け取る

---

## 5. Mastraからエンドポイントを呼び出すツールを作成
1. Mastraプロジェクトの`src/tools/`にHTTPリクエスト用ツールを実装

```js
// src/tools/dynamodbDebugTool.js
import { Tool } from '@mastra/core';
import fetch from 'node-fetch';

export const dynamodbDebugTool = new Tool({
  name: 'DynamoDBデバッグ',
  execute: async ({ table, key }) => {
    const res = await fetch('https://<your-api-endpoint>/mastra-debug', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ table, key }),
    });
    return await res.json();
  },
});
```

---

## 6. Mastraエージェントやワークフローからデバッグツールを利用
- エージェントやワークフローの`tools`に`dynamodbDebugTool`を追加し、自然言語でDB参照・デバッグが可能

---

## 7. 注意事項
- AWS MCPサーバー側の認証・アクセス制御を必ず行ってください
- 本番環境ではデバッグ用ツールの有効化範囲やアクセス権限に注意してください
- デバッグ用途のエンドポイントやツールは不要になったら削除・無効化することを推奨します

---

以上で、既存のAWSサーバーレス構成にMastraからDynamoDBデバッグ用エンドポイントを追加する方法、及びAWS MCPサーバーを活用した最新のデバッグ手法のガイドは完了です。
