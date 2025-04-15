---
title: SS3-S3を選択した場合のバケットキー有効有無の違い
tags:
  - AWS
  - S3
  - KMS
private: false
updated_at: ''
id: ''
organization_url_name: null
slide: false
ignorePublish: false
---
# S3バケットキーとSSE-S3の関係

## はじめに

S3バケットの暗号化設定について、「SSE-S3」と「バケットキー」の関係性について混乱することがあります。

## バケットキーとは

バケットキーは、AWS KMS呼び出しのコスト削減を目的とした機能で、主に「SSE-KMS」暗号化方式で使用されるものです。
バケットレベルのキーを生成し、S3に一時的に保存することで、AWS KMSへのリクエスト回数を大幅に削減できます。

## SSE-S3とバケットキーの関係

結論から言うと：

> **SSE-S3暗号化使用時、バケットキーの有効/無効設定は挙動に影響を与えません**

理由は以下の通りです：

1. バケットキーはKMS呼び出しコスト削減のための機能
2. SSE-S3はS3内で完結する暗号化方式のため、KMS呼び出しが発生しない
3. したがって、SSE-S3使用時はバケットキーが有効でも無効でも違いはない

## UI上の注意点

S3コンソール上では**SSE-S3設定時でもバケットキーの有効/無効を切り替えることが可能です。**  
しかし、この設定はSSE-S3使用時には実質的に何の効果も持ちません。

## 将来的にSS3-S3からSSE-KMSへの設定変更を考慮する場合

AWS re:Postのコミュニティ回答によると：(<a href="#参考ドキュメント">参考ドキュメント[2]</a>)  
個人的な意見ですが、バケットキーは有効化しておけばいいかなと思いました。

> 「将来的にSSE-KMSに切り替える可能性がある場合や、個別のオブジェクトレベルでSSE-KMSを使用する可能性がある場合は、バケットキーを有効にしておくとよいでしょう。」

## まとめ

- SSE-S3暗号化使用時、バケットキーの設定は効果がない
- AWSの公式ドキュメントには明記されていないが、コミュニティ回答で確認可能
- 将来的にSSE-KMSへの移行を考慮する場合は、バケットキーを有効にしておくのも一つの選択肢

## 参考ドキュメント
[1]
[AWS re:Postのコミュニティ回答](https://repost.aws/ja/questions/QUNfHSIoUZQru2sxvaoPEdyQ/enable-bucket-key-with-sse-s3)

```
Keeping bucket key enabled or disabled won't make any difference as it's essentially a concept for SSE-KMS. Keeping bucket key enabled for SSE-KMS reduce the overall KMS cost substantially as KMS API calls request gets reduced marginally by up to 99% but that won't be the case for SSE-S# as in case of SSE-S3, AWS KMS doesn't come in to the picture for user.

バケットキーを有効または無効にしても、SSE-KMSにとっては本質的に同じことなので、違いはありません。SSE-KMSでバケットキーを有効にしておくと、KMS APIコールの要求が最大99%までわずかに減少するため、KMSの全体的なコストが大幅に削減されますが、SSE-S#の場合は、SSE-S3の場合と同様に、AWS KMSはユーザーには関係のない話です。
```

[2]
[AWS re:Postのコミュニティ回答](https://repost.aws/ja/questions/QUQPqQONRJR82adgj3xsSu0w/difference-in-functionality-and-cost-when-enabling-s3-bucket-key-for-default-encryption)

```
It's good to leave the option enabled, despite using SSE-S3, simply so that if you decide to switch to SSE-KMS later or if individual objects are uploaded with SSE-KMS selected at the object level, the SSE-KMS encryption process will leverage bucket keys for optimising costs.

SSE-S3を使用しているにもかかわらず、オプションを有効にしたままにしておくのは良いことです。後でSSE-KMSに切り替えることを決めた場合や、SSE-KMSがオブジェクトレベルで選択された状態で個々のオブジェクトがアップロードされた場合、SSE-KMSの暗号化プロセスではバケットキーが活用され、コストが最適化されます。
```
