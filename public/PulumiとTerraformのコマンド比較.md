---
title: PulumiとTerraformのコマンド比較
tags:
  - Terraform
  - Pulumi
private: false
updated_at: ''
id: ''
organization_url_name: null
slide: false
ignorePublish: false
---

# PulumiとTerraformのコマンド比較

Infrastructure as Code（IaC）ツールとして人気のあるPulumiとTerraformのコマンド比較表です。
どちらかのツールに慣れている方が、もう一方のツールを使う際の参考にしてください。

## コマンド比較表

| 操作 | Pulumi | Terraform | 説明 |
|------|--------|-----------|------|
| 初期化 | `pulumi new [テンプレート]` | `terraform init` | プロジェクトの初期化 |
| プラン確認 | `pulumi preview` | `terraform plan` | 変更内容のプレビュー |
| デプロイ | `pulumi up` | `terraform apply` | リソースのデプロイ/更新 |
| 削除 | `pulumi destroy` | `terraform destroy` | リソースの削除 |
| 状態確認 | `pulumi stack` | `terraform state list` | 管理されているリソースの一覧表示 |
| 出力確認 | `pulumi stack output` | `terraform output` | 出力変数の表示 |

## 言語とシンタックスの違い

Terraformは独自のHCL（HashiCorp Configuration Language）を使用するのに対し、Pulumiは一般的なプログラミング言語（TypeScript、JavaScript、Python、Go、.NET）を使用できます。

**Terraform (HCL)**
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
  acl    = "private"
  
  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}
```

**Pulumi (TypeScript)**
```typescript
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.Bucket("example", {
    bucket: "my-bucket",
    acl: "private",
    tags: {
        Name: "My bucket",
        Environment: "Dev",
    },
});
```

## まとめ

- **Terraform**：独自のHCL構文を使用し、宣言的なアプローチ。広範なコミュニティとエコシステム。
- **Pulumi**：一般的なプログラミング言語を使用し、命令的なコントロールも可能。強力な型システムとテストフレームワークが利用可能。
