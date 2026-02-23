<details>
  <summary> 【Terraform-3】インストール（Mac/Win）
  </summary> 

#### Mac向けコマンド
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform --version
```

#### Windows向けコマンド
```
Terraformダウンロードページ
https://developer.hashicorp.com/terraform/install?product_intent=terraform
```

</details>

---

<details>
  <summary> 【Terraform-5】ハンズオン-VPC,サブネットの作成【20:27】
  </summary> 

#### プロバイダブロックの作成
```hcl
provider "aws" {
 region = "ap-northeast-1"
}
```

#### アクセスキー設定
```ini
■Windows向け
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"

■Mac向け
export AWS_ACCESS_KEY_ID=your_access_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_access_key
```

#### VPCのresourceブロックの作成
```hcl
resource "aws_vpc" "terra_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "aws_vpc_name"
  }
}
```

#### VPCの名前の変更
```hcl
resource "aws_vpc" "terra_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "aws_vpc_handson"
  }
}
```

#### サブネットのリソース追加
```hcl
resource "aws_subnet" "terra_subnet" {

  vpc_id = aws_vpc.terra_vpc.id

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "aws_subnet_name"
  }
}
```
</details>

---

<details>
  <summary> 【Terraform-7】ハンズオン-Webサーバーの作成（locals,variablesブロック）【13:41】
  </summary> 

#### アクセスキー設定
```ini
■Windows向け
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"

■Mac向け
export AWS_ACCESS_KEY_ID=your_access_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_access_key
```

#### VPCリソースの作成（準備）
```hcl
provider "aws" {
 region = "ap-northeast-1"
}

resource "aws_vpc" "web_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "vpc"
  }
}

resource "aws_subnet" "web_subnet" {

  vpc_id = aws_vpc.web_vpc.id

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "subnet"
  }
}

```

#### ローカル変数app_nameを追加
```hcl
provider "aws" {
 region = "ap-northeast-1"
}

# 追加
locals{
    app_name = "web"
}

resource "aws_vpc" "web_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${local.app_name}-vpc" # 変更
  }
}

resource "aws_subnet" "web_subnet" {

  vpc_id = aws_vpc.web_vpc.id

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "${local.app_name}-subnet" # 変更
  }
}
```

#### 入力変数envを追加
```hcl
provider "aws" {
 region = "ap-northeast-1"
}

# 追加
variable "env" {
    type = string
    default = "handson"
}

locals{
    app_name = "web"
}

resource "aws_vpc" "web_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${var.env}-${local.app_name}-vpc" # 変更
  }
}

resource "aws_subnet" "web_subnet" {

  vpc_id = aws_vpc.web_vpc.id

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "${var.env}-${local.app_name}-subnet" # 変更
  }
}
```

#### ローカル変数 name_prefix = "${var.env}-${local.app_name} を追加
```hcl
provider "aws" {
 region = "ap-northeast-1"
}

variable "env" {
    type = string
    default = "prod"
}

locals{
    app_name = "handson-web" # 追加
    name_prefix = "${var.env}-${local.app_name}"
}

resource "aws_vpc" "web_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${local.name_prefix}-vpc" # 変更
  }
}

resource "aws_subnet" "web_subnet" {

  vpc_id = aws_vpc.web_vpc.id

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "${local.name_prefix}-public_subnet" # 変更
  }
}
```

#### webサーバ用のmain.tf
```hcl
provider "aws" {
 region = "ap-northeast-1"
}

variable "env" {
    type = string
    default = "handson"
}

variable "myip" {
    type = string
    description = "Check-> https://www.whatismyip.com/"
}

locals{
    app_name = "web"
    name_prefix = "${var.env}-${local.app_name}"
}

resource "aws_vpc" "web_vpc" {

  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${local.name_prefix}-vpc"
  }
}

resource "aws_subnet" "web_subnet" {

  vpc_id = aws_vpc.web_vpc.id
  map_public_ip_on_launch = true

  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "${local.name_prefix}-public_subnet"
  }
}

