---
title: "AWS Bedrockで Application Inference Profile を作成してタグで利用状況を追跡する"
emoji: "🏷️"
type: "tech"
topics: ["AWS", "Bedrock", "InferenceProfile", "CloudFormation", "コスト管理"]
published: false
---

# AWS Bedrock Application Inference Profile のセットアップ

AWS Bedrock の **Application Inference Profile** は、複数リージョンのモデルをまとめて管理できます。

この記事では、Application Inference Profile を CloudFormation で作成する手順をまとめます。自分用のメモも兼ねています。

> 2026年5月時点の内容です。
> 確認に使った `aws cli` のバージョンは `aws-cli/2.x.x` です。

公式ドキュメント:

https://docs.aws.amazon.com/bedrock/

## Application Inference Profile とは

Application Inference Profile は、複数リージョンのモデルをアグリゲートして、単一の ARN でアクセスできる AWS Bedrock の機能です。

通常のモデル ARN とは異なり、Application Inference Profile の ARN を使うことで、複数リージョンのモデルにアクセスできるようになります。

## 前提条件の確認

デプロイの前に、AWS CLI と認証状態を確認しておきます。

```bash
aws --version
aws configure get region
aws sts get-caller-identity
```

出力例:

```json
{
  "UserId": "<user-id>",
  "Account": "<account-id>",
  "Arn": "arn:aws:iam::<account-id>:user/<user-name>"
}
```

## CloudFormation テンプレートで Application Inference Profile を作成

### テンプレートの場所

リポジトリには `scripts/cfn/application-inference-profile.yaml` という CloudFormation テンプレートが含まれています。

このテンプレートでは以下を作成します：

- Claude Haiku 4.5 ベースの Application Inference Profile
- Nova 2 Lite ベースの Application Inference Profile

テンプレート内のパラメータは、デプロイ時に `--parameter-overrides` で上書きできます。

### テンプレート検証

デプロイの前にテンプレートが有効か確認します。

```bash
aws cloudformation validate-template \
  --template-body file://scripts/cfn/application-inference-profile.yaml
```

### デプロイコマンド

実際にスタックをデプロイします。

```bash
aws cloudformation deploy \
  --stack-name application-inference-profile \
  --template-file scripts/cfn/application-inference-profile.yaml \
  --parameter-overrides \
    InferenceProfileName=application-inference-profile-apne1-claude-haiku-45 \
    Nova2LiteInferenceProfileName=application-inference-profile-apne1-nova-2-lite \
  --no-fail-on-empty-changeset
```

成功時の出力例:

```text
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - application-inference-profile
```

## 作成されたリソースの確認

### スタック出力の確認

デプロイ後、スタックの出力内容を確認します。ここで Application Inference Profile の ARN を取得できます。

```bash
aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query 'Stacks[0].Outputs' \
  --output table
```

確認できる主な Output:

- `InferenceProfileArn`: Claude Haiku 4.5 用の ARN
- `InferenceProfileStatus`: ACTIVE など
- `Nova2LiteInferenceProfileArn`: Nova 2 Lite 用の ARN
- `Nova2LiteInferenceProfileStatus`: ACTIVE など

### プロファイルの詳細確認

作成したプロファイルの詳細情報を確認します。

```bash
PROFILE_ARN=$(aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query "Stacks[0].Outputs[?OutputKey=='Nova2LiteInferenceProfileArn'].OutputValue | [0]" \
  --output text)

aws bedrock get-inference-profile \
  --inference-profile-identifier "$PROFILE_ARN"
```

返り値例:

```json
{
  "inferenceProfileName": "application-inference-profile-apne1-nova-2-lite",
  "inferenceProfileArn": "arn:aws:bedrock:ap-northeast-1:<account-id>:application-inference-profile/<profile-id>",
  "inferenceProfileId": "<profile-id>",
  "status": "ACTIVE",
  "type": "APPLICATION",
  "models": [
    {
      "modelArn": "arn:aws:bedrock:ap-northeast-3::foundation-model/amazon.nova-2-lite-v1:0"
    },
    {
      "modelArn": "arn:aws:bedrock:ap-northeast-1::foundation-model/amazon.nova-2-lite-v1:0"
    }
  ]
}
```

## まとめ

Application Inference Profile を CloudFormation で作成することで、複数リージョンのモデルを単一の ARN でアクセスできるようになります。

セットアップが完了したら、ARN を確認して、実際の利用に進めます。

---

**注記**: この記事は個人的なメモも兼ねています。
