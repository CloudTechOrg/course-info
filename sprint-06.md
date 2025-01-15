[【ハンズオン】CodeシリーズでS3バケットの静的ウェブサイトホスティングを自動デプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでS3バケットの静的ウェブサイトホスティングを自動デプロイする)

---

[【ハンズオン】CodeシリーズでECS Fargateサービスをローリングデプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでECS-Fargateサービスをローリングデプロイする)

---

[【ハンズオン】CodeシリーズでECS FargateサービスをBlueGreenデプロイする](/sprint-06-file/【ハンズオン】CodeシリーズでECS-FargateサービスをBlueGreenデプロイする)

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