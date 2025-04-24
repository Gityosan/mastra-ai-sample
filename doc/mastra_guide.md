# Mastraの使い方ガイド

MastraはAIエージェントやワークフローを簡単に構築・運用できる強力なフレームワークです。ここでは、Mastraの基本的な使い方について日本語で解説します。

## 1. インストール

プロジェクトディレクトリで以下のコマンドを実行してMastraをインストールします。

```bash
npm install @mastra/core
```

または、pnpmを使用する場合:

```bash
pnpm add @mastra/core
```

## 2. 基本的なセットアップ

エージェントやワークフローを作成するには、必要なモジュールをインポートします。

```js
import { Agent } from '@mastra/core';
```

## 3. Mastraプロジェクトのフォルダ構成

Mastraプロジェクトの一般的なディレクトリ構成例は以下の通りです。

```
mastra-project/
├── src/                # エージェントやワークフローの実装コード
│   ├── agents/         # エージェント関連のファイル
│   ├── workflows/      # ワークフロー関連のファイル
│   └── tools/          # ツールやユーティリティ
├── doc/                # ドキュメント（このガイドなど）
├── package.json        # プロジェクト設定・依存関係
├── README.md           # プロジェクト概要
└── ...
```

- `src/` ディレクトリには、エージェントやワークフロー、独自ツールの実装をまとめます。
- `doc/` にはガイドや設計資料などのドキュメントを格納します。
- `package.json` で依存ライブラリやスクリプトを管理します。

プロジェクトの規模や目的に応じて、適宜サブディレクトリやファイルを追加してください。

## 4. 主な機能一覧と使い方

Mastraには以下のような主な機能があります。

### 4.1 エージェントの作成・管理
- 複数のAIエージェントを簡単に作成し、管理できます。
- 各エージェントには役割やプロンプト、利用するツールなどを柔軟に設定可能です。

#### 使い方例
```js
import { Agent } from '@mastra/core';

const agent = new Agent({
  name: 'タスク管理エージェント',
  description: 'タスクを管理するAIエージェント',
  // 追加設定
});

agent.run('今日のタスクを教えて');
```

---

### 4.2 ワークフローの構築
- 複数のエージェントやツールを組み合わせてワークフロー（自動化フロー）を作成できます。
- ステップごとに処理を定義し、複雑な自動化も直感的に設計可能です。

#### 使い方例
```js
import { Workflow } from '@mastra/core';

const workflow = new Workflow({
  steps: [
    // ステップを定義
  ]
});

workflow.run({ input: 'データを処理してください' });
```

---

### 4.3 RAG（Retrieval-Augmented Generation）
- 外部データや大量のドキュメントから必要な情報を検索し、AIの回答に活用できます。
- チャットボットやFAQ、自動レポート生成など、知識ベース型AIに最適です。

#### 主な機能
- ドキュメントのチャンク分割と埋め込み生成（ベクトル化）
- 高速な類似検索による情報取得
- OpenAIやCohereなど主要な埋め込みプロバイダーに対応

#### 使い方例
```js
import { MDocument } from '@mastra/core';

// 1. ドキュメントを読み込み
const doc = MDocument.fromText('大量のテキストデータ...');

// 2. チャンク分割
const chunks = await doc.chunk({
  strategy: 'recursive', // 他にcharacter, token, markdown等も選択可
  size: 512,
  overlap: 50,
});

// 3. 埋め込み生成（OpenAIなどのプロバイダーを利用）
// 詳細は公式ドキュメント参照

// 4. ベクトルDBやRAGワークフローと連携して検索・回答生成
```

> 詳細な設定やプロバイダーごとの使い方は[公式RAGドキュメント](https://docs.mastra.ai/rag/overview)を参照してください。

---

### 4.4 外部ツール・API連携
- 外部APIや独自ツールと連携し、機能を拡張できます。
- 例えば、Web検索やデータベース操作なども可能です。

#### 使い方例
```js
import { Tool } from '@mastra/core';

const searchTool = new Tool({
  name: 'Web検索',
  execute: async (query) => {
    // Web検索処理
    return '検索結果';
  }
});
```

---

### 4.5 MCP（Model Context Protocol）連携
- MCPは、AIエージェントやツール、外部サービスを標準化されたプロトコルで連携するための仕組みです。
- MastraではMCPクライアントを利用することで、さまざまなMCPサーバーやツールと簡単に接続・連携できます。

#### 通信方式の違い（stdioとSSE）
- **stdioベース**: 標準入出力（コマンドライン経由）でサーバーと通信します。ローカル開発やCLIツールとの連携に適しています。
- **SSEベース**: サーバー・クライアント間でHTTPのServer-Sent Events（SSE）を利用してリアルタイム通信します。Webサービスや外部API連携に便利です。

#### 主な用途
- 他のAIエージェントや外部ツールとの連携
- 社内外のMCPサーバー経由でのリソース・機能の統合利用

#### 使い方例
```js
import { MastraMCPClient } from '@mastra/core';

const client = new MastraMCPClient({
  name: 'my-mcp-client',
  server: {
    url: 'https://your-mcp-server.example.com',
    // 必要に応じて認証や追加設定
  },
});

// MCPサーバー上のツールを呼び出し
const result = await client.callTool({
  tool: 'search',
  input: { query: 'AIとは' },
});

console.log(result);
```

> 詳細な設定や高度な使い方は[公式MCPドキュメント](https://docs.mastra.ai/reference/tools/mcp-configuration)を参照してください。

---
