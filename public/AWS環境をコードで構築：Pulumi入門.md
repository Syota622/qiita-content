---
title: AWS環境をコードで構築：Pulumi入門
tags:
  - AWS
  - TypeScript
  - Pulumi
private: false
updated_at: '2025-03-16T17:23:46+09:00'
id: 6879a6a6c75cd9a8f5d4
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

現場でPulumiというIaCを初めて利用することになりました。
そこで、TypeScriptを使ってAWSリソースを管理できるPulumiというツールの基本を無料で学びました。

# Pulumiとは？

Pulumiは、クラウドインフラを一般的なプログラミング言語（TypeScript、JavaScript、Python、Go、.NETなど）で定義できるIaCツールです。
従来のAWS CloudFormationやTerraformとは異なり、普段使い慣れたプログラミング言語でインフラを構築できるのが特徴です。
また、AWSだけでなく、Azure、Google Cloudなど、複数のクラウドプロバイダーに対応しています。

# 環境構築

まずはPulumiをインストールし、AWSの認証情報を設定しました。

## Pulumiのインストール

macOSで、勉強をしたため、brewでインストールしました。

```bash
% brew install pulumi
```

インストールが完了したら、以下のコマンドでバージョンを確認します。

```bash
% pulumi version
```

## AWS認証情報の設定

AWS認証情報の設定をCLIで行います。

```bash
# 認証情報の設定
% aws configure
AWS Access Key ID [****************]:
AWS Secret Access Key [****************]:
Default region name [ap-northeast-1]:
Default output format [json]:
```

プロンプトに従って、AWSのアクセスキー、シークレットキー、デフォルトリージョンなどを入力します。

# プロジェクト構造の作成

効率的にコードを管理するため、AWSサービスごとにファイルを分ける構造を採用しました。

```bash
# メインプロジェクトディレクトリを作成
% mkdir pulumi-aws-infra
% cd pulumi-aws-infra
```

```bash
# Pulumiへログイン
% pulumi login
Manage your Pulumi stacks by logging in.
Run `pulumi login --help` for alternative login options.
Enter your access token from https://app.pulumi.com/account/tokens
    or hit <ENTER> to log in using your browser                   :
We've launched your web browser to complete the login process.

Waiting for login to complete...


  Welcome to Pulumi!

  Pulumi helps you create, deploy, and manage infrastructure on any cloud using
  your favorite language. You can get started today with Pulumi at:

      https://www.pulumi.com/docs/get-started/

  Tip: Resources you create with Pulumi are given unique names (a randomly
  generated suffix) by default. To learn more about auto-naming or customizing resource
  names see https://www.pulumi.com/docs/intro/concepts/resources/#autonaming.

```

```bash
# Pulumiプロジェクトの初期化
% pulumi new aws-typescript
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

Project name (pulumi-aws-infra):
Project description (A minimal AWS TypeScript Pulumi program):
Created project 'pulumi-aws-infra'

Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).
Stack name (dev):
Created stack 'dev'

The package manager to use for installing dependencies npm
The AWS region to deploy into (aws:region) (us-east-1): ap-northeast-1
Saved config

Installing dependencies...


added 392 packages, and audited 393 packages in 34s

46 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Finished installing dependencies

Your new project is ready to go! ✨

To perform an initial deployment, run `pulumi up`
```

```bash
# 構造化されたディレクトリを作成
% mkdir -p src/{network,compute,storage,database,security}

# ディレクトリ構造を確認
% ls -la
total 472
drwxr-xr-x   11 suzukishouta  staff     352  3 16 16:43 .
drwx------@  18 suzukishouta  staff     576  3 16 16:42 ..
-rw-r--r--    1 suzukishouta  staff      21  3 16 16:42 .gitignore
-rw-r--r--    1 suzukishouta  staff      37  3 16 16:42 Pulumi.dev.yaml
-rw-r--r--    1 suzukishouta  staff     207  3 16 16:42 Pulumi.yaml
-rw-r--r--    1 suzukishouta  staff     275  3 16 16:42 index.ts
drwxr-xr-x  222 suzukishouta  staff    7104  3 16 16:42 node_modules
-rw-r--r--    1 suzukishouta  staff  216585  3 16 16:42 package-lock.json
-rw-r--r--    1 suzukishouta  staff     285  3 16 16:42 package.json
drwxr-xr-x    7 suzukishouta  staff     224  3 16 16:43 src
-rw-r--r--    1 suzukishouta  staff     438  3 16 16:42 tsconfig.json
```

# VPC、IGW、サブネットグループ、ルートテーブルの作成

