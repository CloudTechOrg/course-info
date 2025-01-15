<details>
  <summary> 
【AWS CLI】AWS公式のAWSCLIコンテナ環境を利用したコマンド実行のスライド説明とハンズオン
</summary> 

#### コマンド

```bash
docker info

docker run --rm -it amazon/aws-cli help

docker run --rm -ti -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli

alias

alias aws-cli='docker run --rm -ti -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli'

aws-cli s3 sync s3://同期元バケットAの名前 s3://同期先バケットBの名前

unalias aws-cli
```
</details>

<details>
  <summary> 
【ハンズオン】Lambda同期呼び出し/非同期呼び出し
</summary> 

```bash
■DryRun実行
aws lambda invoke --function-name hello-python --invocation-type DryRun --payload '{ "key1": "value1","key2": "value2","key3": "value3" }' --cli-binary-format raw-in-base64-out response.json

■同期呼び出し
aws lambda invoke --function-name hello-python --invocation-type RequestResponse --payload '{ "key1": "value1","key2": "value2","key3": "value3" }' --cli-binary-format raw-in-base64-out response.json

■非同期呼び出し
aws lambda invoke --function-name hello-python --invocation-type Event --payload '{ "key1": "value1","key2": "value2","key3": "value3" }' --cli-binary-format raw-in-base64-out response.json

■aws lambda invoke公式リファレンス
https://docs.aws.amazon.com/cli/latest/reference/lambda/invoke.html
```
</details>

<details>
  <summary> 
【ハンズオン】FSx for Windowsを複数のWindowsインスタンスからアタッチ
</summary> 

#### CloudFormationテンプレート
```yml
AWSTemplateFormatVersion: 2010-09-09
Parameters:
  VPCIDParameter:
    Type: String
    Description: Enter VPC ID.
Resources: 
  secGroupName:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: FSx-SecurityGroup
      GroupDescription: FSx-SecurityGroup
      VpcId:
        Ref: VPCIDParameter
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 53
          ToPort: 53
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 53
          ToPort: 53
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 88
          ToPort: 88
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 88
          ToPort: 88
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 123
          ToPort: 123
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 135
          ToPort: 135
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 389
          ToPort: 389
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 389
          ToPort: 389
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 445
          ToPort: 445
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 464
          ToPort: 464
          CidrIp: 0.0.0.0/0
        - IpProtocol: udp
          FromPort: 464
          ToPort: 464
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 636
          ToPort: 636
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3268
          ToPort: 3268
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3269
          ToPort: 3269
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9389
          ToPort: 9389
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 49152
          ToPort: 65535
          CidrIp: 0.0.0.0/0
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 445
          ToPort: 445
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 5985
          ToPort: 5985
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: "Name"
          Value: "FSx-SecurityGroup"
```
</details>