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

