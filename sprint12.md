<details>
  <summary> 
【ハンズオン】マルチアカウント/Organizationsメンバーアカウント作成とスイッチロール設定
</summary> 

#### IAMロール信頼ポリシー

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::715731572821:user/kurokawa"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

#### scp設定
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "iam",
      "Effect": "Deny",
      "Action": [
        "iam:AttachRolePolicy",
        "iam:DeleteRole",
        "iam:DeleteRolePermissionsBoundary",
        "iam:DeleteRolePolicy",
        "iam:DetachRolePolicy",
        "iam:PutRolePermissionsBoundary",
        "iam:PutRolePolicy",
        "iam:UpdateAssumeRolePolicy",
        "iam:UpdateRole",
        "iam:UpdateRoleDescription"
      ],
      "Resource": [
        "arn:aws:iam::*:role/*"
      ],
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalARN": [
            "arn:aws:iam::*:role/Hoka-Admin-Role-test",
            "arn:aws:iam::*:root"
          ]
        }
      }
    }
  ]
}
```

#### IAMロール信頼ポリシー（タグ条件）

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::715731572821:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalTag/Group": "Manager"
                }
            }
        }
    ]
}
```

</details>

<details>
  <summary> 
【ハンズオン】AWS STS/sts:AssumeRoleアクション深掘りハンズオン
</summary> 

#### Hoka-SSM-Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::216989122692:role/Hoka-SSM-Role"
        }
    ]
}
```

#### Hoka-SSM-Roleの信頼ポリシー
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::715731572821:role/My-EC2-Role"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

#### コマンド
```bash
aws sts assume-role 
  --role-arn "arn:aws:iam::216989122692:role/Hoka-SSM-Role" 
  --role-session-name "demo-session"
```

#### コマンド
```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```

#### Bashスクリプト
```bash
#!/bin/bash

ASSUME_ROLE_OUTPUT=$(aws sts assume-role 
  --role-arn "arn:aws:iam::216989122692:role/Hoka-SSM-Role" 
  --role-session-name "demo-session")

AWS_ACCESS_KEY_ID=$(echo $ASSUME_ROLE_OUTPUT | jq -r '.Credentials.AccessKeyId')
AWS_SECRET_ACCESS_KEY=$(echo $ASSUME_ROLE_OUTPUT | jq -r '.Credentials.SecretAccessKey')
AWS_SESSION_TOKEN=$(echo $ASSUME_ROLE_OUTPUT | jq -r '.Credentials.SessionToken')

export AWS_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY
export AWS_SESSION_TOKEN

# ParameterStoreにアクセスしてリストを取得
aws ssm get-parameter --name test
```

#### Lambda関数
```python
import boto3
import json

def lambda_handler(event, context):
    # STSクライアントを作成し、他アカウントのIAMロールをAssumeする
    sts_client = boto3.client('sts')
    
    # AssumeRoleを使って一時的な認証情報を取得
    assumed_role = sts_client.assume_role(
        RoleArn="arn:aws:iam::216989122692:role/Hoka-SSM-Role",  # 他アカウントのロールARN
        RoleSessionName="AssumeRoleSession"
    )
    
    # 一時的な認証情報を使ってSSMクライアントを作成
    credentials = assumed_role['Credentials']
    ssm_client = boto3.client(
        'ssm',
        aws_access_key_id=credentials['AccessKeyId'],
        aws_secret_access_key=credentials['SecretAccessKey'],
        aws_session_token=credentials['SessionToken']
    )
    
    # SSMパラメータ 'test' を取得
    response = ssm_client.get_parameter(
        Name='test',  # 取得したいSSMパラメータの名前
        WithDecryption=True  # 暗号化されたパラメータの場合は復号化する
    )
    
    # パラメータの値を取得
    parameter_value = response['Parameter']['Value']
    
    # 結果を返す
    return {
        'statusCode': 200,
        'body': json.dumps({'ParameterValue': parameter_value}),
        'headers': {
            'Content-Type': 'application/json'
        }
    }
```