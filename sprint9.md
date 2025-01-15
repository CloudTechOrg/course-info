<details>
  <summary> 
【ハンズオン】Amazon BedrockでRAG構築ハンズオン
</summary> 

#### コマンド
```bash
sudo yum update -y
sudo yum install -y python3-pip
pip install boto3
pip install boto3==1.34.87 langchain==0.2.0 langchain-aws==0.1.4 langchain-community==0.2.0 streamlit==1.33.0 python-dateutil==2.8.2
vi app.py
streamlit run app.py
```

#### app.py

```python
# 必要なライブラリをインポート
import boto3
import streamlit as st

# ページ設定
st.set_page_config(
    page_title="Bedrockサンプルアプリ",
    page_icon=":smile:",
    layout="centered",
    initial_sidebar_state="collapsed",
)

# カスタムCSSを追加
st.markdown(
    """
    <style>
    .main {
        background-color: #f5f5f5;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    }
    h1 {
        color: #0073e6;
    }
    .stButton>button {
        background-color: #0073e6;
        color: white;
        border-radius: 8px;
        padding: 10px;
        font-size: 16px;
    }
    .stTextInput>div>input {
        border: 2px solid #0073e6;
        border-radius: 8px;
        padding: 10px;
    }
    </style>
    """,
    unsafe_allow_html=True,
)

# タイトルと説明
st.title("Bedrockサンプルアプリ:smile:")
st.write("ナレッジベースに質問して、AIからの回答を受け取りましょう。")

# ユーザー入力
question = st.text_input("質問を入力してください", placeholder="例: AWSとは何ですか？")

# ボタンが押されたら処理を実行
if st.button("質問する") and question:

    # Bedrockクライアントを作成
    kb = boto3.client("bedrock-agent-runtime")

    # ナレッジベースを呼び出し
    with st.spinner("回答を生成中..."):
        response = kb.retrieve_and_generate(
            input={"text": question},
            retrieveAndGenerateConfiguration={
                "type": "KNOWLEDGE_BASE",
                "knowledgeBaseConfiguration": {
                    "knowledgeBaseId": "DYFUHCOVWZ",  # ナレッジベースID
                    "modelArn": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0",
                },
            },
        )

    # 結果を表示
    st.success("回答が見つかりました！")
    st.write(response["output"]["text"])

# フッター
st.markdown(
    """
    <hr>
    <footer style="text-align: center;">
        <p>© 2024 CloudTech</p>
    </footer>
    """,
    unsafe_allow_html=True,
)
```
</details>

---

#### 【AWS CloudShell】機能解説

```bash
aws s3api create-bucket –bucket ここにバケットネームを記載する –region ap-northeast-1 –create-bucket-configuration LocationConstraint=ap-northeast-1

aws s3api put-object –bucket ここにバケットネームを記載する –key ここにS3に保存する名前を記載する –body ここにS3にアップロードするファイルを記載する
```

<details>
  <summary> 
【ハンズオン】CloudWatchエージェントインストールのハンスオン
</summary> 

#### ■CloudWatch Agentに対するコマンド
```bash
・インストールコマンド
sudo yum install -y amazon-cloudwatch-agent

・起動確認コマンド
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status

・対話型ウィザード起動コマンド
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

・起動コマンド
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl 
    -a fetch-config 
    -m ec2 
    -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json 
    -s
```

```json
{
    "agent": {
        "metrics_collection_interval": 60,
        "run_as_user": "root"
    },
    "metrics": {
        "append_dimensions": {
            "InstanceId": "${aws:InstanceId}"
        },
        "metrics_collected": {
            "mem": {
                "measurement": [
                    "mem_used_percent",
                    "mem_available_percent"
                ],
                "metrics_collection_interval": 60
            },
            "disk": {
                "measurement": [
                    "used_percent"
                ],
                "metrics_collection_interval": 60,
                "resources": [
                    "/",
                    "/tmp"
                ]
            }
        }
    }
}
```
</details>

<details>
  <summary> 
【ハンズオン】Systems Manager（SSM）パラメーターストア/デモ(CloudWatch Agentインストール)
</summary> 

<details>
  <summary> CloudFormationテンプレートコード（クリックして展開）
  </summary>