resource "aws_route_table" "web_public_rtb" {
  vpc_id = aws_vpc.web_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.web_igw.id
  }

  tags = {
    Name = "${local.name_prefix}-public-rtb"
  }
}

resource "aws_route_table_association" "web_public_rtb_assoc" {
  subnet_id      = aws_subnet.web_subnet.id
  route_table_id = aws_route_table.web_public_rtb.id
}

resource "aws_internet_gateway" "web_igw" {
  vpc_id = aws_vpc.web_vpc.id

  tags = {
    Name = "${local.name_prefix}-igw"
  }
}

resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.web_vpc.id

  name        = "${local.name_prefix}-sg"
  description = "Allow HTTP access from my IP"

  ingress {
    description = "Allow HTTP traffic from my IP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["${var.myip}/32"] # var.myipからのHTTPアクセスを許可
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${local.name_prefix}-sg"
  }
}

resource "aws_instance" "web_ec2" {
  ami                         = "ami-094dc5cf74289dfbc" 
  instance_type               = "t2.micro"
  security_groups             = [aws_security_group.web_sg.id]
  subnet_id = aws_subnet.web_subnet.id

  user_data = <<-EOF
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable --now nginx
cat <<HTML > /usr/share/nginx/html/index.html
    <div style="text-align:center; font-size:1.5em; color:#333; margin:20px; line-height:1.8;">
        <b>env: ${var.env}</b><br>
        <b>app_name: ${local.app_name}</b><br>
        <b>name_prefix: ${local.name_prefix}</b><br>
        <b>myip: ${var.myip}</b>
    </div>
HTML
  EOF

  tags = {
    Name = "${local.name_prefix}-ec2"
  }
}
```
</details>

---

<details>
  <summary> 【Terraform-9】ハンズオン-ローカルモジュールを活用した複数環境の作成
  </summary> 

#### プロセス環境変数の設定
```powershell
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"
```

#### webモジュール用(main.tf)
```terraform
provider "aws" {
  region = "ap-northeast-1"
}

locals {
  app_name = "web"
  name_prefix = "${var.env}-${local.app_name}"
}

resource "aws_vpc" "web_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${local.name_prefix}-vpc"
  }
}

resource "aws_subnet" "web_subnet" {
  vpc_id = aws_vpc.web_vpc.id
  map_public_ip_on_launch = true
  cidr_block = "10.0.0.0/24"
  tags = {
    Name = "${local.name_prefix}-public_subnet"
  }
}

resource "aws_route_table" "web_public_rtb" {
  vpc_id = aws_vpc.web_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.web_igw.id
  }

  tags = {
    Name = "${local.name_prefix}-public-rtb"
  }
}

resource "aws_route_table_association" "web_public_rtb_assoc" {
  subnet_id      = aws_subnet.web_subnet.id
  route_table_id = aws_route_table.web_public_rtb.id
}

resource "aws_internet_gateway" "web_igw" {
  vpc_id = aws_vpc.web_vpc.id
  tags = {
    Name = "${local.name_prefix}-igw"
  }
}

resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.web_vpc.id
  name        = "${local.name_prefix}-sg"
  description = "Allow HTTP access from my IP"

  ingress {
    description = "Allow HTTP traffic from my IP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["${var.myip}/32"] # var.myipからのHTTPアクセスを許可
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${local.name_prefix}-sg"
  }
}

resource "aws_instance" "web_ec2" {
  ami                         = "ami-094dc5cf74289dfbc" 
  instance_type               = "t2.micro"
  security_groups             = [aws_security_group.web_sg.id]
  subnet_id = aws_subnet.web_subnet.id

  user_data = <<-EOF
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable --now nginx
cat <<EOG > /usr/share/nginx/html/index.html
    <div style="text-align:center; font-size:1.5em; color:#333; margin:20px; line-height:1.8;">
        <b>env: ${var.env}</b><br>
        <b>app_name: ${local.app_name}</b><br>
        <b>name_prefix: ${local.name_prefix}</b><br>
        <b>myip: ${var.myip}</b>
    </div>
EOG
EOF

  tags = {
    Name = "${local.name_prefix}-ec2"
  }
}
```

#### webモジュール用(variables.tf)
```terraform
variable "env" {
  type = string
}

