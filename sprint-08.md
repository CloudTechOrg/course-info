<details>
  <summary> 
【ハンズオン】Lambda関数の実行
</summary> 

```
import json

def lambda_handler(event, context):
    # イベントから 'key1' の値を取得
    key1_value = event.get('key1', 'No key1 provided')
    
    # レスポンスとして 'key1' の値を返す
    return {
        'statusCode': 200,
        'body': json.dumps({'key1': key1_value})
    }
``` 
</details>

### クラウドエンジニアのためのPython講座

ファイル等は以下にまとめられています。  
[クラウドエンジニアのためのPython講座リポジトリへのリンク](https://github.com/CloudTechOrg/course-python)