参考: 下記コードをvimでコピー&ペースト
```bash
% vim src/network/vpc.ts
% vim src/network/internet-gateway.ts
% vim src/network/public-subnet.ts
% vim src/network/public-route.ts
% vim src/network/index.ts
% vim index.ts
% vim Pulumi.dev.yaml
```

## VPCモジュール

`src/network/vpc.ts` ファイルを作成します：

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// VPCの引数を定義するインターフェース
export interface VpcArgs {
    cidrBlock: string;
    tags?: { [key: string]: string };
}

// VPCを作成するクラス
export class Vpc extends pulumi.ComponentResource {
    public readonly vpc: aws.ec2.Vpc;

    constructor(name: string, args: VpcArgs, opts?: pulumi.ComponentResourceOptions) {
        super("custom:network:Vpc", name, {}, opts);

        // デフォルトのタグを準備
        const defaultTags = {
            Name: name,
            ManagedBy: "Pulumi"
        };

        // ユーザー指定のタグとマージ
        const tags = { ...defaultTags, ...(args.tags || {}) };

        // VPCを作成
        this.vpc = new aws.ec2.Vpc(name, {
            cidrBlock: args.cidrBlock,
            enableDnsSupport: true,
            enableDnsHostnames: true,
            tags: tags,
        }, { parent: this });

        this.registerOutputs({
            vpcId: this.vpc.id,
            vpcArn: this.vpc.arn,
            vpcCidrBlock: this.vpc.cidrBlock,
        });
    }
}
```

## IGWモジュール

VPCをインターネットに接続するため、`src/network/internet-gateway.ts` ファイルを作成します：

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// インターネットゲートウェイの引数を定義するインターフェース
export interface InternetGatewayArgs {
    vpcId: pulumi.Input<string>;
    tags?: { [key: string]: string };
}

// インターネットゲートウェイを作成するクラス
export class InternetGateway extends pulumi.ComponentResource {
    public readonly internetGateway: aws.ec2.InternetGateway;

    constructor(name: string, args: InternetGatewayArgs, opts?: pulumi.ComponentResourceOptions) {
        super("custom:network:InternetGateway", name, {}, opts);

        // デフォルトのタグを準備
        const defaultTags = {
            Name: name,
            ManagedBy: "Pulumi"
        };

        // ユーザー指定のタグとマージ
        const tags = { ...defaultTags, ...(args.tags || {}) };

        // インターネットゲートウェイを作成
        this.internetGateway = new aws.ec2.InternetGateway(name, {
            vpcId: args.vpcId,
            tags: tags,
        }, { parent: this });

        this.registerOutputs({
            internetGatewayId: this.internetGateway.id,
            internetGatewayArn: this.internetGateway.arn,
        });
    }
}
```

## パブリックサブネットモジュール

次に、インターネットからアクセス可能なパブリックサブネットを定義します。`src/network/public-subnet.ts` ファイルを作成します：

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// パブリックサブネットの引数を定義するインターフェース
export interface PublicSubnetArgs {
    vpcId: pulumi.Input<string>;
    cidrBlock: string;
    availabilityZone: string;
    mapPublicIpOnLaunch?: boolean;
    tags?: { [key: string]: string };
}

// パブリックサブネットを作成するクラス
export class PublicSubnet extends pulumi.ComponentResource {
    public readonly subnet: aws.ec2.Subnet;

