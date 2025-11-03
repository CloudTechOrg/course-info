#### 【ハンズオン】CI/CD初級編/CopePipelineとCodeBuildでECSタスクをデプロイしよう
[https://github.com/CloudTechOrg/container-sample](https://github.com/CloudTechOrg/container-sample)

<details>
  <summary> IAMポリシー
  </summary> 
  
  ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "ecr:GetAuthorizationToken",
                        "ecr:BatchCheckLayerAvailability",
                        "ecr:GetDownloadUrlForLayer",
                        "ecr:GetRepositoryPolicy",
                        "ecr:DescribeRepositories",
                        "ecr:ListImages",
                        "ecr:DescribeImages",
                        "ecr:BatchGetImage",
                        "ecr:InitiateLayerUpload",
                        "ecr:UploadLayerPart",
                        "ecr:CompleteLayerUpload",
                        "ecr:PutImage"
                    ],
                    "Resource": "*"
                }
            ]
        }
```
</details>

---

[【ハンズオン】CodeシリーズでS3バケットの静的ウェブサイトホスティングを自動デプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでS3バケットの静的ウェブサイトホスティングを自動デプロイする)

---

[【ハンズオン】CodeシリーズでECS Fargateサービスをローリングデプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでECS-Fargateサービスをローリングデプロイする)

---

[【ハンズオン】CodeシリーズでECS FargateサービスをBlueGreenデプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでECS-FargateサービスをBlueGreenデプロイする)

---

[【ハンズオン】ECSネイティブ機能BlueGreenデプロイメント/ECS execを使用してコンテナログイン【30:15】](#ecs-native-bluegreen-deployment)

---

<details>
  <summary> 【AWS CLI】CLIコマンド実行時のMFA追加認証設定
  </summary> 

  #### IAMロール信頼ポリシー
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::715731572821:user/test-2"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
  ```

```
source_profile =
role_arn =
mfa_serial =
```
</details>

---

## 【ハンズオン】ECSネイティブ機能BlueGreenデプロイメント/ECS execを使用してコンテナログイン【30:15】 {#ecs-native-bluegreen-deployment}

<details>
  <summary> CodePipelineECSDeployPolicy(IAM Policy)
  </summary>

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": [
        "arn:aws:iam::<ACCOUNT_ID>:role/ecsTaskExecutionRole",
        "arn:aws:iam::<ACCOUNT_ID>:role/ecsExecTaskRole"
      ],
      "Condition": {
        "StringEquals": {
          "iam:PassedToService": "ecs-tasks.amazonaws.com"
        }
      }
    }
  ]
}
```
</details>

<details>
  <summary> cth1-deploy-check(Lambda)
  </summary>

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "hookStatus": "SUCCEEDED"
    }
```
</details>

<details>
  <summary> cth1-ecsDeploymentHookRole(IAM Role)
  </summary>

#### 信頼ポリシー
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
</details>

<details>
  <summary> ECS Exec 検証
  </summary>

```bash
grep nginx /proc/*/comm 2>/dev/null
yes > /dev/null &
grep yes /proc/*/comm 2>/dev/null
kill $(grep -l yes /proc/*/comm | cut -d/ -f3)
```
</details>