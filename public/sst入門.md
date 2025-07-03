---
title: AWSで学ぶSST入門
tags:
  - AWS
  - IaC
  - Pulumi
  - sst
private: false
updated_at: '2025-07-03T21:57:42+09:00'
id: 76f0836ade5b1545c224
organization_url_name: null
slide: false
ignorePublish: false
---
# SSTを使ったAWSインフラ構築

## SSTとは
SST (Serverless Stack Toolkit) は、pulumiをベースにした、サーバーレスアプリケーションを構築するためのフレームワークです。  
インフラのコード化とデプロイを容易にし、開発体験を向上させるツールセットを提供します。

## 環境セットアップ手順

### 1. サンプルプロジェクト
- [Next.js](https://sst.dev/docs/start/aws/nextjs/)
- [Remix](https://sst.dev/docs/start/aws/remix/)
```bash
# example
npx create-next-app@latest aws-nextjs
npx create-remix@latest aws-remix
```

### 2. デプロイの事前確認
```bash
pnpm sst diff --stage dev
```

### 3. 開発環境へのデプロイ
```bash
pnpm sst deploy --stage dev
```

### 4. リソースの削除
```bash
pnpm sst remove --stage dev
```

### 5. ローカル開発時のロック解除
```bash
pnpm sst runlock
```

## プロジェクト構造

以下は私が検討したSSTプロジェクトのディレクトリ構造のサンプルです。

```bash
sst-example/
├── sst.config.ts        # SST設定ファイル
├── package.json         # プロジェクトの依存関係
├── tsconfig.json        # TypeScript設定
├── stacks/              # インフラ定義のスタック
│   ├── index.ts         # スタックのエントリーポイント
│   ├── network/         # ネットワーク関連スタック
│   │   └── vpc-stack.ts # VPC定義
│   ├── database/        # データベース関連スタック
│   │   └── rds-stack.ts # RDS定義（将来用）
│   ├── storage/         # ストレージ関連スタック
│   │   └── s3-stack.ts  # S3バケット定義
│   ├── compute/         # コンピュート関連スタック
│   │   └── lambda-stack.ts # Lambda関数定義
│   └── api/             # API関連スタック
│       └── api-stack.ts # API定義
├── packages/
│   ├── core/            # 共通機能とユーティリティ
│   │   ├── package.json
│   │   └── src/
│   │       └── index.ts
│   └── functions/       # Lambda関数コード
│       ├── package.json
│       └── src/
│           └── api/
│               └── index.ts
└── config/              # 環境設定ファイル
    ├── dev.json
    └── prod.json
```

## デプロイ結果例

```
SST 3.10.0  ready!

➜  App:        sst
   Stage:      dev

~  Deploy

|  Created     default_4_16_6 pulumi:providers:random
|  Created     MyBucket sst:aws:Bucket
...中略...
|  Created     MyApi sst:aws:Function (32.6s)

↗  Permalink   https://sst.dev/u/???

✓  Complete    
   MyApi: https://???????????.lambda-url.ap-northeast-1.on.aws/
   ---
   MyBucket: sst-dev-???????-mubkbbrr
```
