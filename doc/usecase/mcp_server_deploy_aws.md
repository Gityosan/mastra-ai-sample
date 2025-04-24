# AWSでMCPサーバーをデプロイする方法

このガイドでは、AWS上（Lambda + API Gateway, ECS, など）でMCPサーバーをデプロイする代表的な方法をまとめます。

---

## 1. 必要なもの
- AWSアカウント
- AWS CLIセットアップ済み
- Node.js（v16以上推奨）
- Git

---

## 2. MCPサーバーの取得
```bash
git clone https://github.com/awslabs/mcp.git
cd mcp
```

---

## 3. Lambda + API Gatewayでデプロイ（例）
### a. Lambda用プロジェクトの準備
`src/aws-lambda-mcp-server` ディレクトリを参考にLambda用のMCPサーバーを作成。

### b. 必要な依存パッケージをインストール
```bash
cd src/aws-lambda-mcp-server
npm install
```

### c. Lambda関数としてデプロイ
SAMやServerless Frameworkを使うのが便利です。例：
```bash
# AWS SAM CLIでデプロイ
sam build
sam deploy --guided
```
- デプロイ時にAPI Gatewayのエンドポイントが発行されます。

---

## 4. ECS/Fargateでデプロイ（例）
### a. Dockerイメージをビルド
```bash
docker build -t mcp-server .
```
### b. ECRへプッシュ & ECS/Fargateでサービス作成
- ECRリポジトリを作成し、イメージをプッシュ
- ECSタスク定義を作成し、Fargateでサービスを起動
- ALBやAPI GatewayでHTTPSエンドポイントを公開

---

## 5. デプロイ後の確認
- API GatewayやALBのエンドポイントにcurlやMCP Inspectorでアクセスし、ツール一覧取得などができれば成功

---

## 6. 注意点
- LambdaやECS/FargateのIAM権限設定に注意
- 公開APIの場合は認証・セキュリティ対策必須
- コールドスタートやタイムアウト設定にも注意

---

## 参考
- [aws-mcp公式](https://github.com/awslabs/mcp)
- [AWS Lambda公式ドキュメント](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli.html)
