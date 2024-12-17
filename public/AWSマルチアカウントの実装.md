---
title: AWSマルチアカウントの実装
tags:
  - AWS
  - IAM Identity Center
  - Organizations
  - Control Tower
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# 1. はじめに

AWSのマルチアカウント構成について、Control Tower、Organizations、IAM Identity Centerを使用した実装方法をまとめてみました。

以下の構成図は、GitHubとの連携やSlack通知を含めた、実践的なマルチアカウント環境を示しています。

制限のない自社開発企業やスタートアップ企業には是非とも導入して欲しい構成です。

![aws_multi_account.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/7b3a356d-63b5-0773-7841-7ef1662436ab.png)

# 2. アカウント構成と役割

## 2.1. 管理アカウント（Management Account）
管理アカウントには以下の主要サービスを配置します：
- Control Tower：マルチアカウントの管理
- Organizations：組織単位（OU）の管理
- IAM Identity Center：シングルサインオンの提供

## 2.2. Security OU
セキュリティ関連のアカウントを管理するOUです：

### 2.2.1. 監査アカウント（Audit Account）
- セキュリティ監査の実施
- コンプライアンスの確認
- AWS Configルールの管理

### 2.2.2. ログアカウント（Log Account）
- CloudTrailログの集約
- セキュリティログの保管
- ログ分析環境の提供

## 2.3. Member OU
各環境のワークロードを管理するOUです：

### 2.3.1. 開発環境（dev-アプリ）
- 開発用のリソース管理
- テスト環境の提供
- 開発者向けの柔軟な権限設定

### 2.3.2. ステージング環境（stg-アプリ）
- 本番相当の検証環境
- 結合テスト環境
- リリース前の最終確認環境

### 2.3.3. 本番環境（prod-アプリ）
- 実サービスの運用環境
- 厳格なセキュリティ設定
- 本番データの管理

# 3. CI/CD構成

## 3.1. GitHub連携
- アプリケーションコードの管理
- インフラコードの管理
- ブランチ戦略の統制

## 3.2. GitHub Actions
- CI/CDパイプラインの実行
- 各環境へのデプロイ自動化
- テストの自動実行

## 3.3. 各種フレームワーク対応
- Terraform：インフラのコード化
- Rails/Go/Nuxt.js等のアプリケーションデプロイ

# 4. 通知・モニタリング

## 4.1. Slack連携
- デプロイ結果の通知
- アラート通知
- 承認ワークフローの連携

# 5. IAM Identity Center構成

## 5.1. 権限管理
- AWS Developer権限の付与
- 環境別のアクセス制御
- MFAの強制適用

## 5.2. SSO設定
- GitHubとの連携
- 環境別のロール設定
- アクセス監視

# 6. セキュリティ設定

## 6.1. ガードレール
- Control Towerの予防的ガードレール
- Control Towerの検出的ガードレール
- 各OUごとのSCP設定

## 6.2. アカウント間の連携
- CloudTrailログの集約設定
- AWS Config集約設定
- SecurityHubの統合

# 7. さいごに

この構成により、開発からセキュリティまでをカバーした、実践的なマルチアカウント環境を実現できます。Control Tower、Organizations、IAM Identity Centerの組み合わせにより、効率的かつセキュアな運用が可能になります。