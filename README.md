# try-serverless-framework
## 前提
AWS CLIが使えること

## 使ってみる
1. serverless frameworkのインストール
`npm install serverless`

2. 実行環境の作成
`npx serverless create --template aws-nodejs`

3. リージョンの変更
```
us-east-1
→ap-northeast-1
```

4. デプロイしてみる
`npx sls deploy `

AWS側のCloudFormationが実行されます。
TODO 画像1

5. デプロイしたLambdaを実行してみる
`npx serverless invoke --function hello`

無事にリクエストが完了しました。
```
[try-serverless-framework]$ npx serverless invoke --function hello                                                                              +[main]
{
    "statusCode": 200,
    "body": "{\n  \"message\": \"Go Serverless v1.0! Your function executed successfully!\",\n  \"input\": {}\n}"
}
```

## その他設定
### 環境変数の設定
各functionの下にenvironmentという記述をすれば、環境変数が設定されます。
```yaml
functions:
  hello:
    handler: handler.hello
    environment:
      CONFIG: "CONFIGの内容"
```

