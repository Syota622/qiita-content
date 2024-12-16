---
title: 個人的にモダンだと思うAWSのWebコンテナ基盤
tags:
  - AWS
  - スタートアップ
  - Web
  - コンテナ
  - DevOps
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

スタートアップ企業でAWSを使った開発に3~4年ほど携わってきました。スタートアップならではの特徴として、開発の生産性を重視し、よりモダンな構成が選ばれます

これまでの経験から、特に効果的だった構成や選択についてまとめてみました。

# インフラ構成のポイント

## IaC
インフラはTerraformで管理するのがベスト。コードベースで環境を管理でき、再現性も高いです。

## コンテナ基盤
フロントエンド・バックエンドともにECS Fargateを採用。サーバー管理が不要で、コンテナに集中できます。

## API構成
シンプルにバックエンドのECSだけで十分です。API GatewayやAppSyncは、必要に迫られるまで導入を見送っても問題ありません。

## CI/CD
GitHub Actionsが使いやすいです。AWSのCodeシリーズは機能も十分ですが、GitHubと統合された環境にすることで開発効率が上がります。

## データベース
Aurora Serverlessを採用。
スケールアップなどの運用がほぼほぼ発生せず、非常に管理しやすいです。

## AWSアカウント構成
Control Towerを使って以下の6つのアカウントを作成：
- 開発環境（dev）
- ステージング環境（stg）
- 本番環境（prod）
- 監査用（audit）
- ログ用（log）
- 管理用（management）

# AWS開発者に求められるスキル

- Dockerfileが書けること
- フロントエンド・バックエンドの基本的な実装ができること（ポートフォリオレベルで可）
- 各言語やフレームワークの基本的な使い方

# 実装時の選択ポイント

## コンテナイメージの管理
ECRでは`latest`タグは使わず、GitHub Actionsの`${{ github.sha }}`を使ってバージョン管理するのがおすすめです。

https://docs.github.com/ja/actions/use-cases-and-examples/deploying/deploying-to-amazon-elastic-container-service


## 監視設計
まずはCloudWatchからスタート：
- CloudWatch Alarm
- CloudWatch Dashboard
- Performance Insights
- Container Insights

## メディアファイルの配信
S3またはCloudFrontと署名付きURLの組み合わせで対応できます。

# セキュリティ対策

## 脆弱性検知
Amazon Inspectorを導入し、定期的なスキャンを実施。

## WAF設定
最低限以下の対策は必要：
- DDoS対策
- SQLインジェクション対策
- XSS対策
- などなど、WAF側でマネージドに使えるものを利用するのがいいかも

# デプロイ戦略

## デプロイ頻度
理想はトラブルを最小限に抑える・トラブルシューティングやロールバックのしやすさからpushする度にデプロイですが、現実的には週1回程度が多いです。

## デプロイ方式
Blue/Greenデプロイがおすすめ。安全に新バージョンへの切り替えができます。

## データベースマイグレーション
LambdaまたはFargateを使って自動化。
デプロイプロセスの一部としてGitHub Actionsに組み込むのが理想的です。

# さいごに

スタートアップでは、過度に複雑な構成は避け、必要十分な構成からスタートすることが重要です。
事業の成長に応じて、必要な機能を追加していく方針が効果的だと考えています。