variable "myip" {
  type = string
}
```

#### webモジュール用(output.tf)
```terraform
output "web_ec2_public_ip" {
  value = aws_instance.web_ec2.public_ip
}
```

#### ルートモジュール用(main.tf)
```terraform
module "web" {
  source = "../../modules/web"
  myip = var.myip
  env = var.env
}
```

#### グローバルIPアドレス確認用
```
https://www.whatismyip.com/
```

#### ルートモジュール用(terraform.tfvars)
```terraform
myip = "自分のIPアドレス"
env = "dev"
```

#### ルートモジュール用(output.tf)
```terraform
output "web_addr" {
  value = "http://${module.web.web_ec2_public_ip}"
}
```

</details>

---

<details>
  <summary> 【Terraform-11】ハンズオン-既存リソースの利用(importブロック, dataブロック)
  </summary> 


プロセス環境変数の設定
```powershell
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"
```

インポート用のmain.tf
```terraform
provider "aws" {
  region = "ap-northeast-1"
}

import {
  id = "【vpcid】"
  to = aws_vpc.imported_vpc
}
```


修正前generated.tf
```terraform
# __generated__ by Terraform
# Please review these resources and move them into your main configuration files.
# __generated__ by Terraform
resource "aws_vpc" "imported_vpc" {
  assign_generated_ipv6_cidr_block     = false
  cidr_block                           = "10.0.0.0/24"
  enable_dns_hostnames                 = false
  enable_dns_support                   = true
  enable_network_address_usage_metrics = false
  instance_tenancy                     = "default"
  ipv4_ipam_pool_id                    = null
  ipv4_netmask_length                  = null
  ipv6_cidr_block                      = null
  ipv6_cidr_block_network_border_group = null
  ipv6_ipam_pool_id                    = null
  ipv6_netmask_length                  = 0
  tags = {
    Name = "handson-vpc"
  }
  tags_all = {
    Name = "handson-vpc"
  }
}
```

修正後(generated.tf)
```terraform
resource "aws_vpc" "imported_vpc" {
  cidr_block           = "10.0.0.0/24"
  
  tags = {
    Name = "handson-vpc"
  }
}
```

dataブロックハンズオン用main.tf
```terraform
provider "aws" {
  region = "ap-northeast-1"
}

data "aws_vpc" "existing_vpc" {
  filter {
    name   = "tag:Name"
    values = ["handson-vpc"]
  }
}

resource "aws_subnet" "data_subnet" {

  vpc_id = data.aws_vpc.existing_vpc.id
  cidr_block             = "10.0.0.0/24"

  tags = {
    Name = "handson-subnet"
  }
}
```


</details>

---

<details>
  <summary> 【Terraform-13】ハンズオン-リモートバックエンドの設定(terraformブロック)
  </summary> 


プロセス環境変数の設定
```powershell
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"
```

リモートバックエンドの設定を含んだmain.tf
```terraform
provider "aws" {
  region = "ap-northeast-1"
}

terraform {
  backend "s3" {
    bucket        = "【作成したS3バケットの名前】"
    key           = "test/terraform.tfstate"
    region        = "ap-northeast-1"
    use_lockfile  = true
  }
}

動作確認用
```terraform
resource "aws_vpc" "remote_state_test_vpc" {
  cidr_block           = "10.0.0.0/24"
  tags = {
    Name = "remote_state_test_vpc"
  }
}
```


</details>

---


<details>
  <summary> 【CloudFormation】CloudFormationでVPCを作成する
  </summary> 

#### cloudformation.yml
```yml
Resources:
  MyVPC2:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.8.0/21"
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: "MyVPCfromCF"
```
</details>

---

<details>
  <summary> 
【CloudFormation】CloudFormationパラメーター
</summary> 

