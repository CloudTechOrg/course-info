<details>
  <summary> 【ハンズオン】VPCエンドポイント-デモ / VPC内部DNSの動きについて補足
</summary>

**コマンドメモ**

- **S3のバケットリストを表示**

  ```bash
  aws s3 ls --region ap-northeast-1
  nslookup s3.ap-northeast-1.amazonaws.com
  ```

- **CloudWatch Logsのロググループを表示**

  ```bash
  aws logs describe-log-groups --region ap-northeast-1
  ```

  *補足：*
  
  EC2インスタンスには以下のポリシーを持つIAMロールがアタッチされています。

  - `AmazonS3ReadOnlyAccess`
  - `CloudWatchLogsFullAccess`

  上記ポリシーがない場合、AWS CLIのコマンド実行時に権限エラーとなるのでご注意ください。

**S3バケットの概要や作成方法について**

順番が前後しますが、S3バケット作成の手順は以下のデモのチャプター「S3バケットの作成とオブジェクトアップロード」をご覧ください。

[【ハンズオン】CloudFrontとS3のOAC機能デモ](https://kws-cloud-tech.com/topic/%E3%80%90%E3%83%8F%E3%83%B3%E3%82%BA%E3%82%AA%E3%83%B3%E3%80%91cloudfront%E3%81%A8s3%E3%81%AEoac%E6%A9%9F%E8%83%BD%E3%83%87%E3%83%A2)

</details>

---

<details>
  <summary> 【ハンズオン】CloudFrontとS3のOAC機能デモ
</summary>

```html
<!DOCTYPE html>
<html>
  <head>
    <title>S3 test</title>
  </head>
  <body>
    <h1>S3 test</h1>
    <p>test html</p>
  </body>
</html>
```
</details>

---

<details>
  <summary> 【ハンズオン】独自ドメインとHTTPSによるWebサイトのホスティング
</summary>

**index.html**

```html
<!DOCTYPE html>
<html lang="jp">
  <head>
    <title>CloudTech Handson</title>
  </head>
  <body>
    <h1>EC2ハンズオン</h1>
  </body>
</html>
```

**EC2にセッションマネージャーでログインを行い、以下のコマンドを実行**

- **Nginxのインストール**

  ```bash
  sudo dnf install nginx
  ```

- **Nginxの開始とシステム再起動時に自動的に開始されるように設定**

  ```bash
  sudo systemctl start nginx
  sudo systemctl enable nginx
  ```

- **`/usr/share/nginx/html` ディレクトリに移動**

  ```bash
  cd /usr/share/nginx/html
  ```

- **`index.html` ファイルを作成**

  ```bash
  sudo vi index.html
  ```
</details>