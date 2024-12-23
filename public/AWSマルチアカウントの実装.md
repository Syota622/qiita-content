---
title: モダンなAWSマルチアカウントの実装
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
- SecurityHub、Inspector、GuardDuty、IAM Access Analyzerなどのセキュリティサービスの集約

### 2.2.2. ログアカウント（Log Account）
- Config、CloudTrailログの集約
- セキュリティログの保管
- ログ分析環境の提供

## 2.3. Member OU
各環境のワークロードを管理するOUです。
アプリケーションやWebサービスの種類ごとに、独立したアカウントを作成することをお勧めします：

### 2.3.1. アカウント分割の考え方
1. **環境による分割**
- 開発環境（dev）：開発者が自由に検証できる環境
- ステージング環境（stg）：本番相当の検証環境
- 本番環境（prod）：実サービスの運用環境

2. **サービスによる分割**
- アプリケーションごとにアカウントを分離
- マイクロサービスごとの分離
- プロジェクトやチームごとの分離

このような分割により：
- サービスごとの独立した開発が可能
- 環境ごとの適切なセキュリティ設定
- コストの明確な分離
- 障害影響範囲の局所化

### 2.3.2. 開発環境（dev-アプリ）
- 開発用のリソース管理
- テスト環境の提供
- 開発者向けの柔軟な権限設定

### 2.3.3. ステージング環境（stg-アプリ）
- 本番相当の検証環境
- 結合テスト環境
- リリース前の最終確認環境

### 2.3.4. 本番環境（prod-アプリ）
- 実サービスの運用環境
- 厳格なセキュリティ設定
- 本番データの管理

# 3. CI/CD構成

## 3.1. GitHub連携
- アプリケーションコードの管理
- インフラコードの管理
- GitFlowベースのブランチ戦略

## 3.2. ブランチ戦略とデプロイフロー

### 開発環境（dev）
- feature/XXXブランチで開発
- developブランチへのマージで自動デプロイ
- 単体テスト、lint チェックの自動実行

### ステージング環境（stg）
- developブランチからstagingブランチへのマージで自動デプロイ
- 結合テスト、E2Eテストの自動実行
- QA・動作確認用の環境として利用

### 本番環境（prod）
- stagingブランチからmainブランチへのマージで自動デプロイ
- 本番リリース用のタグ付け
- リリースノートの自動生成

### 共通基盤環境（Management/Security OU）
- management、audit、logアカウントはdev/stg/prodの環境分離なし
- mainブランチへのマージで直接デプロイ(GitHub Flow)
- 変更時は事前承認プロセスを実施
- デプロイ前の自動テスト実施必須

## 3.3. GitHub Actions
- 環境変数は各環境のSecretsで管理
- デプロイ時のSlack通知連携
- 各種テストの自動実行
- インフラのProvisioning自動化
- フロント・バックエンドの自動デプロイ

## 3.4. 各種フレームワーク対応
- Terraform：インフラのコード化
- Rails/Go/Nuxt.js等のアプリケーションデプロイ
- コンテナイメージのビルドと管理

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

# 7. シングルアカウントからの移行

## 7.1. 移行の基本方針
- 既存のシングルアカウントを本番環境（prod）として位置づけ
- シングルアカウント内の開発・ステージング環境は段階的に削除
- 新規の開発・ステージング環境は新アカウントで構築

## 7.2. 移行手順
1. Control Towerのセットアップ
2. 既存アカウントのOrganizationsへの招待
3. Security OUの構築（audit, log）
4. Member OUの構築（dev, stg）
5. IAM Identity Centerの設定
6. 既存環境からの段階的移行

## 7.3. 移行時の注意点
- 既存リソースの依存関係の確認
- 移行期間中の一時的な権限設定
- デプロイパイプラインの更新
- 監視・アラートの再設定

# 8. さいごに

この構成により、開発からセキュリティまでをカバーした、実践的なマルチアカウント環境を実現できます。Control Tower、Organizations、IAM Identity Centerの組み合わせにより、効率的かつセキュアな運用が可能になります。