#### cloudformation.yml
```yml
AWSTemplateFormatVersion: 2010-09-09

Parameters:
  VPCCIDR:
    Description: CIDR Block for VPC 
    Type: String
    Default: 10.0.16.0/21
  Name:
    Description: Tags Name for VPC 
    Type: String
    Default: MyVPC3fromCF

Resources: 
  MyVPC3:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VPCCIDR
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Ref Name
```
</details>

---

<details>
  <summary> 
【CloudFormation】CloudFormationテンプレート更新/Ref関数
</summary> 

#### cloudformation.yml
```yml
Resources:
  MyVPC2:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.8.0/21"
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: "MyVPCfromCF"
  subnetName:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: "ap-northeast-1a"
      VpcId: !Ref MyVPC2
      CidrBlock: "10.0.8.0/24"
      Tags:
        - Key: Name
          Value: "subnet1fromCF"
  secGroupName:
    Type: "AWS::EC2::SecurityGroup"
    Properties: 
      GroupName: "GroupName-SG"
      GroupDescription: "GroupDescription-SG"
      VpcId: !Ref MyVPC2
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: "0.0.0.0/0"
      Tags:
        - Key: Name
          Value: "SGfromCF"
```
</details>

---

<details>
  <summary> 
【CloudFormation】クロススタック参照/Outputsセクション/ImportValue関数/GetAtt関数
</summary> 

#### cloudformation.yml
```yml
Resources:
  MyVPC2:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.8.0/21"
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: "MyVPCfromCF"

  subnetName:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: "ap-northeast-1a"
      VpcId: !Ref MyVPC2
      CidrBlock: "10.0.8.0/24"
      Tags:
        - Key: Name
          Value: "subnet1fromCF"

  secGroupName:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: "GroupName-SG"
      GroupDescription: "GroupDescription-SG"
      VpcId: !Ref MyVPC2
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: "0.0.0.0/0"
      Tags:
        - Key: Name
          Value: "SGfromCF"

Outputs:
  Subnet1:
    Value: !Ref subnetName
    Export:
      Name: Subnet1Name

  SG1:
    Value: !Ref secGroupName
    Export:
      Name: SG1Name

AWSTemplateFormatVersion: "2010-09-09"
Resources:
  myEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      KeyName: Mykeypair
      ImageId: インスタンスID
      InstanceType: t2.micro
      Monitoring: false
      SecurityGroupIds:
        - !ImportValue SG1Name
      SubnetId: !ImportValue Subnet1Name
      Tags:
        - Key: Name
          Value: CFec2
```

#### ec2.yml
```yml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  myEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      KeyName: Mykeypair
      ImageId: インスタンスID
      InstanceType: t2.micro
      Monitoring: false
      SecurityGroupIds:
        - !ImportValue SG1Name
      SubnetId: !ImportValue Subnet1Name
      Tags:
        - Key: Name
          Value: CFec2
```
</details>

---

<details>
  <summary> 
「VisualStudioCode RemoteSSH拡張機能」でEC2インスタンスのファイルを便利に編集する方法
</summary> 

#### .ssh/configの記載例（IPアドレスとキーペアのパスはご自身の環境に合わせてください。）

```bash
Host WebServer1
  HostName 52.192.16.252
  IdentityFile /Users/blackriver/AWS/keypair01.pem
  User ec2-user
```

</details>

---

<details>
  <summary> 
【CloudFormation】AWS CLIを使ったスタックの基本操作/aws cloudformation deployコマンド
</summary> 

#### stack_s3_param.yml
```yml
AWSTemplateFormatVersion: "2010-09-09"
Description: CloudTechDemoS3

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: cloudtechsamplebucket
```

#### コマンド
```bash
aws cloudformation deploy --template-file stack_s3_param.yml --stack-name s3bucketcreate

aws cloudformation delete-stack --stack-name s3bucketcreate
```
</details>

---

<details>
  <summary> 
【CloudFormation】AWS CLIでパラメーター値を指定する/外部設定ファイルからパラメーターを読み込む
</summary> 

#### stack_s3.yml
```yml
AWSTemplateFormatVersion: "2010-09-09"

Description: CloudTechDemoS3

Parameters:
  S3BucketName:
    Type: String
    Description: Type of this BacketName.

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub ${S3BucketName}
```

