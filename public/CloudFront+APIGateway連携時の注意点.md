# CloudFrontとAPI Gateway連携時の注意点

## はじめに

CloudFrontとAPI Gatewayを連携させる際、思わぬ問題に遭遇することがあります。
CloudFrontからAPI Gatewayへのルーティングで「Host ヘッダー」「パスの不一致」によって発生する問題と、その解決策について解説します。

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

### 具体的な不一致の例

この問題は、CloudFrontとAPI Gatewayのパス設定の不一致から発生しています。具体的な例を見てみましょう：

#### CloudFrontの設定
- オリジン: `test.execute-api.ap-northeast-1.amazonaws.com/prod`
- パスパターン: `/api/*`

#### API Gatewayの設定
- ステージ: `prod`
- リソース: `/test`（GETメソッド設定済み）

#### リクエストの流れ
1. ユーザーが `https://test.cloudfront.net/api/test` にアクセス
2. CloudFrontは `/api/*` パターンに一致し、API Gatewayにリクエストを転送
3. **重要**: このとき、CloudFrontは `https://test.execute-api.ap-northeast-1.amazonaws.com/prod/api/test` にリクエストを送信
4. しかし、API Gatewayには `/api/test` というリソースは定義されておらず、`/test` リソースのみ存在
5. 結果として「Missing Authentication Token」エラーが返される

直接 `https://test.execute-api.ap-northeast-1.amazonaws.com/prod/test` にアクセスすると正常に動作するのに、CloudFront経由では失敗する理由はここにあります。

### 解決策

この問題を解決するには2つの方法があります：

1. **API Gateway側の対応**:
   - API Gatewayに `/api/test` リソースを追加する
   - これにより、CloudFrontから転送されるパスと一致するようになる

2. **CloudFront側の対応**:
   - オリジンパスの設定を変更する（例：`/prod/api` の代わりに `/prod` を指定）
   - ビヘイビアのパスパターンを API Gateway のリソース構造に合わせて調整する（例：`/api/*` から `/test` に変更）

## まとめ

エッジ最適化API Gatewayをオリジンとして使用する場合は：
1. **Hostヘッダーを転送しない**設定にする
2. **パスの一致**を確認する
   - CloudFrontのパスパターンとAPI Gatewayのリソース構造が整合していることを確認
   - オリジンパス設定が正しいことを確認
   - ※エッジ最適化API Gatewayが直接的な原因ではない
