---
title: HTML・CSSで作成したサイトをGitHub Pagesを使って公開しよう！
tags:
  - HTML
  - CSS
  - Git
  - GitHub
private: false
updated_at: '2023-02-22T22:17:33+09:00'
id: 3a0d9daa1cf7c1bddfbe
organization_url_name: null
slide: false
ignorePublish: false
---
# GitHub Pagesとは
サーバーや他のSaaSを利用せずに、無料でインターネット上へ公開できるツール。
ポートフォリオなどで作成したコードをインターネットへ公開したい場合、手間をかけることなく設定ができるためとても便利です！

# リポジトリの作成
はじめに、インターネット上へ公開するためのリポジトリを作成する。

![スクリーンショット 2023-02-20 8.58.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/e6548b8d-3329-06f1-2717-36dbc0f750a8.png)

HTML：index.html、CSS：cssフォルダをアップロードします。
![スクリーンショット 2023-02-20 9.31.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/8929fbc1-7a03-b73e-07b0-2ff70df14074.png)

index.html
```
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>はじめてのGitHub Pages</title>
  <link rel="stylesheet" href="css/base.css">
</head>
<body>
  <h1 style="color: #aa0000;">はじめてのGitHub Pages①</h1>
  <h2>はじめてのGitHub Pages②</h2>
  <h3>はじめてのGitHub Pages③</h3>
</body>
</html>
```
css/base.css
```
h3 {
  color: #0000aa;
}
```

# インターネットへ公開するための設定手順
「Settings」の「Pages」をクリックする。
![スクリーンショット 2023-02-20 9.14.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/68521f19-fe98-69c9-bf1a-59811c7d72b3.png)

「main」ブランチを選択して、「Save」をクリックする。
他ブランチでも対応可能です！（mainブランチへマージする前に確認など）
![スクリーンショット 2023-02-20 9.16.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/3cc51c06-227d-86b7-441c-19771d770dd8.png)

設定が全て完了したため、ブラウザからアクセスして確認できます！
インターネットへの公開URLは *https://アカウント名.github.io/リポジトリ名* となります。
簡単に設定できるため、是非ポートフォリオや履歴書などで、有効活用してください！
![スクリーンショット 2023-02-20 9.22.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/263017/a0554583-3bb8-9a69-c59d-84265abdaa1e.png)
