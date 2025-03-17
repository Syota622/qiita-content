---
title: PulumiとTerraformのコマンド比較
tags:
  - Terraform
  - Pulumi
private: false
updated_at: '2025-03-17T22:59:58+09:00'
id: 7169988bf1de65700da4
organization_url_name: null
slide: false
ignorePublish: false
---

# PulumiとTerraformのコマンド比較

Infrastructure as Code（IaC）ツールとして人気のあるPulumiとTerraformのコマンド比較表を作成しました。
どちらかというと、Pulumiは馴染みのないIaCかと思います。
最近になって、初めて利用する機会があり、有名どころのTerraformと比較してみました。

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