#### s3config.cfg
```ini
S3BucketName=samplecloudtechbucket2
```

#### コマンド
```bash
aws cloudformation deploy --template-file stack_s3.yml --stack-name s3bucketcreate --parameter-overrides S3BucketName=samplecloudtechbucket

aws cloudformation deploy --template-file stack_s3.yml --stack-name s3bucketcreate --parameter-overrides $(cat s3config.cfg)

aws cloudformation describe-stack-resource --stack-name s3bucketcreate --logical-resource-id S3Bucket

aws cloudformation delete-stack --stack-name s3bucketcreate
```
</details>

---

<details>
  <summary> 
【CloudFormation】RDSを作成する/外部設定ファイルからパラメーターを読み込む
</summary> 

#### Stack.yml （*AWSアップデートに合わせて以下箇所を動画と変更しております。ご了承ください。

>EngineVersion: 8.0<br>
>DBInstanceClass: db.t3.micro）

```yml
AWSTemplateFormatVersion: "2010-09-09"

Description: CloudTechDemo

Parameters:
  DatabasePassword:
    Type: String
    Description: Database password
    NoEcho: "true"
  ApplicationSubnets:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Target subnets
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: Target VPC
  DBinboundCidrIPs:
    Type: String
    Description: SecurityGroupInboundIP

Resources:
  ApplicationDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: MySQL
      EngineVersion: 8.0
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 10
      StorageType: gp2
      MasterUsername: CloudTech
      MasterUserPassword:
        Ref: DatabasePassword
      DBName: CloudTech
      VPCSecurityGroups:
        - !Ref ApplicationDatabaseSecurityGroup
      DBSubnetGroupName: !Ref ApplicationDatabaseSubnetGroup
      MultiAZ: "false"
      AvailabilityZone: !Sub ${AWS::Region}a
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-db
  ApplicationDatabaseSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Application Database Subnet Group
      SubnetIds: !Ref ApplicationSubnets
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-db-subnet-group
  ApplicationDatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Sub ${AWS::StackName} Application Database Security Group
      VpcId: !Ref VpcId
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 3306
        ToPort: 3306
        CidrIp: !Ref DBinboundCidrIPs
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-db-sg
```

#### dev.cfg（各自の環境に合わせて置き換えてください）
```ini
DatabasePassword=Thisispassword123!#$%
ApplicationSubnets=subnet-0791ad96ce7a109ea,subnet-09185025781affa32
VpcId=vpc-09f13cc71120d5cca
DBinboundCidrIPs=172.31.16.0/20
```
#### 実行コマンド
```bash
aws cloudformation deploy –template-file stack.yaml –stack-name RDSmySQLcreate –parameter-overrides $(cat dev.cfg)
```
</details>

---

<details>
  <summary> 
【CDKハンズオン】環境セットアップ/Node.js・AWS CLI・CDKインストール/cdk init・bootstrap/VPC定義/cdk synth・deploy【32:01】
</summary>

#### 参考文献リンク
- Node.js 公式: https://nodejs.org/
- AWS CLI インストールガイド: https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html
- aws loginコマンド: https://aws.amazon.com/jp/blogs/news/simplified-developer-access-to-aws-with-aws-login/
- AWS CDK ガイド (Getting Started): https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/getting-started.html
- CDK API Reference (v2): https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html
- VPC クラス API Reference: https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.Vpc.html

#### Node.jsのインストール
```bash
# nvmをダウンロードしてインストールする：
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# シェルを再起動する代わりに実行する
\. "$HOME/.nvm/nvm.sh"

# Node.jsをダウンロードしてインストールする：
nvm install 24

# Node.jsのバージョンを確認する：
node -v # "v24.13.0"が表示される。

# npmのバージョンを確認する：
npm -v # "11.6.2"が表示される。
```

#### AWS CLIのバージョン確認
```bash
aws --version
# バージョンが2.32.0以上であること(そうでないとaws login コマンドが利用できないため)
```

