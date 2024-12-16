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
# はじめに

AWSのマルチアカウント構成について、Control Tower、Organizations、IAM Identity Centerを使用した実装方法をまとめてみました。
セキュリティとガバナンスを考慮した6つのアカウント構成について、それぞれの役割と設定方法を解説します。

# AWSアカウントの役割と構成

## 管理アカウント（management）
- Control Towerの管理
- IAM Identity Centerの設定
- AWS Organizationsの管理
- 請求の一元管理

### 主な設定項目
- Control Towerのランディングゾーン設定
- SSO（IAM Identity Center）の設定
- 組織単位（OU）の設定
- 一括請求（Consolidated Billing）の有効化

## ログアカウント（log）
- 全アカウントのCloudTrailログの集約
- セキュリティログの保管
- コンプライアンス監査用ログの保存

### 主な設定項目
- CloudTrailの集約設定
- S3バケットのライフサイクル設定
- CloudWatch Logsの集約設定
- Athenaによるログ分析環境

## 監査アカウント（audit）
- セキュリティ監査の実施
- コンプライアンス状況の確認
- AWS Configルールの管理
- SecurityHubの集中管理

### 主な設定項目
- AWS Configアグリゲーターの設定
- SecurityHubの集約設定
- GuardDutyの管理
- コンプライアンスレポートの生成

## 開発環境アカウント（dev）
- 開発環境のリソース管理
- テスト用環境の提供
- 開発者向けの権限管理

### 主な設定項目
- 開発者用のIAMロール設定
- 予算アラートの設定
- タグ付けポリシーの適用
- 開発環境固有のセキュリティ設定

## ステージング環境アカウント（stg）
- 本番環境と同様の構成
- 統合テスト環境の提供
- リリース前の最終確認

### 主な設定項目
- 本番環境に近い権限設定
- テスト用データの管理
- パフォーマンステスト環境の構築
- ステージング用のセキュリティ設定

## 本番環境アカウント（prod）
- 実サービスの運用
- 最も厳格なセキュリティ設定
- 本番データの管理

### 主な設定項目
- 厳格なアクセス制御
- バックアップポリシーの設定
- 高可用性の確保
- 本番用のセキュリティ設定

# 実装のポイント

## Control Towerの設定
```text
1. ランディングゾーンの作成
2. アカウント作成の自動化設定
3. ガードレールの設定
4. コンプライアンスパッケージの適用
```

## Organizations設定
```text
1. 組織単位（OU）の階層設定
2. サービスコントロールポリシー（SCP）の適用
3. タグポリシーの設定
4. バックアップポリシーの設定
```

## IAM Identity Center設定
```text
1. 権限セットの作成
2. グループの作成と権限割り当て
3. ユーザーの作成とグループ割り当て
4. MFAポリシーの設定
```

# アカウント間の連携設定

## CloudTrailの集約
```text
1. ログアカウントにメインのTrailを作成
2. 他アカウントからのログ転送設定
3. ログ保持期間の設定
4. アクセス権限の設定
```

## AWS Configの集約
```text
1. 監査アカウントにアグリゲーターを設定
2. 各アカウントのConfig Recorder設定
3. ルールの一元管理設定
4. レポート出力の設定
```

# セキュリティ設定

## アカウント横断的な設定
- パスワードポリシー
- MFA強制
- ルートユーザーの保護
- アクセスキーの管理

## 環境別のセキュリティ設定
- 本番環境：最も厳格な設定
- ステージング環境：本番に近い設定
- 開発環境：柔軟な設定

# さいごに

マルチアカウント構成は、初期設定は複雑ですが、セキュリティとガバナンスの観点で大きなメリットがあります。Control Tower、Organizations、IAM Identity Centerを組み合わせることで、効率的な管理が可能になります。
