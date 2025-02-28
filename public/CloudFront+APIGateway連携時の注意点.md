---
title: CloudFront+APIGateway連携時の注意点
tags:
  - AWS
  - CloudFront
  - APIGateway
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# CloudFrontとAPI Gateway連携時の注意点

## はじめに

CloudFrontとAPI Gatewayを連携させる際、思わぬ問題に遭遇することがあります。CloudFrontからAPI Gatewayへのルーティングで「Host ヘッダー」によって発生する問題と、その解決策について解説します。

## 問題の現象

CloudFrontで以下のような設定をした場合：
- defaultBehavior: S3 Origin
- additionalBehaviors: '/api/*': API Gateway Origin

次のような症状が発生しました：
- `/api/*` パターンに一致するリクエストが、API Gatewayではなく**S3にルーティング**される
- エラーメッセージ: `404 Not Found, Code: NoSuchKey, Key: prod/api/test`

## 原因

この問題の核心は**API Gatewayのエッジ最適化エンドポイント**と**Hostヘッダーの扱い**にありました。

### 発生メカニズム

1. CloudFrontで「全てのビューワーヘッダー」設定を使用すると、**Hostヘッダー**も転送される
2. エッジ最適化API Gateway（例：`test.execute-api.ap-northeast-1.amazonaws.com`）は**内部的にCloudFrontを使用**している
3. HostヘッダーにCloudFrontのドメイン（例：`test.cloudfront.net`）が含まれると、API Gatewayの内部CloudFrontが**元のCloudFrontを呼び出してしまう**
4. 結果的にリクエストが**ループ**し、最終的にデフォルトのS3オリジンにルーティングされる

## 解決策

オリジンリクエストポリシーのヘッダー設定を「**全てのビューワーヘッダー**」から「**なし**」に変更することで解決します。

これにより、API Gatewayへのリクエスト時にHostヘッダーが転送されなくなり、適切にルーティングされるようになります。

## もう一つの問題：パスの不一致

問題を解決した後、今度は別のエラーが発生することがあります：
- エラーメッセージ: `{"message":"Missing Authentication Token"}`

これは以下の理由で発生します：

1. CloudFrontのパスパターン `/api/*` とオリジンパス設定のミスマッチ
2. API Gatewayで定義されていないパスへのリクエスト

### 解決策

以下のいずれかの方法で対応します：
1. **API Gateway側**: `/api/test` リソースを追加する
2. **CloudFront側**: オリジンパスやビヘイビア設定を調整する

## まとめ

エッジ最適化API Gatewayをオリジンとして使用する場合は：
1. **Hostヘッダーを転送しない**設定にする
2. **パスの一致**を確認する

これらの点に注意することで、CloudFrontとAPI Gatewayの連携における問題を回避できます。