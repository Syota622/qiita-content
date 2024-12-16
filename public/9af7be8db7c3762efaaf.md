---
title: 超簡単！GitHub Actionsが「失敗した場合のみ」Slackへ通知する方法
tags:
  - Git
  - GitHub
  - GitHubActions
private: false
updated_at: '2024-09-28T12:10:16+09:00'
id: 9af7be8db7c3762efaaf
organization_url_name: null
slide: false
ignorePublish: false
---
# GitHub Actions 「失敗のみ通知」のコード
GitHub Actionsに下記コードを追加するだけ！！！
実装しているソースコードの最終行に追加すれば良いです！！
```yml
      # 失敗時はこちらのステップが実行される
      - name: Slack Notification on Failure
        uses: rtCamp/action-slack-notify@v2.0.2
        if: failure()
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_TITLE: GitHub Comment
          SLACK_COLOR: danger
          SLACK_MESSAGE: '<!here> Docker Cache Retention job failed!'
```

# ソースコードの説明

1. `name: Slack Notification on Failure`
   このステップの名前を定義しています。GitHubのアクション実行ログでこの名前が表示されます

2. `uses: rtCamp/action-slack-notify@v2.0.2`
   rtCampが提供するSlack通知アクションを使用しています。バージョン2.0.2を指定しています

3. `if: failure()`
   このステップは、ワークフローの前のステップのいずれかが失敗した場合にのみ実行されます

4. `env:` 以下の環境変数設定：
   - `SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}`
     Slack通知を送信するためのWebhook URLを、GitHubのシークレットから取得します。
     最初のうちはテスト用のWEBHOOK_URLをハードコーディングするのもありですが、開発・本番運用するWEBHOOK_URLをコミットするのはセキュリティ上よろしくないため、注意

   - `SLACK_TITLE: GitHub Comment`
   　Slack通知のタイトルを設定します

   - `SLACK_COLOR: danger`
     通知の色を赤（danger）に設定し、失敗を視覚的に示します

   - `SLACK_MESSAGE: '<!here> Docker Cache Retention job failed!'`
     通知の本文メッセージを設定します。`<!here>`はSlackの@hereメンションで、チャンネル内のアクティブなメンバー全員に通知します

このステップにより、Docker Cache Retentionジョブが失敗した際に、設定されたSlackチャンネルに通知が送られ、チーム全体に問題を迅速に知らせることができます。

# Slack 通知
Slackへの通知は、添付画像の通りです。
@hereも追記しており、Slackからのメンションも行えるため非常に便利です！！！
![スクリーンショット 2024-09-28 11.17.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/eb53340f-4963-9b2b-02b1-471dc0153911.png)
