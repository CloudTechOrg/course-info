<details>
  <summary> 
    【EC2】EC2インスタンス構築デモ/ユーザーデータ/インスタンスメタデータ/windowsサーバー作成【21:20】
  </summary>

```bash
#!/bin/bash
echo "===yum -y install httpd==="
yum -y install httpd

echo "===systemctl start httpd.service==="
systemctl start httpd.service

echo "===systemctl enable httpd.service==="
systemctl enable httpd.service

echo "===get token===" #トークンを取得
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
echo $TOKEN

echo "===get instance-id >> /tmp/test===" #見出し
echo "instance-id" >> /tmp/test #見出し
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id >> /tmp/test

echo "===get Tag===" #見出し
echo "Name tag" >> /tmp/test #見出し
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/tags/instance/Name >> /tmp/test

echo "Environment tag" >> /tmp/test #見出し
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/tags/instance/Environment >> /tmp/test
```

</details>

