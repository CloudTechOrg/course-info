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

---

<details>
  <summary>02.【ハンズオン】New Relic AWS連携/APIポーリング・Metric Streams設定/連携確認/一本化【14:15】
  </summary>

#### CloudFormationテンプレート
[cfn-newrelic-aws-integration.yaml](sprint-09-file/02/cfn-newrelic-aws-integration.yaml)
- EC2インスタンス（t3.micro） + Lambda関数を作成
- スタック名: `newrelic-handson-resources`

#### ハンズオンの概要
CloudFormationで検証用EC2・Lambdaを作成し、API PollingとMetric Streamsの両方でAWS連携を設定。
連携状況をNRQLで確認した後、Metric Streamsに一本化する構成を体験します。

#### NRQL（連携状況確認）
```sql
-- EC2メトリクス確認
FROM Metric SELECT average(aws.ec2.CPUUtilization)
WHERE metricName = 'aws.ec2.CPUUtilization' SINCE 3 hours ago

-- Lambda メトリクス確認（Metric Streams経由）
FROM Metric
SELECT latest(aws.lambda.Invocations), latest(aws.lambda.Errors), latest(aws.lambda.Duration)
WHERE collector.name = 'cloudwatch-metric-streams'
  AND entity.name = 'newrelic-session2-function'
SINCE 30 minutes ago
```

</details>

---

<details>
  <summary>03.【ハンズオン】Infrastructure Agent導入/ガイデッドインストール/CPU負荷テスト/プロセスフィルタリング【10:14】
  </summary>

#### CloudFormationテンプレート
[cfn-newrelic-infrastructure.yaml](sprint-09-file/03/cfn-newrelic-infrastructure.yaml)
- EC2インスタンス（t3.micro） + SSM Session Manager用IAMロールを作成
- スタック名: `newrelic-handson-resources`

#### ハンズオンの概要
CloudFormationでEC2を作成し、Guided InstallでInfrastructure Agentをインストール。
CPU負荷テストとプロセスフィルタリング設定を体験します。

#### Infrastructure Agent インストール
New Relic UI > Integrations & Agents > Linux で生成されるコマンドを実行

#### CPU負荷テスト（stress-ng）
```bash
stress-ng --cpu 1 --timeout 300s
```

#### プロセスフィルタリング設定（/etc/newrelic-infra.yml）
```yaml
include_matching_metrics:
  process.name:
    - amazon-ssm-agent
    - newrelic-infra
```

#### Agent再起動・状態確認
```bash
sudo systemctl restart newrelic-infra
sudo systemctl status newrelic-infra
```

</details>

---

<details>
  <summary>04.【ハンズオン】APM・Lambda監視/APMエージェント導入/Lambda Layer追加/分散トレーシング確認【15:22】
  </summary>

#### CloudFormationテンプレート
[cfn-newrelic-apm-lambda.yaml](sprint-09-file/04/cfn-newrelic-apm-lambda.yaml)
- EC2（Flask + Gunicorn）、Lambda、API Gateway（HTTP API）、DynamoDBを作成
- スタック名: `newrelic-handson-resources`

#### ハンズオンの概要
FlaskアプリにAPMエージェント、LambdaにNew Relic Layerを導入し、
分散トレーシングとエラートラッキングを体験します。

#### APMエージェント導入（Flask / Python）
New Relic UI > Integrations & Agents > Python の手順に従う
```bash
pip install newrelic
newrelic-admin generate-config <LICENSE_KEY> newrelic.ini
```

#### Flaskサービスの起動（APMエージェント付き）
systemdのサービスファイル（flask-app.service）を編集し、`newrelic-admin run-program` 経由で起動
```bash
sudo systemctl daemon-reload
sudo systemctl restart flask-app
sudo systemctl status flask-app
```

