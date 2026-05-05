---
title: "AWS Bedrockで Application Inference Profile を作成してタグで利用状況を追跡する"
emoji: "🏷️"
type: "tech"
topics: ["AWS", "Bedrock", "InferenceProfile", "CloudFormation", "コスト管理"]
published: false
---

# AWS Bedrock Application Inference Profile のセットアップ

AWS Bedrock の **Application Inference Profile** を使うと、複数リージョンにまたがるモデルを 1 つの ARN で扱えます。さらにタグを付けておくと、あとから Cost Explorer で利用状況を追いやすくなります。

この記事では、CloudFormation で Application Inference Profile を作成し、タグ確認、Cost Explorer 用の有効化、`aws bedrock-runtime converse` による動作確認までをまとめます。

> 2026年5月時点の内容です。  
> 確認に使った `aws cli` は `aws-cli/2.34.13 Python/3.14.3 Linux/6.6.87.2-microsoft-standard-WSL2 exe/x86_64.ubuntu.24` です。

公式ドキュメント:

https://docs.aws.amazon.com/bedrock/

## Application Inference Profile とは

Application Inference Profile は、複数リージョンのモデルを集約して、単一の ARN でアクセスできる AWS Bedrock の機能です。

通常のモデル ARN を直接指定する代わりに、Application Inference Profile の ARN を使うことで、アプリケーション側の参照先を固定しつつ、背後では複数リージョンのモデルを利用できます。

## 今回の対象

- リージョン: `ap-northeast-1`
- CloudFormation スタック名: `application-inference-profile`
- 作成する Application Inference Profile
  - `application-inference-profile-apne1-claude-haiku-45`
  - `application-inference-profile-apne1-nova-2-lite`

## 前提条件

- AWS CLI が利用できること
- `jq` が利用できること
- AWS 認証情報が設定済みであること
- Bedrock を利用できる権限があること
- `ap-northeast-1` で対象モデルを利用できること

まずは CLI と認証状態を確認します。

```bash
aws --version
aws configure get region
aws sts get-caller-identity
```

`aws --version` の出力例:

```text
aws-cli/2.34.13 Python/3.14.3 Linux/6.6.87.2-microsoft-standard-WSL2 exe/x86_64.ubuntu.24
```

出力例:

```json
{
  "UserId": "<user-id>",
  "Account": "<account-id>",
  "Arn": "arn:aws:iam::<account-id>:user/<user-name>"
}
```

## CloudFormation テンプレート

テンプレートは `scripts/cfn/application-inference-profile.yaml` です。このテンプレートでは、以下 2 つの Application Inference Profile を作成します。

実際のテンプレートは GitHub の public リポジトリでも参照できます。  
https://github.com/DarthMazz/1-aws-inferenceprofile-1-setup/blob/main/scripts/cfn/application-inference-profile.yaml

- Claude Haiku 4.5 ベース
- Nova 2 Lite ベース

既定値は次のとおりです。

| パラメータ | 値 |
|---|---|
| `InferenceProfileName` | `application-inference-profile-apne1-claude-haiku-45` |
| `ModelSourceProfileId` | `jp.anthropic.claude-haiku-4-5-20251001-v1:0` |
| `Nova2LiteInferenceProfileName` | `application-inference-profile-apne1-nova-2-lite` |
| `Nova2LiteModelSourceProfileId` | `jp.amazon.nova-2-lite-v1:0` |

コピー元 ARN はテンプレートに直書きせず、`AWS::Partition` / `AWS::Region` / `AWS::AccountId` から動的に組み立てます。

また、作成時に次のタグを付与しています。

- `Name`
- `Owner` (`setup0`)

## テンプレートの検証

デプロイ前に CloudFormation テンプレートを検証します。

```bash
aws cloudformation validate-template \
  --template-body file://scripts/cfn/application-inference-profile.yaml
```

返り値にはテンプレートの Description や Parameters が含まれます。

## Application Inference Profile の作成・更新

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

デプロイ後は、まず CloudFormation の Output を確認します。

```bash
aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query 'Stacks[0].Outputs' \
  --output table
```

主に確認したい Output:

- `InferenceProfileArn`
- `InferenceProfileIdentifier`
- `InferenceProfileStatus`
- `Nova2LiteInferenceProfileArn`
- `Nova2LiteInferenceProfileIdentifier`
- `Nova2LiteInferenceProfileStatus`

### `aws bedrock` と `jq` で直接取得する

CloudFormation Output を経由せず、Bedrock API から直接 Application Inference Profile を探すこともできます。

Claude Haiku 4.5 の例:

```bash
aws bedrock list-inference-profiles \
  --region ap-northeast-1 \
  --type-equals APPLICATION \
  --output json \
| jq -r '
  .inferenceProfileSummaries[]
  | select(.inferenceProfileName == "application-inference-profile-apne1-claude-haiku-45")
  | {
      inferenceProfileName,
      inferenceProfileArn,
      inferenceProfileId,
      status
    }
'
```

Nova 2 Lite の ARN だけ取り出す例:

```bash
PROFILE_ARN=$(aws bedrock list-inference-profiles \
  --region ap-northeast-1 \
  --type-equals APPLICATION \
  --output json \
| jq -r '
  .inferenceProfileSummaries[]
  | select(.inferenceProfileName == "application-inference-profile-apne1-nova-2-lite")
  | .inferenceProfileArn
')
```