    constructor(name: string, args: PublicSubnetArgs, opts?: pulumi.ComponentResourceOptions) {
        super("custom:network:PublicSubnet", name, {}, opts);

        // デフォルトのタグを準備
        const defaultTags = {
            Name: name,
            ManagedBy: "Pulumi",
            Type: "Public" // パブリックサブネットであることを明示
        };

        // ユーザー指定のタグとマージ
        const tags = { ...defaultTags, ...(args.tags || {}) };

        // パブリックサブネットを作成
        this.subnet = new aws.ec2.Subnet(name, {
            vpcId: args.vpcId,
            cidrBlock: args.cidrBlock,
            availabilityZone: args.availabilityZone,
            mapPublicIpOnLaunch: args.mapPublicIpOnLaunch !== undefined ? args.mapPublicIpOnLaunch : true,
            tags: tags,
        }, { parent: this });

        this.registerOutputs({
            subnetId: this.subnet.id,
            subnetArn: this.subnet.arn,
            subnetCidr: this.subnet.cidrBlock,
        });
    }
}
```

## パブリックルートテーブルモジュール

サブネットからインターネットへの接続を定義するため、`src/network/public-route.ts` ファイルを作成します：

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// パブリックルートテーブルの引数を定義するインターフェース
export interface PublicRouteTableArgs {
    vpcId: pulumi.Input<string>;
    internetGatewayId: pulumi.Input<string>;
    tags?: { [key: string]: string };
}

// パブリックルートテーブルを作成・管理するクラス
export class PublicRouteTable extends pulumi.ComponentResource {
    public readonly routeTable: aws.ec2.RouteTable;
    public readonly internetRoute: aws.ec2.Route;
    public readonly associations: aws.ec2.RouteTableAssociation[] = [];

    constructor(name: string, args: PublicRouteTableArgs, opts?: pulumi.ComponentResourceOptions) {
        super("custom:network:PublicRouteTable", name, {}, opts);

        // デフォルトのタグを準備
        const defaultTags = {
            Name: name,
            ManagedBy: "Pulumi",
            Type: "Public" // パブリックルートテーブルであることを明示
        };

        // タグをマージ
        const tags = { ...defaultTags, ...(args.tags || {}) };

        // パブリックルートテーブルを作成
        this.routeTable = new aws.ec2.RouteTable(name, {
            vpcId: args.vpcId,
            tags: tags,
        }, { parent: this });

        // インターネットへのルートを自動的に追加
        this.internetRoute = new aws.ec2.Route(`${name}-internet`, {
            routeTableId: this.routeTable.id,
            destinationCidrBlock: "0.0.0.0/0",
            gatewayId: args.internetGatewayId,
        }, { parent: this });

        this.registerOutputs({
            routeTableId: this.routeTable.id,
            internetRouteId: this.internetRoute.id,
        });
    }

    // サブネットとルートテーブルを関連付けるメソッド
    public associateSubnet(subnetId: pulumi.Input<string>, name?: string): aws.ec2.RouteTableAssociation {
        // 関連付け名を生成
        const associationName = name || `${name}-assoc-${this.associations.length + 1}`;
        
        // 関連付けリソースを作成
        const association = new aws.ec2.RouteTableAssociation(associationName, {
            routeTableId: this.routeTable.id,
            subnetId: subnetId,
        }, { parent: this });
        
        // 作成した関連付けを配列に追加
        this.associations.push(association);
        
        // 作成した関連付けを返す
        return association;
    }
}
```

## モジュールのエクスポート

作成したモジュールをまとめてエクスポートするため、`src/network/index.ts` ファイルを作成します：

```typescript
// ネットワークモジュールをエクスポート
export * from "./vpc";
export * from "./internet-gateway";
export * from "./public-subnet";
export * from "./public-route";
```

## メインファイルの実装

最後に、メインの `index.ts` ファイルでこれらのモジュールを使用してリソースを作成します：

```typescript
import * as pulumi from "@pulumi/pulumi";
import { Vpc, InternetGateway, PublicSubnet, PublicRouteTable } from "./src/network";

// 設定を取得
const config = new pulumi.Config();
const projectName = config.get("projectName") || "pulumi-aws-infra";
const env = config.get("environment") || "dev";

// 共通のタグ
const commonTags = {
    Project: projectName,
    Environment: env,
    ManagedBy: "Pulumi"
};

// 可用性ゾーンの設定
const availabilityZones = ["us-east-1a", "us-east-1b"];

// VPCの作成
const mainVpc = new Vpc(`${projectName}-vpc-${env}`, {
    cidrBlock: "10.0.0.0/16",
    tags: commonTags,
});

// インターネットゲートウェイの作成
const mainIgw = new InternetGateway(`${projectName}-igw-${env}`, {
    vpcId: mainVpc.vpc.id,
    tags: commonTags,
});

// パブリックサブネットの作成
const publicSubnets: PublicSubnet[] = [];

for (let i = 0; i < 2; i++) {
    const az = availabilityZones[i];
    const cidr = `10.0.${i}.0/24`;
    
    const subnet = new PublicSubnet(`${projectName}-public-subnet-${i+1}-${env}`, {
        vpcId: mainVpc.vpc.id,
        cidrBlock: cidr,
        availabilityZone: az,
        mapPublicIpOnLaunch: true,
        tags: {
            ...commonTags,
            Name: `${projectName}-public-subnet-${i+1}-${env}`,
        },
    });
    
    publicSubnets.push(subnet);
}

// パブリックルートテーブルの作成
const publicRouteTable = new PublicRouteTable(`${projectName}-public-rt-${env}`, {
    vpcId: mainVpc.vpc.id,
    internetGatewayId: mainIgw.internetGateway.id,
    tags: commonTags,
});

// パブリックサブネットにルートテーブルを関連付け
for (let i = 0; i < publicSubnets.length; i++) {
    const subnet = publicSubnets[i];
    publicRouteTable.associateSubnet(
        subnet.subnet.id,
        `${projectName}-public-rt-assoc-${i+1}-${env}`
    );
}

// 出力値の設定
export const vpcId = mainVpc.vpc.id;
export const vpcCidrBlock = mainVpc.vpc.cidrBlock;
export const internetGatewayId = mainIgw.internetGateway.id;
export const publicSubnetIds = publicSubnets.map(subnet => subnet.subnet.id);
export const publicRouteTableId = publicRouteTable.routeTable.id;
```

