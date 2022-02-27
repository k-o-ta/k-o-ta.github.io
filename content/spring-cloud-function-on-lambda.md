+++
title = "spring cloud function on AWS Lambda"
date = 2022-02-27
[taxonomies]
tags = ["kotlin", "spring", "lambda"]
[extra]
toc = true
+++

## やること
kotlinでspring cloud functionをつかったバッチ処理を実装してAWS lambdaにデプロイする。コンテナ環境で実行できるようにする。

## 簡単に始める
ドキュメントは[こちら](https://docs.spring.io/spring-cloud-function/docs/3.2.1/reference/html/spring-cloud-function.html)

依存を入れる。(gradleを使っているとする)
```kotlin
// build.gradle.kts
dependencies {
  implementation("org.springframework.cloud:spring-cloud-function-context")
  implementation("org.springframework.cloud:spring-cloud-starter-function-web")
}
```

関数を実装する。kotlinを使う場合はlambdaを使って実装するのが手軽。
lambdaを返すメソッドを定義してあげれば良い。返り値のlambdaのsignatureが関数のsignatureと一致することに注意。
```kotlin
package com.project.my

@SpringBootApplication

class Samplelication(
// 必要に応じて@Autowiredする
) {

    @Bean
    fun test(): () -> String {
        return { "test" }
    }

    @Bean
    fun uppercase(): (String) -> String {
        return { it.uppercase() }
    }
}
```

gradleでプロジェクトを作っている場合は
`gradle bootRun`を実行するとwebサーバーが立ち上がるので、
`curl -H "Content-Type: text/plain" localhost:8080/test` や `curl -H "Content-Type: text/plain" localhost:8080/upercase -d Hello` のようにしてあげるとそれぞれの関数をローカルで実行できる。

もっと複雑な引数はjson形式で渡すことができる(jsonをマッピングさせるkotlinのdataクラスを作りlambdaの引数や返り値に設定する)

## AWS lambdaにデプロイする
ドキュメントは[こちら](https://docs.spring.io/spring-cloud-function/docs/3.2.1/reference/html/aws.html)

* ドキュメントで指定されたpluginを入れてjarファイルを作りaws lambdaにアップロード
* 環境変数`MAIN_CLASS` に`com.project.my::SmapleApplication`と入れる
* Handlerには `org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest` を指定する
* 関数が複数ある場合は環境変数(`SPRING_CLOUD_FUNCTION_DEFINITION`)に実装したメソッド名を指定すると関数を指定できる。

## コンテナ実行
せっかくなのでコンテナ上で実行したい。コンテナは[aws-lambda-java](https://hub.docker.com/r/amazon/aws-lambda-java)を使う。

```Dockerfile
# lambda/Dockerfile

FROM public.ecr.aws/lambda/java:11

ENV MAIN_CLASS=com.project.my.SmapleApplication

COPY my-0.0.1-SNAPSHOT-aws.jar ${LAMBDA_TASK_ROOT}
# awsのjava用コンテナは実行に必要なクラスファイルを所定の場所に設置することを要求する
RUN jar -xf my-0.0.1-SNAPSHOT-aws.jar

# aws lambda javaコンテナではhandlerをCMDに指定することになっている
CMD ["org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest"]
```
基本的にはaws-lambda-javaの説明通りなのだが、MAIN_CLASSを環境変数内で指定しておくとawsコンソール上での設定を省けて楽。`SPLING_CLOUD_FUNCTION_DEFINITION` を指定もできるが、そうするとこのコンテナが特定の関数専用になってしまうので、deploy時にlambdaの方に指定してあげるのが良いと思う。

手元で実行するときは以下のようにする
1. shadowJarファイルを生成しておく
  * `gradle shadowJar`
2. buildする
  * `docker build ./lambda -t sample-application/lambda`
3. 起動する
  * `docker run --env SPRING_CLOUD_FUNCTION_DEFINITION=uppercase -p 9000:8080 sample-application/lambda`
    * 起動時に環境変数として`SPRING_CLOUD_FUNCTION_DEFINITIOIN=関数名`とすることで実行対象の関数を指定できる。staging環境にしたり、IAMユーザーの情報も起動時に環境変数から渡せる
4. 関数を実行する(コンテナイメージにはAWS Lambda Runtime Interface Clients が含まれている)
  * `curl  -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '"fOo"'`
    * 3で指定した関数が実行される。違う関数を試したい場合は3にわたす環境変数名を変える。引数を取らない関数でも空の引数を渡してあげる必要がある。
      * `curl  -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d {}`

## deploy

蛇足だが、CicleCIでdeployするときの一部設定
```YAML
jobs:
  build:
    steps:
      # generate shadowJar
      - run: gradle shadowJar
      - persist_to_workspace:
          root: lambda
          paths:
            - my-0.0.1-SNAPSHOT-aws.jar
            - Dockerfile
    build-and-push-image:
    parameters:
      aws_access_key_id:
        type: env_var_name
      aws_secret_access_key:
        type: env_var_name
      region:
        type: env_var_name
      aws_ecr_account_url:
        type: env_var_name
      repository_name:
        type: string
      secret_name_msk:
        type: string
    executor: aws-ecr/default
    working_directory: ~/repo
    steps:
      - attach_workspace:
          at: ~/repo/lambda
      - aws-cli/setup:
          aws-access-key-id: <<parameters.aws_access_key_id>>
          aws-region: <<parameters.region>>
          aws-secret-access-key: <<parameters.aws_secret_access_key>>
      - aws-ecr/build-and-push-image:
          checkout: false
          account-url: <<parameters.aws_ecr_account_url>>
          repo: <<parameters.repository_name>>
          region: <<parameters.region>>
          tag: "${CIRCLE_SHA1}"
          aws-access-key-id: <<parameters.aws_access_key_id>>
          aws-secret-access-key: <<parameters.aws_secret_access_key>>
          path: ~/repo/lambda
  update-lambda:
    parameters:
      aws_access_key_id:
        type: env_var_name
      aws_secret_access_key:
        type: env_var_name
      region:
        type: env_var_name
      aws_ecr_account_url:
        type: env_var_name
      repository_name:
        type: string
      function_name:
        type: string
      spring_cloud_function_definition:
        type: string
    executor: aws-cli/default
    steps:
      - aws-cli/setup:
          aws-access-key-id: <<parameters.aws_access_key_id>>
          aws-region: <<parameters.region>>
          aws-secret-access-key: <<parameters.aws_secret_access_key>>
      - jq/install
      - run:
          command: |
            aws lambda update-function-code --function-name <<parameters.function_name>> --image-uri ${<<parameters.aws_ecr_account_url>>}/<<parameters.repository_name>>:${CIRCLE_SHA1}
            for _i in {1..5} ; do
              if [ $(aws lambda get-function-configuration  --function-name my-sample-application | jq .LastUpdateStatus -r) = Successful ]; then
                break
              else
                sleep 15
              fi
            done
            aws lambda update-function-configuration --function-name <<parameters.function_name>> --environment "Variables={SPRING_CLOUD_FUNCTION_DEFINITION=<<parameters.spring_cloud_function_definition>>}"
```