#### AWS 認証設定
```bash
aws login

# 東京リージョンを指定
ap-northeast-1
```

#### 別ターミナルでもNode.jsを利用できるようにする設定（必要な場合）
```bash
# ファイルの作成
touch ~/.zshrc

# .zshrcに追記
vi ~/.zshrc

# 追記内容
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

# 設定の反映
source ~/.zshrc
```

#### AWS CDK インストール
```bash
npm install -g aws-cdk

# バージョン確認
cdk --version
```

#### CDKテンプレートの作成
```bash
# テンプレートを展開するディレクトリの作成
mkdir cdk-handson

# 作成したディレクトリに移動　※作成したディレクトリ内にファイルやディレクトリを作成しないこと
cd cdk-handson

# テンプレートの作成  言語Typescriptを指定
cdk init app --language typescript
```

#### CDKで展開するための準備
```bash
cdk bootstrap 
```

#### テンプレートの生成
```bash
cdk synth
```

#### AWS環境へデプロイ
```bash
cdk deploy
```

#### 最終的な完成コード（cdk-handson-stack.ts）
```typescript
import * as cdk from 'aws-cdk-lib/core';
import { Construct } from 'constructs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as rds from 'aws-cdk-lib/aws-rds';

//ファイルを読み込むライブラリのインポート
import { readFileSync } from 'fs';

export class CdkHandsonStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    //VPCの作成
    const vpc = new ec2.Vpc(this, 'vpc', {
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/20'),
      maxAzs: 2,
      natGateways: 0,
    });

    //ECの作成
    const webserver01 = new ec2.Instance(this, 'webserver01', {
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023(),
      vpc: vpc,
      vpcSubnets: {
        availabilityZones: [vpc.availabilityZones[0]],
        subnetType: ec2.SubnetType.PUBLIC
      }
    });

    //User Dataの作成
    const script = readFileSync("./lib/src/user-data.sh", "utf-8")
    webserver01.addUserData(script)

    //セキュリティグループの作成
    webserver01.connections.allowFromAnyIpv4(ec2.Port.tcp(80))

    //RDSの作成
    const dbserver = new rds.DatabaseInstance(this, 'dbserver', {
     engine: rds.DatabaseInstanceEngine.mysql({version: rds.MysqlEngineVersion.VER_8_4_5}),
     vpc: vpc,
     vpcSubnets: {subnetType: ec2.SubnetType.PRIVATE_ISOLATED},
     availabilityZone: vpc.availabilityZones[0],
     instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
     allocatedStorage: 20,
     databaseName: 'handsonwordpress',
   });

   // セキュリティグループを設定
   dbserver.connections.allowFrom(webserver01, ec2.Port.tcp(3306))
  }
}
```

</details>

---

<details>
  <summary> 
【CDKハンズオン】EC2構築/インスタンスタイプ・AMI設定/VPCサブネット指定/ユーザーデータ/セキュリティグループ/cdk synth・deploy【23:57】
</summary>

#### 参考文献リンク
- AWS CDK ガイド (Getting Started): https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/getting-started.html
- CDK API Reference (v2): https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html
- EC2 クラス API Reference: https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.Instance.html

#### user-data.sh の中身
```bash
#!/bin/bash
dnf -y update
dnf -y install php php-mysqlnd php-mbstring php-xml httpd mariadb105
wget http://ja.wordpress.org/latest-ja.tar.gz -P /tmp/
tar zxvf /tmp/latest-ja.tar.gz -C /tmp
cp -r /tmp/wordpress/* /var/www/html/
chown apache:apache -R /var/www/html
systemctl enable httpd.service
systemctl start httpd.service
```

#### 利用コマンド

テンプレートの生成
```bash
cdk synth
```

AWS環境との差分確認
```bash
cdk diff
```

AWS環境へデプロイ
```bash
cdk deploy
```