# デプロイと管理

## 環境設定ファイル

開発環境の設定を行うため、`Pulumi.dev.yaml` ファイルを作成します。

```yaml
config:
  aws:region: ap-northeast-1  # 使用するリージョンに変更してください
  pulumi-aws-infra:environment: dev
  pulumi-aws-infra:projectName: my-awesome-project
```

## デプロイ前のチェック

実際にリソースを作成する前に、変更内容をプレビューします。

```bash
# プロジェクトの依存関係をインストール
% npm install

# TypeScriptコードの型チェック
% tsc --noEmit

# 変更のプレビュー（デプロイは行わない）
% pulumi preview
```

問題がなければ、以下のコマンドでデプロイを実行します：

```bash
# スタックの選択（初回実行時）
% pulumi stack select dev

# スタックの確認
% pulumi stack ls
NAME  LAST UPDATE  RESOURCE COUNT  URL
dev*  n/a          n/a             https://app.pulumi.com/???/pulumi-aws-infra/dev

# デプロイの実行
% pulumi up
```

## リソースの状態確認

作成されたリソースの状態を確認するには。

```bash
# スタックの出力値を確認
% pulumi stack output
```

## リソースの削除

不要になったリソースを削除するには。

```bash
# すべてのリソースを削除
% pulumi destroy

# スタック自体を削除（オプション）
% pulumi stack rm dev
```

# TypeScriptの基本構文解説

Pulumi with TypeScriptを使う際に役立つ基本的な構文を解説します。

## インターフェース
オブジェクトの形状（どんなプロパティを持つか）を定義します。
```typescript
interface VpcArgs {
    cidrBlock: string;  // 必須プロパティ
    tags?: { [key: string]: string };  // ?を付けると任意プロパティになる
}
```

## クラス
関連する機能とデータをまとめたものです。
```typescript
class Vpc extends pulumi.ComponentResource {
    // プロパティ定義
    public readonly vpc: aws.ec2.Vpc;

    // コンストラクタ
    constructor(name: string, args: VpcArgs, opts?: pulumi.ComponentResourceOptions) {
        // 親クラスのコンストラクタ呼び出し
        super("custom:network:Vpc", name, {}, opts);
        
        // 実装...
    }
}
```

## スプレッド構文
オブジェクトやプロパティを展開します。
```typescript
const defaultTags = { Name: "my-vpc" };
const userTags = { Environment: "dev" };
const allTags = { ...defaultTags, ...userTags };  // { Name: "my-vpc", Environment: "dev" }
```

## 配列操作
配列を処理する便利なメソッドがあります。
```typescript
// mapメソッド：各要素を変換
const subnetIds = subnets.map(subnet => subnet.id);

// forEachメソッド：各要素に対して処理
subnets.forEach((subnet, index) => {
    // 処理...
});
```

# まとめ

Pulumiを使うことで、TypeScriptという汎用プログラミング言語を使ってAWSインフラをコード化できました。
今回はVPC、インターネットゲートウェイ、サブネット、ルートテーブルという基本的なネットワークリソースを作成しましたが、同じアプローチでEC2インスタンス、RDSデータベース、S3バケットなど、他のAWSリソースも管理できます。

コードを構造化し、AWSサービスごとにモジュール化することで、コードの再利用性が高まり、大規模なインフラ管理にも対応できます。

Infrastructure as Codeの利点は、バージョン管理できること、チーム間で共有できること、そして自動化できることです。
Pulumiはその中でも、普段使い慣れたプログラミング言語を活用できる点が大きな強みです。

# 次のステップ

- プライベートサブネットとNATゲートウェイの追加
- セキュリティグループの作成
- EC2インスタンスのデプロイ
- RDSデータベースの設定
- S3バケットの作成

Pulumiを使ったAWSインフラ構築を、ぜひ続けてみてください！
