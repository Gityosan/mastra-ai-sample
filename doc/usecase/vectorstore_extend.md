# VectorStoreの拡張方法

このドキュメントでは、Mastraで利用できるVectorStore（ベクトルストア）の拡張方法について解説します。

---

## 1. VectorStoreとは？

VectorStoreは、RAG（Retrieval-Augmented Generation）や類似検索のためのベクトルデータを保存・検索するための仕組みです。標準ではインメモリ型ですが、永続化や大規模運用には外部DBの利用が推奨されます。

---

## 2. 主な拡張先と選定ポイント

| ストア種別      | 特徴・用途例                         |
|-----------------|--------------------------------------|
| インメモリ型     | 小規模・一時検証用。プロセス終了で消える |
| PostgreSQL（pgvector） | 本番運用・永続化・SQL連携が必要な場合 |
| Pinecone        | クラウド型・大規模・高速・API簡単     |
| Qdrant          | オープンソース・高機能・自前運用も可能 |

---

## 3. PostgreSQL（pgvector）への拡張例

### 1. PostgreSQLサーバーの用意
- サーバーにpgvector拡張をインストール
- 公式: https://github.com/pgvector/pgvector

### 2. Mastraプロジェクトでの利用例
```js
import { PgVector } from '@mastra/pg';

const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// ベクトルの保存
await pgVector.upsert({
  indexName: 'embeddings',
  vectors: embeddings,
});

// 類似検索
const results = await pgVector.query({
  indexName: 'embeddings',
  queryVector: queryEmbedding,
  topK: 3,
});
```

---

## 4. Pineconeへの拡張例

- Pineconeアカウントを作成しAPIキーを取得
- MastraでPineconeクライアントを利用

```js
import { PineconeStore } from '@mastra/pinecone';

const pinecone = new PineconeStore({ apiKey: process.env.PINECONE_API_KEY });
await pinecone.upsert({
  indexName: 'embeddings',
  vectors: embeddings,
});
const results = await pinecone.query({
  indexName: 'embeddings',
  queryVector: queryEmbedding,
  topK: 3,
});
```

---

## 5. Qdrantへの拡張例

- Qdrantサーバーを用意（Dockerやクラウドサービス）
- MastraでQdrantクライアントを利用

```js
import { QdrantStore } from '@mastra/qdrant';

const qdrant = new QdrantStore({ url: 'http://localhost:6333' });
await qdrant.upsert({
  indexName: 'embeddings',
  vectors: embeddings,
});
const results = await qdrant.query({
  indexName: 'embeddings',
  queryVector: queryEmbedding,
  topK: 3,
});
```

---

## 6. 独自VectorStoreの実装

`@mastra/core`の`VectorStore`クラスを継承し、独自の保存先や検索ロジックを実装することも可能です。

```js
import { VectorStore } from '@mastra/core';

class MyCustomStore extends VectorStore {
  async addMany(vectors) {
    // 独自の保存処理
  }
  async similaritySearch(queryVector, options) {
    // 独自の検索処理
    return [];
  }
}
```

---

## 8. Google DriveのDocument/Slide/Spreadsheetをpgvectorに格納する方法

Google Driveからドキュメント（Google Docs）、スライド（Google Slides）、スプレッドシート（Google Sheets）の内容を抽出し、テキストをチャンク・埋め込み生成してpgvectorに保存する一連の流れを紹介します。

### 1. Google APIの準備
- Google Cloud ConsoleでAPIキー・認証情報を取得
- `googleapis`パッケージなどを利用

### 2. ドキュメントの取得例
```js
import { google } from 'googleapis';
import { MDocument } from '@mastra/rag';
import { PgVector } from '@mastra/pg';

// Google認証とAPIクライアント初期化（OAuth2など）
const auth = ...; // 省略
const drive = google.drive({ version: 'v3', auth });
const docs = google.docs({ version: 'v1', auth });
const slides = google.slides({ version: 'v1', auth });
const sheets = google.sheets({ version: 'v4', auth });

// 例: Google Docsからテキスト抽出
const docId = 'xxx';
const docRes = await docs.documents.get({ documentId: docId });
const text = docRes.data.body.content.map(block => block.paragraph?.elements?.map(e => e.textRun?.content || '').join('')).join('');

// 例: Google Slidesからテキスト抽出
const slideId = 'yyy';
const slideRes = await slides.presentations.get({ presentationId: slideId });
const slideTexts = slideRes.data.slides.map(s => s.pageElements?.map(e => e.shape?.text?.textElements?.map(te => te.textRun?.content || '').join('')).join('')).join('\n');

// 例: Google Sheetsからテキスト抽出
const sheetId = 'zzz';
const sheetRes = await sheets.spreadsheets.values.get({ spreadsheetId: sheetId, range: 'A1:Z1000' });
const sheetText = sheetRes.data.values.map(row => row.join('\t')).join('\n');

// --- 要約処理を追加 ---
// 例: OpenAI APIやMastraのLLMツールを使って要約
import { openai } from '@ai-sdk/openai';
const summaryRes = await openai.chat.completions.create({
  model: 'gpt-3.5-turbo',
  messages: [
    { role: 'system', content: '以下のテキストを日本語で簡潔に要約してください。' },
    { role: 'user', content: text }
  ]
});
const summarizedText = summaryRes.choices[0].message.content;

// --- 以降は要約済みテキストを利用 ---
const allText = [summarizedText, slideTexts, sheetText].join('\n');

// 3. チャンク分割・埋め込み生成
const doc = MDocument.fromText(allText);
const chunks = await doc.chunk({ strategy: 'recursive', size: 512, overlap: 50 });
const { embeddings } = await embedMany({
  values: chunks.map(chunk => chunk.text),
  model: openai.embedding('text-embedding-3-small'),
});

// 4. pgvectorに保存
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
await pgVector.upsert({
  indexName: 'embeddings',
  vectors: embeddings,
});
```

### ポイント
- 各GoogleサービスごとにAPIでテキスト抽出処理が異なる
- まとめて1つのコーパスとして扱う場合は`join`で連結
- チャンク・埋め込み生成はRAGの標準手順
- pgvector以外のストアにも同様の流れで保存可能

> 詳細なGoogle APIの認証・権限設定や、より大規模なデータ処理例は公式ドキュメントやGoogle APIリファレンスを参照してください。
