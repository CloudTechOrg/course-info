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

プロセス環境変数の設定
```powershell
$Env:AWS_ACCESS_KEY_ID="アクセスキー"
$Env:AWS_SECRET_ACCESS_KEY="シークレットアクセスキー"
```

VPCとサブネットの作成
```terraform
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

ローカル変数app_nameを追加
```terraform
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


入力変数envを追加
```terraform
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

ローカル変数 name_prefix = "${var.env}-${local.app_name} を追加
```
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


webサーバ用のmain.tf
```
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


