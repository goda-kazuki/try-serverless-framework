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

AWS側のCloudFormationが実行されます。 TODO 画像1

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

### ステージによって環境変数や名前を分けたい時

開発環境と本番環境で環境変数を分けたい時ってあると思いますが、そういった設定も可能です。

1. configの中にdevとprdの環境設定ファイルを用意しておきます。

2. stageの設定を行います。
   `stage: ${opt:stage, self:custom.defaultStage}`
3. 環境変数のファイルの場所を設定します。

```yaml
custom:
  defaultStage: dev
  otherfile:
    environment:
      dev: ${file(./config/dev.yml)}
      prd: ${file(./config/prd.yml)}
```

4. 環境変数指定している部分を書き換える
   `CONFIG: ${self:custom.otherfile.environment.${self:provider.stage}.CONFIG}`

5. デプロイします
   `npx sls deploy --stage prd`

### VPCに関連付けたい場合
サブネットとセキュリティグループを指定することで、VPCに関連付けることが出来ます。
```yaml
  vpc:
    securityGroupIds:
      - ${self:custom.otherfile.environment.${self:provider.stage}.LAMBDA_VPC_SECURITY_GROUP}
    subnetIds:
      - ${self:custom.otherfile.environment.${self:provider.stage}.LAMBDA_VPC_SUBNET_ID}
```

### IAM Roleの作成と紐付け
1. resourceにIAM Roleを作成するように設定
```yaml
resources:
  Resources:
     tryServerlessFrameworkLambdaRole:
      Type: AWS::IAM::Role
      Properties:
        RoleName: try-serverless-framework-${opt:stage, self:provider.stage}-lambdaRole
        AssumeRolePolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Principal:
                Service:
                  - lambda.amazonaws.com
              Action: sts:AssumeRole
        ManagedPolicyArns:
          - "arn:aws:iam::aws:policy/AmazonS3FullAccess"
          - "arn:aws:iam::aws:policy/AWSLambda_FullAccess"
          - "arn:aws:iam::aws:policy/AmazonRDSFullAccess"
          - "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
          - "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
```

2. 作成したRoleを使うように設定
```yaml
provider:
  iam:
    role: tryServerlessFrameworkLambdaRole
```

### S3バケットの作成
```yaml
resources:
   Resources:
    Bucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: try-serverless-framework-${opt:stage, self:provider.stage}-image-buckets
```