```yml
AWSTemplateFormatVersion: "2010-09-09"
Description: Parameter Store Demo

#======================
# パラメーター
#======================
Parameters:

# VPCのCIDRレンジ
  VpcCIDR:
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String
    Default: 172.31.0.0/16

# プライベートサブネット1
  PrivateSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
    Type: String
    Default: 172.31.1.0/24

# パブリックサブネット1
  PublicSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
    Type: String
    Default: 172.31.2.0/24

# 環境変数
  EnvironmentName:
    Description: Evname-tag
    Type: String
    Default: TEST

  Ec2ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

  Ec2InstanceType:
    Type: String
    Default: t2.micro

#======================
# リソース
#======================

Resources:

#======================
##### VPCなどのリソースを作成 #####
#======================
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName
  
  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    Properties:
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
      
# パブリックサブネット1を作成
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      CidrBlock: !Ref PublicSubnet1CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ1)

  # ルートテーブル（パブリックサブネット用）作成
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Routes
  
  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  # Internet Gatewayへのルート作成
  PublicRoute:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway


# プライベートサブネット1を作成
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnet1CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Subnet (AZ1)

# ルートテーブル（プライベートサブネット用）作成
  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Routes

  PrivateSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      SubnetId: !Ref PrivateSubnet1

  # NAT Gatewayに関連づけるElastic IP を作成
  NatGatewayEIP:
    Type: 'AWS::EC2::EIP'
    Properties:
      Domain: 'vpc'

  # NAT Gatewayを作成
  NatGateway:
    Type: 'AWS::EC2::NatGateway'
    Properties:
      AllocationId: !GetAtt NatGatewayEIP.AllocationId
      SubnetId: !Ref PublicSubnet1

  # NAT Gatewayへのルート作成
  PrivateSubnetRoute:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NatGateway

#======================
##### IAMロール作成 #####
#======================
  EC2IAMRole:
    Type: AWS::IAM::Role
    Properties:
      Path: /
      RoleName: !Sub ${EnvironmentName}-EC2IAMRole
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
  
  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - Ref: EC2IAMRole
      InstanceProfileName: !Sub ${EnvironmentName}-EC2InstanceProfile


#======================
##### EC2作成 #####
#======================
  DemoInstance:
    Type: 'AWS::EC2::Instance'
    Properties: 
      ImageId: !Ref Ec2ImageId
      InstanceType: !Ref Ec2InstanceType
      IamInstanceProfile: !Ref EC2InstanceProfile
      AvailabilityZone: !GetAtt   PrivateSubnet1.AvailabilityZone
      NetworkInterfaces:
        - DeviceIndex: 0
          AssociatePublicIpAddress: false
          DeleteOnTermination: true
          SubnetId: !Ref PrivateSubnet1
          GroupSet: 
            - !Ref DemoSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} EC2        

  DemoSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      VpcId: !Ref VPC
      GroupDescription: SG for EC2
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} SG
```

</details>

#### CloudWatch エージェント設定コード
```json
{
	"agent": {
		"metrics_collection_interval": 60,
		"run_as_user": "root"
	},
	"metrics": {
		"metrics_collected": {
			"disk": {
				"measurement": [
					"used_percent"
				],
				"metrics_collection_interval": 60,
				"resources": [
					"*"
				]
			},
			"mem": {
				"measurement": [
					"mem_used_percent"
				],
				"metrics_collection_interval": 60
			}
		}
	}
}
```

</details>


<details>
  <summary> 
【ハンズオン】SQS標準キュー/FIFOキュー/DLQの動作確認
</summary> 


■SQSからポーリングしてきたメッセージをログに出力するシンプルな関数
```python
import json

def lambda_handler(event, context):
    # TODO implement
    for record in event['Records']:
        print(f"Message Body: {record['body']}")


    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

■キューに数字が入力されて偶数の場合にエラーを投げる。奇数の場合は正常に動作を行う。

```python
import json

def lambda_handler(event, context):
    for record in event['Records']:
        body = record['body']
        try:
            number = int(body)  
            
            if number % 2 == 0:
                raise ValueError(f'Error: {number} is an even number')
            else:
                print(f"Message Body: {body}")
        except ValueError as e:
            print(e)
            raise  
    return {
        'statusCode': 200,
        'body': json.dumps('Processed successfully')
    }
```

■コマンド
```bash
aws sqs send-message-batch --queue-url https://sqs.ap-northeast-1.amazonaws.com/339712847759/hands-on-queue --entries "$(seq 10 | jq -nR '[inputs | {Id: (.|tostring), MessageBody: ("standard:" + .)}]')"
aws sqs send-message-batch --queue-url https://sqs.ap-northeast-1.amazonaws.com/{アカウントID}/{キューのName} --entries "$(seq 10 | jq -nR '[inputs | {Id: (.|tostring), MessageBody: ("standard:" + .)}]')"
```

```bash
aws sqs send-message-batch --queue-url https://sqs.ap-northeast-1.amazonaws.com/339712847759/hands-on-queue.fifo --entries "$(seq 10 | jq -nR '[inputs | {Id: (.|tostring), MessageBody: ("fifo:" + .), MessageGroupId: "aaa"}]')"
aws sqs send-message-batch --queue-url https://sqs.ap-northeast-1.amazonaws.com/{アカウントID}/{キューのName} --entries "$(seq 10 | jq -nR '[inputs | {Id: (.|tostring), MessageBody: ("fifo:" + .), MessageGroupId: "aaa"}]')"

```