#### Lambda Layer設定
- Lambda Layer ARN: [layers.newrelic-external.com](https://layers.newrelic-external.com) から取得（東京リージョン / x86_64 / Python 3.13）
- ハンドラー変更: `newrelic_lambda_wrapper.handler`

#### Lambda環境変数
```
NEW_RELIC_LAMBDA_HANDLER = index.lambda_handler
NEW_RELIC_LICENSE_KEY     = <Ingest License Key>
NEW_RELIC_ACCOUNT_ID      = <Account ID>
```

#### トラフィック生成（テスト用）
```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/users
curl "http://127.0.0.1:8000/slow?seconds=2"
curl -X POST http://127.0.0.1:8000/invoke-lambda -H "Content-Type: application/json" -d '{"action":"write"}'
curl -X POST http://127.0.0.1:8000/invoke-lambda -H "Content-Type: application/json" -d '{"action":"fail"}'
```

</details>

---

<details>
  <summary>05.【ハンズオン】NRQL実践/集計関数・FACET・TIMESERIES/カスタムダッシュボード構築/JSONエクスポート【15:40】
  </summary>

#### CloudFormationテンプレート
04と同じ [cfn-newrelic-apm-lambda.yaml](sprint-09-file/04/cfn-newrelic-apm-lambda.yaml) を使用

#### ハンズオンの概要
Flask + Lambdaを再デプロイし、NRQLの基本構文から
カスタムダッシュボードの構築・JSONエクスポートまでを体験します。

#### NRQL クエリ集
```sql
-- 生データ確認
SELECT * FROM Transaction LIMIT 5

-- トランザクション件数
SELECT count(*) FROM Transaction SINCE 30 minutes ago

-- 平均レスポンスタイム
SELECT average(duration) FROM Transaction SINCE 30 minutes ago

-- WHERE句で特定エンドポイントに絞り込み
SELECT average(duration) FROM Transaction
WHERE request.uri = '/slow/2' SINCE 30 minutes ago

-- 最大・最小レスポンスタイム
SELECT max(duration), min(duration) FROM Transaction SINCE 30 minutes ago

-- FACET（エンドポイント別の件数）
SELECT count(*) FROM Transaction FACET request.uri SINCE 30 minutes ago

-- TIMESERIES（時系列グラフ）
SELECT count(*) FROM Transaction
FACET request.uri TIMESERIES 1 minute SINCE 30 minutes ago
```

#### ダッシュボード
- Query Your Data からウィジェットを「Add to Dashboard」で追加
- JSON形式でエクスポート可能（右上メニュー > Copy JSON to clipboard）

</details>

---

<details>
  <summary>06.【ハンズオン】アラート設定/NRQL Condition作成/Event Flow・Event Timer/テスト発火・メール通知確認【16:44】
  </summary>

#### CloudFormationテンプレート
04と同じ [cfn-newrelic-apm-lambda.yaml](sprint-09-file/04/cfn-newrelic-apm-lambda.yaml) を使用

#### ハンズオンの概要
Flask + Lambda + Infrastructure Agentの環境を再デプロイし、
NRQL Alert Conditionの作成からテスト発火・メール通知確認までを体験します。

#### Alert Condition 1: CPU使用率（Event Flow）
```sql
SELECT average(cpuPercent) FROM SystemSample
```
- Threshold: Static / Critical
- Aggregation method: Event Flow

#### Alert Condition 2: Lambdaエラー検知（Event Timer）
```sql
SELECT count(*) FROM AwsLambdaInvocationError
```
- Threshold: Static / Critical（1件以上で発火）
- Aggregation method: Event Timer

#### Workflow設定
Alert Policy に Email通知のWorkflowを紐付け

#### テストアラート発火
```bash
# CPU使用率アラート: stress-ngでCPU負荷
stress-ng --cpu 1 --timeout 300s

# Lambdaエラーアラート: エラーを意図的に発生させる
curl -X POST http://127.0.0.1:8000/invoke-lambda -H "Content-Type: application/json" -d '{"action":"fail"}'
```

#### アラートクローズ
Issues画面 > 対象Issue > Close Issue

</details>

