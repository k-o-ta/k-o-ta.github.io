+++
title = "intelliJ IDEAを使ったsolr pluginのデバッグ"
date = 2020-09-24
[taxonomies]
tags = ["solr", "intelliJ", "java"]
+++

solrのプラグイン開発を行うときにintelliJ IDEAを使ってdebugを行う方法

## 手順
1. plugin用のリポジトリを作成&開発して、jarファイルを生成する(生成されたjarファイルを`plugin.jar`とする)
2. [sorl本体のリポジトリ](https://github.com/apache/lucene-solr)をpluginと別の場所にcloneしてくる。
3. antとivyをインストールする(solr本体をbuildするのに使う)
4. solr本体をビルドする
```
cd lucene-solr
ant compile
cd solr
ant compile
ant example
```
5. 必要に応じて`solr.xml`を修正
6. pluginで生成したjarファイルをsolr本体の`lucene-solr/solr/ext_lib/`配下に置く
7. lucene-solr/solr配下で `bin/solr start -s sample_project_name/solr/ -f -a "-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=7666"`
8. intelliJ IDEAでsolr本体のプロジェクトを作成する。デバッグでステップ実行するのはこのプロジェクト。`File > New > Project` からプロジェクトを作成する(`Project From Existing Source`ではない)
9. `Run > Edit Configuration`から以下のように設定する。ポイントはSettingsの欄
![configuration](https://www.dropbox.com/s/4vh5wjjbllxvzbk/solor-plugin-debug-on-intellij.png?dl=1)
10. project paneの`lucene-sorl/solr/ext_lib/plugin.jar`を右クリックしてAdd As Libraryをクリックして、表示されたポップアップもそのままOKにする
11. jarファイルのソースが表示されるようになったはずなので適当な箇所にbreak pointを貼って`Run Debug`を実行する
12. デバック実行時に使うサンプル用のデータをsolrに登録して、`http://localhost:8983` にsolrリクエストを投げるとbreak pointでstopする