#### 最終的な完成コード（cdk-handson-stack.ts）
```typescript
import * as cdk from 'aws-cdk-lib/core';
import { Construct } from 'constructs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as rds from 'aws-cdk-lib/aws-rds';

//ファイルを読み込むライブラリのインポート
import { readFileSync } from 'fs';

export class CdkHandsonStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    //VPCの作成
    const vpc = new ec2.Vpc(this, 'vpc', {
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/20'),
      maxAzs: 2,
      natGateways: 0,
    });

    //ECの作成
    const webserver01 = new ec2.Instance(this, 'webserver01', {
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023(),
      vpc: vpc,
      vpcSubnets: {
        availabilityZones: [vpc.availabilityZones[0]],
        subnetType: ec2.SubnetType.PUBLIC
      }
    });

    //User Dataの作成
    const script = readFileSync("./lib/src/user-data.sh", "utf-8")
    webserver01.addUserData(script)

    //セキュリティグループの作成
    webserver01.connections.allowFromAnyIpv4(ec2.Port.tcp(80))

    //RDSの作成
    const dbserver = new rds.DatabaseInstance(this, 'dbserver', {
     engine: rds.DatabaseInstanceEngine.mysql({version: rds.MysqlEngineVersion.VER_8_4_5}),
     vpc: vpc,
     vpcSubnets: {subnetType: ec2.SubnetType.PRIVATE_ISOLATED},
     availabilityZone: vpc.availabilityZones[0],
     instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
     allocatedStorage: 20,
     databaseName: 'handsonwordpress',
   });

   // セキュリティグループを設定
   dbserver.connections.allowFrom(webserver01, ec2.Port.tcp(3306))
  }
}
```

</details>

---

<details>
  <summary> 
【CDKハンズオン】RDS構築/MySQL・サブネットグループ設定/セキュリティグループ/WordPress初期設定・動作確認/cdk destroy【31:14】
</summary>

#### 参考文献リンク
- AWS CDK ガイド (Getting Started): https://docs.aws.amazon.com/ja_jp/cdk/v2/guide/getting-started.html
- CDK API Reference (v2): https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html
- RDS モジュール API Reference: https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_rds-readme.html
- RDS インスタンス API Reference: https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_rds.DatabaseInstance.html

#### 利用コマンド

テンプレートの生成
```bash
cdk synth
```

AWS環境との差分確認
```bash
cdk diff
```

AWS環境へデプロイ
```bash
cdk deploy
```

リソース削除
```bash
cdk destroy
```

#### 最終的な完成コード（cdk-handson-stack.ts）
```typescript
import * as cdk from 'aws-cdk-lib/core';
import { Construct } from 'constructs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as rds from 'aws-cdk-lib/aws-rds';

//ファイルを読み込むライブラリのインポート
import { readFileSync } from 'fs';

export class CdkHandsonStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    //VPCの作成
    const vpc = new ec2.Vpc(this, 'vpc', {
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/20'),
      maxAzs: 2,
      natGateways: 0,
    });

    //ECの作成
    const webserver01 = new ec2.Instance(this, 'webserver01', {
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023(),
      vpc: vpc,
      vpcSubnets: {
        availabilityZones: [vpc.availabilityZones[0]],
        subnetType: ec2.SubnetType.PUBLIC
      }
    });

    //User Dataの作成
    const script = readFileSync("./lib/src/user-data.sh", "utf-8")
    webserver01.addUserData(script)

    //セキュリティグループの作成
    webserver01.connections.allowFromAnyIpv4(ec2.Port.tcp(80))

    //RDSの作成
    const dbserver = new rds.DatabaseInstance(this, 'dbserver', {
     engine: rds.DatabaseInstanceEngine.mysql({version: rds.MysqlEngineVersion.VER_8_4_5}),
     vpc: vpc,
     vpcSubnets: {subnetType: ec2.SubnetType.PRIVATE_ISOLATED},
     availabilityZone: vpc.availabilityZones[0],
     instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
     allocatedStorage: 20,
     databaseName: 'handsonwordpress',
   });

   // セキュリティグループを設定
   dbserver.connections.allowFrom(webserver01, ec2.Port.tcp(3306))
  }
}
```

</details>
