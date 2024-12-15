# aws-fargate-express
# AWS FargateでExpress APIをデプロイする手順書

## 1. ネットワーク環境の構築

### VPCの作成
1. VPCダッシュボードで「VPCの作成」を選択
2. 基本設定:
   - VPC名: `aws-fargate-express-01-vpc`
   - 構成: パブリックサブネット×1
   - CIDR: `10.0.0.0/16`（デフォルト）
   - AZ数: 1
   - パブリックサブネット設定:
     - 名前: `aws-fargate-express-01--subnet-public-1a`
     - CIDR: `10.0.0.0/24`
     - ルートテーブル名: `aws-fargate-express-01--rtb-public`
     - インターネットゲートウェイ名: `aws-fargate-express-01-igw`

### セキュリティグループの設定
1. EC2ダッシュボードで「セキュリティグループの作成」
2. 基本設定:
   - 名前: `aws-fargate-express-01-sg-web`
3. インバウンドルール:
   - HTTP (TCP/80): `0.0.0.0/0`
   - アプリケーションポート (TCP/3000): `0.0.0.0/0`

## 2. アプリケーションの準備

### GitHubリポジトリの作成
- リポジトリ名: `aws-fargate-express`
- 以下のファイルを作成:

#### Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

#### app.js
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello World from Express!' });
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### ローカルでの動作確認
```bash
node app.js
# http://localhost:3000 で動作確認
```

## 3. ECSの設定

### タスク定義の作成
1. ECSコンソールで「タスク定義」→「新規作成」
2. 基本設定:
   - ファミリー名: `aws-fargate-express-01-task`
   - コンテナ設定:
     - 名前: `fargate-express`
     - イメージURI: [ECRのイメージURI]
     - ポート: 3000
   - リソース:
     - CPU: 0.25 vCPU
     - メモリ: 0.5 GB
     - OS: Linux

### クラスターの作成
1. 「クラスターの作成」を選択
2. 設定:
   - 名前: `aws-fargate-express-01-cluster`
   - VPC: 作成済みのVPCを選択
   - サブネット: 作成済みのパブリックサブネット

### サービスのデプロイ
1. クラスター内で「デプロイ」を選択
2. 基本設定:
   - サービス名: `aws-fargate-express-01-service`
   - 起動タイプ: Fargate
   - プラットフォーム: 最新版
   - タスク数: 1
3. ネットワーク:
   - VPC/サブネット: 作成済みのものを選択
   - セキュリティグループ: 作成済みのものを選択
   - パブリックIP: 自動割り当て有効

## 4. デプロイ後の確認

1. タスクの状態が「RUNNING」になるまで待機
2. パブリックIPアドレスを確認
3. `http://<パブリックIP>:3000` でアクセス
4. `{"message":"Hello World from Express!"}` が表示されることを確認