## 作成したプロファイルの詳細確認

### Claude Haiku 4.5

```bash
PROFILE_ARN=$(aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query "Stacks[0].Outputs[?OutputKey=='InferenceProfileArn'].OutputValue | [0]" \
  --output text)

aws bedrock get-inference-profile \
  --region ap-northeast-1 \
  --inference-profile-identifier "$PROFILE_ARN"
```

### Nova 2 Lite

```bash
PROFILE_ARN=$(aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query "Stacks[0].Outputs[?OutputKey=='Nova2LiteInferenceProfileArn'].OutputValue | [0]" \
  --output text)

aws bedrock get-inference-profile \
  --region ap-northeast-1 \
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

## タグの確認

Application Inference Profile に設定されたタグは `aws bedrock list-tags-for-resource` で確認できます。

```bash
PROFILE_ARN=$(aws bedrock list-inference-profiles \
  --region ap-northeast-1 \
  --type-equals APPLICATION \
  --output json \
| jq -r '
  .inferenceProfileSummaries[]
  | select(.inferenceProfileName == "application-inference-profile-apne1-nova-2-lite")
  | .inferenceProfileArn
')

aws bedrock list-tags-for-resource \
  --region ap-northeast-1 \
  --resource-arn "$PROFILE_ARN" \
  --output json \
| jq
```

`Owner` タグの値だけ確認する場合:

```bash
aws bedrock list-tags-for-resource \
  --region ap-northeast-1 \
  --resource-arn "$PROFILE_ARN" \
  --output json \
| jq -r '.tags[] | select(.key == "Owner") | .value'
```

返り値例:

```json
{
  "tags": [
    {
      "key": "Name",
      "value": "application-inference-profile-apne1-nova-2-lite"
    },
    {
      "key": "Owner",
      "value": "setup0"
    }
  ]
}
```

## Cost Explorer でコストタグとして使うための有効化

Application Inference Profile に `Owner` タグを付けただけでは、Cost Explorer ですぐに絞り込みには使えません。Billing 側で **cost allocation tag** として有効化する必要があります。

手順:

1. AWS Billing and Cost Management コンソールを開く
2. **Cost allocation tags** を開く
3. タグキー `Owner` を選ぶ
4. **Activate** を実行する

注意点:

- タグキーが **Cost allocation tags** 画面に出るまで最大 24 時間かかることがある
- 表示後、タグの有効化完了までさらに最大 24 時間かかることがある
- Cost Explorer のデータ更新も日次なので、実際に見え始めるまで時間差がある
- AWS Organizations 環境では、通常は management account 側で有効化する

過去分のコストも必要なら、Billing の **Backfill tags** で最大 12 か月までバックフィルできます。ただし反映されるのは、当時すでにそのリソースに対象タグが付いていた期間だけです。

## Nova 2 Lite の `converse` 動作確認

Application Inference Profile ARN を `--model-id` に指定して、そのまま推論できます。

```bash
PROFILE_ARN=$(aws cloudformation describe-stacks \
  --stack-name application-inference-profile \
  --query "Stacks[0].Outputs[?OutputKey=='Nova2LiteInferenceProfileArn'].OutputValue | [0]" \
  --output text)

aws bedrock-runtime converse \
  --region ap-northeast-1 \
  --model-id "$PROFILE_ARN" \
  --messages '[{"role":"user","content":[{"text":"Please reply with exactly: Nova 2 Lite via application profile works."}]}]' \
  --inference-config '{"maxTokens":64,"temperature":0}' \
  --output json
```

返り値例:

```json
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [
        {
          "text": "Nova 2 Lite via application profile works."
        }
      ]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 60,
    "outputTokens": 12,
    "totalTokens": 72
  },
  "metrics": {
    "latencyMs": 321
  }
}
```

## 参考: 利用可能な Nova 系 System-Defined Inference Profile の確認

コピー元の System-Defined Inference Profile を確認したい場合は次のコマンドを使います。

```bash
aws bedrock list-inference-profiles \
  --type-equals SYSTEM_DEFINED \
  --region ap-northeast-1 \
  --query "inferenceProfileSummaries[?contains(inferenceProfileName, 'Nova') || contains(inferenceProfileName, 'nova')].[inferenceProfileName,inferenceProfileArn,models[0].modelArn]" \
  --output table
```

## スタック削除

不要になったら CloudFormation スタックごと削除します。

```bash
aws cloudformation delete-stack \
  --stack-name application-inference-profile

aws cloudformation wait stack-delete-complete \
  --stack-name application-inference-profile
```

削除後の確認例:

```text
aws: [ERROR]: An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id application-inference-profile does not exist
```

## まとめ

CloudFormation で Application Inference Profile を作成しておくと、複数リージョンのモデルを 1 つの ARN で扱えるようになります。

さらに `Owner` などのタグを付けて Billing 側で cost allocation tag として有効化しておけば、Cost Explorer で利用状況やコストを追いやすくなります。作成後は `get-inference-profile`、`list-tags-for-resource`、`bedrock-runtime converse` まで確認しておくと安心です。

今後の作業で迷わないように、確認手順も含めて自分向けに整理しておく感覚でまとめました。
