<details>
  <summary> 【ハンズオン】起動テンプレート作成/ALBとAutoScalingを組み合わせたハンズオン
</summary>

#### ユーザーデータ

```bash
#!/bin/bash
echo "===yum -y install httpd==="
yum -y install httpd

echo "===systemctl start httpd.service==="
systemctl start httpd.service

echo "===systemctl enable httpd.service==="
systemctl enable httpd.service

echo "===get token===" # トークンを取得
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

echo "===get instance-id >> /tmp/test===" # 見出し
echo "instance-id:" >> /var/www/html/index.html # 見出し
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id >> /var/www/html/index.html
echo "   " >> /var/www/html/index.html

echo "===get Tag===" # 見出し
echo "Name tag:" >> /var/www/html/index.html # 見出し
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/tags/instance/Name >> /var/www/html/index.html
```

#### サーバーに負荷を与えるコマンド

```bash
yes >> /dev/null &
```

#### サーバーの処理状況を確認するコマンド

```bash
top
```

#### サーバーの処理状況を確認するtopコマンドを抜けるにはqを押してください

```bash
q
```

#### サーバーに負荷を与えるコマンドのプロセスを終了するコマンド

```bash
pkill yes
```
</details>

---
<details>
  <summary>  【ハンズオン】CloudWatchアラーム設定とSNSによるメール通知設定
  </summary>

#### CPUに負荷をかけて使用率を上げるコマンド

```bash
sudo yum install -y stress
stress --cpu 1 --timeout 600
```