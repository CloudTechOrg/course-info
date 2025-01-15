<details>
  <summary> 【ハンズオン】IAMポリシーをグループにアタッチして表示の変化を確認する
</summary>
以下の許可設定をJSON形式のテキストに追加しましょう。

```json
"s3:ListBucket"
```
</details>

---

<details>
  <summary> 【ハンズオン】Aurora作成-接続/手動フェイルオーバーによる動作確認/レプリカオートスケーリング/クローン取得
</summary>

#### EC2セットアップ手順

1. Amazon Linux 2023の場合
   - `sudo dnf update -y`
   - `sudo dnf install mariadb105`

2. Amazon Linux 2の場合
   - `sudo yum update -y`
   - `sudo amazon-linux-extras install mariadb10.5`

#### EC2からAurora接続

```bash
mysql -h endpoint -P 3306 -u admin -p
```

#### データベースにサンプルデータ投入

```sql
CREATE DATABASE db;
SHOW DATABASES;
USE db;

CREATE TABLE db (
  column1 INT,
  column2 INT,
  column3 INT
) CHARSET=utf8;

INSERT INTO db VALUES (1,2,3);
INSERT INTO db VALUES (10,20,30);

SELECT * FROM db;
```

#### 手動フェイルオーバーコマンド

```bash
aws rds failover-db-cluster --db-cluster-identifier mydatabase
```
</details>

